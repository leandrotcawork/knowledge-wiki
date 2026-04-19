---
domain: domains/ai/agentic-coding-workflows-plugins.md
confidence: high
sources: 12
last_updated: 2026-04-14
---

# Workflows, Plugins e Skills para Codex e Claude Code

A transição de assistentes de código baseados em autocompletar (como as primeiras versões do GitHub Copilot) para **agentes autônomos de engenharia de software** (como Claude Code, Cursor e implementações modernas baseadas em OpenAI Codex) exigiu uma nova arquitetura de extensibilidade. Em vez de apenas prever a próxima linha de código, essas ferramentas operam em loops de *Reasoning and Acting* (ReAct), necessitando de acesso a ferramentas externas, contexto de repositório dinâmico e diretrizes de comportamento estritas.

Este documento detalha a arquitetura, os mecanismos de configuração e os padrões de implementação para estender o Claude Code e sistemas baseados em Codex através de Plugins, Skills e Workflows.

---

## 1. Arquitetura de Extensibilidade

Para que um Large Language Model (LLM) atue como um agente de software, ele requer "mãos" (ferramentas de execução/skills) e "olhos" (recursos de leitura/contexto). A padronização dessa comunicação divergiu em duas abordagens principais: o **Model Context Protocol (MCP)**, liderado pela Anthropic, e as interfaces de **Function Calling / Tool Use**, popularizadas pela OpenAI.

### O Loop de Execução Agêntica (ReAct)

Quando o Claude Code ou um agente Codex é invocado, o fluxo interno segue um padrão de máquina de estados iterativa, frequentemente referida como o loop ReAct:

```ascii
+-------------------+       +-----------------------+       +-------------------+
|                   |       |                       |       |                   |
|  User Prompt /    | ----> |  Context Assembly     | ----> |  LLM Inference    |
|  Task Definition  |       |  (CLAUDE.md, RAG)     |       |  (Opus/Sonnet/GPT)|
|                   |       |                       |       |                   |
+-------------------+       +-----------------------+       +-------------------+
                                      ^                               |
                                      |                               v
                            +-------------------+           +-------------------+
                            |                   |           |                   |
                            |  Environment      | <-------- |  Tool Execution   |
                            |  Feedback (Logs)  |           |  (Bash, Git, AST) |
                            |                   |           |                   |
                            +-------------------+           +-------------------+
```

1. **Context Assembly:** O agente coleta regras do repositório e estado atual.
2. **Inference (Reasoning):** O modelo decide qual ferramenta usar com base no objetivo.
3. **Tool Execution (Acting):** O plugin ou skill é executado no ambiente local ou em um contêiner seguro.
4. **Feedback:** O resultado (stdout/stderr) é devolvido ao modelo, que decide se a tarefa foi concluída ou se uma nova iteração é necessária.

---

## 2. Model Context Protocol (MCP) no Claude Code

O **Model Context Protocol (MCP)** é um padrão de código aberto introduzido para padronizar como assistentes de IA se conectam a fontes de dados e ferramentas. O Claude Code utiliza o MCP nativamente para expandir suas capacidades além da edição de texto.

### Componentes do MCP
*   **Resources:** Dados estáticos ou dinâmicos que o agente pode ler (ex: logs de banco de dados, documentação de API, tickets do Jira).
*   **Prompts:** Templates parametrizados que ajudam a estruturar fluxos de trabalho específicos.
*   **Tools (Skills):** Funções executáveis que o modelo pode invocar (ex: `execute_sql_query`, `run_npm_test`, `restart_docker_container`).

### Implementação de um Servidor MCP (Exemplo Node.js)
Servidores MCP rodam localmente e expõem ferramentas para o Claude Code via stdio ou SSE (Server-Sent Events).

```javascript
// Exemplo conceitual de um MCP Server para gerenciar Docker
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { exec } from "child_process";

const server = new Server({ name: "docker-mcp", version: "1.0.0" }, { capabilities: { tools: {} } });

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "restart_container") {
    const containerId = request.params.arguments.containerId;
    // Executa o comando real no sistema do usuário
    return new Promise((resolve) => {
      exec(`docker restart ${containerId}`, (error, stdout, stderr) => {
        resolve({ content: [{ type: "text", text: stdout || stderr }] });
      });
    });
  }
});
```

---

## 3. Plugins e Function Calling no Ecossistema Codex

Sistemas baseados em modelos da OpenAI (Codex, GPT-4o) dependem fortemente de **Function Calling**. Ao contrário do MCP, que é um protocolo de transporte e descoberta, o Function Calling exige que o cliente (a IDE ou o CLI) injete um JSON Schema detalhando as ferramentas disponíveis diretamente no payload da API.

### Padrões de Extensão (GitHub Copilot Extensions)
No ecossistema GitHub Copilot, os plugins são frequentemente implementados como serviços RESTful que recebem o contexto do código e retornam ações ou metadados. As *Skills* comuns incluem:
*   **Code Search:** Integração com AST (Abstract Syntax Trees) para navegação semântica.
*   **Linter Integration:** Execução de ferramentas como ESLint ou Ruff em background, alimentando os erros de volta no prompt do agente para autocorreção.

---

## 4. Workflows Agênticos Comuns

A introdução de plugins e skills permite a automação de fluxos de trabalho complexos de engenharia de software.

### 4.1. Test-Driven Development (TDD) Autônomo
O agente recebe um requisito, escreve o teste, executa o framework de testes (via ferramenta CLI), analisa o erro (stderr), escreve a implementação e repete o ciclo até que o teste passe.
*   *Ferramentas necessárias:* Acesso de leitura/escrita ao sistema de arquivos, execução de shell restrita.

### 4.2. Refatoração Guiada por CI/CD
O agente monitora o pipeline de CI. Se um build falhar devido a um erro de tipagem (ex: TypeScript ou Go), o agente usa uma skill de leitura de logs, localiza o arquivo problemático, propõe o patch e cria um commit de correção.

---

## 5. Configuração e Gerenciamento de Contexto

Para evitar que agentes tomem decisões arquiteturais erradas ou utilizem bibliotecas não aprovadas, o comportamento deve ser ancorado por arquivos de configuração no nível do repositório.

### O Padrão `CLAUDE.md` e `.cursorrules`
Arquivos como `CLAUDE.md` (para Claude Code) ou `.cursorrules` (para Cursor IDE) atuam como o *System Prompt* base para o repositório. Eles devem ser densos e factuais.

**Exemplo de estrutura ideal de um `CLAUDE.md`:**
```markdown
# Build Commands
- Run tests: `go test ./... -v`
- Build: `go build -o bin/app cmd/main.go`
- Lint: `golangci-lint run`

# Architecture Rules
- Use standard library `net/http` for routing, no external frameworks.
- Database access must go through the `internal/repository` package using `pgx`.
- Error handling: Always wrap errors with context using `fmt.Errorf("...: %w", err)`.

# Agentic Permissions
- You are allowed to run `go test` autonomously.
- Do NOT modify `go.mod` without explicit user permission.
```

Esses arquivos reduzem a alucinação do modelo e garantem que as *Skills* executadas (como compilação ou linting) usem os comandos exatos esperados pela equipe.

---

## See Also
*   [Model Context Protocol (MCP)](https://modelcontextprotocol.io)
*   [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
*   [Retrieval-Augmented Generation (RAG) em Engenharia de Software](https://en.wikipedia.org/wiki/Retrieval-augmented_generation)
*   [Abstract Syntax Tree (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree)

## Sources
1.  **Anthropic Documentation**: *Model Context Protocol Specification*. (docs.anthropic.com)
2.  **OpenAI API Reference**: *Function Calling and Tool Use*. (platform.openai.com/docs)
3.  **Martin Fowler**: *The Shift to Agentic Workflows in Software Engineering*. (martinfowler.com)
4.  **The Pragmatic Engineer**: *How AI Coding Assistants are Evolving Beyond Autocomplete*. (blog.pragmaticengineer.com)
5.  **Hacker News (YCombinator)**: *Discussions on Claude Code CLI and local MCP servers*. (news.ycombinator.com)
6.  **Andrej Karpathy**: *Software 2.0 and the future of IDEs*. (karpathy.ai)