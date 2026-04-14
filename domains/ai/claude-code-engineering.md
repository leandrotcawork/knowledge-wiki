domain: ai
confidence: low
sources: 3
last_updated: 2026-04-14

# Claude Code Best Practices and Agentic Engineering

A engenharia agêntica (Agentic Engineering) representa uma mudança de paradigma: deixamos de escrever código imperativo para orquestrar sistemas probabilísticos autônomos. O **Claude Code** é a interface de linha de comando (CLI) agêntica da Anthropic, projetada para atuar diretamente no sistema de arquivos, executar comandos, interagir com APIs e escrever código de forma autônoma.

Este documento serve como a referência definitiva para arquitetar repositórios, gerenciar contexto e orquestrar fluxos de trabalho utilizando o Claude Code em ambientes de produção.

---

## 1. Princípios da Engenharia Agêntica

Trabalhar com agentes requer uma mentalidade diferente da engenharia de software tradicional. O recurso mais escasso não é o tempo de CPU, mas a **janela de contexto (Context Window)** e a **atenção do modelo**.

1. **O Contexto é como Memória RAM**: Quanto mais cheio o contexto, maior a degradação de performance (alucinações, esquecimento de regras). Gerencie-o agressivamente.
2. **Verificação > Geração**: Agentes são excelentes em gerar código, mas péssimos em saber se ele funciona sem feedback empírico. Sempre forneça um loop de verificação (testes automatizados, linters, scripts de validação).
3. **Disclosure Progressivo (Progressive Disclosure)**: Não injete todo o conhecimento do domínio no prompt inicial. Ensine o agente a *descobrir* o conhecimento sob demanda.
4. **Isolamento de Falhas**: Use subagentes para tarefas exploratórias. Se um subagente falhar ou poluir seu contexto com erros de compilação, o contexto do agente principal permanece limpo.

---

## 2. Arquitetura e Componentes do Claude Code

O ecossistema do Claude Code é composto por várias primitivas que permitem estender suas capacidades e controlar seu comportamento.

```text
+-------------------------------------------------------------------+
|                        Claude Code CLI                            |
|                                                                   |
|  +----------------+    +-----------------+    +----------------+  |
|  |   CLAUDE.md    |    |     Skills      |    |    Commands    |  |
|  | (System Prompt)|    | (Auto-discovery)|    | (Slash /cmds)  |  |
|  +-------+--------+    +--------+--------+    +--------+-------+  |
|          |                      |                      |          |
+----------|----------------------|----------------------|----------+
           v                      v                      v
+-------------------------------------------------------------------+
|                       Contexto Principal                          |
|                                                                   |
|   [User Prompt] + [Files (@)] + [Tool Outputs] + [History]        |
+---------------------------------+---------------------------------+
                                  |
            +---------------------+---------------------+
            |                     |                     |
            v                     v                     v
  +-------------------+ +-------------------+ +-------------------+
  |    Subagentes     | |    MCP Servers    | |      Hooks        |
  | (Contexto Forked) | | (Ferramentas Ext) | | (Eventos de Ciclo)|
  +-------------------+ +-------------------+ +-------------------+
```

### 2.1. CLAUDE.md (O Ponto de Entrada)
O arquivo `CLAUDE.md` na raiz do repositório atua como o *System Prompt* injetado em todas as sessões. 

**Regra de Ouro:** Mantenha-o com **menos de 200 linhas**.
Ele não deve conter a documentação completa do sistema. Deve conter apenas:
- Comandos essenciais de build/test (ex: `make test`, `go run main.go`).
- Regras estritas de formatação ou arquitetura.
- Ponteiros para onde o agente pode encontrar mais informações (Skills).

### 2.2. Skills (Habilidades)
Skills são pastas contendo arquivos Markdown que o Claude Code pode descobrir e ler automaticamente. Elas implementam o padrão de *Progressive Disclosure*.

Cada Skill deve ter um *YAML frontmatter* definindo seu propósito:

```yaml
---
name: database-schema-guide
description: Regras e padrões para alterar o schema do banco de dados usando SQLAlchemy.
context: fork
---
# Database Schema Guide
...
```
*Nota: O parâmetro `context: fork` instrui o Claude a abrir um subagente ao utilizar esta skill, protegendo o contexto principal.*

### 2.3. Subagentes (Subagents)
Subagentes são atores autônomos iniciados com um contexto limpo e isolado. O agente principal delega uma tarefa (ex: "Pesquise como a API de pagamentos funciona e me traga um resumo") para um subagente.

**Por que usar Subagentes?**
Quando o Claude tenta resolver um bug complexo, ele pode executar dezenas de comandos, ler logs extensos e cometer erros. Se tudo isso acontecer no contexto principal, a janela de contexto enche rapidamente com "lixo" (stack traces irrelevantes), degradando a capacidade de raciocínio futuro. O subagente absorve esse lixo, morre após a tarefa, e retorna apenas a solução destilada para o agente principal.

### 2.4. Model Context Protocol (MCP) Servers
O MCP é um protocolo aberto que padroniza como modelos de IA acessam ferramentas e fontes de dados externas. Em vez de hardcodar integrações, você expõe um servidor MCP local ou remoto que o Claude Code consome.

---

## 3. Fluxo de Trabalho Agêntico (Workflow)

Engenheiros seniores não pedem ao Claude para "fazer a feature X". Eles orquestram um pipeline de desenvolvimento: **Research -> Plan -> Execute -> Review -> Ship**.

### Fase 1: Research & Plan (Modo de Planejamento)
Sempre inicie tarefas complexas no *Plan Mode*.
1. Peça ao Claude para explorar a base de código usando ferramentas de busca (grep, ast-grep).
2. Forneça contexto específico usando `@arquivo.py` ou `@https://docs.api.com`.
3. Exija que ele escreva um plano de implementação em um arquivo temporário (ex: `PLAN.md`).

### Fase 2: Execute (Modo Auto)
Uma vez que o plano está aprovado, mude para o modo de execução (Auto mode).
- O agente deve seguir o `PLAN.md` passo a passo.
- **Dica Crítica:** Instrua o agente a fazer commits atômicos por arquivo alterado, com mensagens descritivas.

### Fase 3: Review & Verify (O Loop de TDD Agêntico)
Nunca confie em código gerado sem verificação. O Claude Code precisa de uma forma de validar seu próprio trabalho.

**Matriz de Decisão de Verificação:**

| Tipo de Alteração | Ferramenta de Verificação Recomendada | Ação do Agente |
| :--- | :--- | :--- |
| Lógica de Backend | Pytest / Go test | Executar testes específicos da unit alterada. |
| UI / Frontend | Screenshots / Playwright | Tirar screenshot via CLI e analisar visualmente. |
| Refatoração | Linters (Ruff, golangci-lint) | Rodar linter e corrigir avisos antes do commit. |

---

## 4. Gerenciamento Avançado de Contexto

A degradação de contexto é o inimigo número um da engenharia agêntica. Quando o contexto atinge cerca de 50% da capacidade máxima, a capacidade do modelo de seguir instruções complexas (instruction-following) cai drasticamente.

### A Estratégia `/compact`
O comando `/compact` força o Claude a resumir a conversa atual, descartar o histórico de turnos intermediários (tentativas e erros) e reter apenas o estado atual e o objetivo.
- **Quando usar:** Manualmente, sempre que o contexto atingir 50%, ou após a conclusão de uma sub-tarefa complexa.

### Injeção de Contexto Cirúrgica
Evite pedir ao Claude para "procurar onde a autenticação é feita". Isso gasta tokens de raciocínio e contexto com buscas `grep`. Em vez disso, injete cirurgicamente:
`> Adicione suporte a PKCE no fluxo de login. Use @auth/oauth2.py e @docs/pkce-rfc.md como referência.`

---

## 5. Implementação Prática: Estendendo o Claude Code

### 5.1. Criando um Servidor MCP (Exemplo em Python)

Para integrar o Claude Code com sistemas internos (ex: um banco de dados de staging ou uma API proprietária), construímos um servidor MCP. Abaixo, um exemplo conceitual usando Python e Pydantic para definir ferramentas estritas.

```python
# mcp_server.py
import json
import sys
from typing import Any, Dict
from pydantic import BaseModel, Field

class QueryUserRequest(BaseModel):
    email: str = Field(..., description="Email do usuário para buscar no banco de staging")

def get_user_from_staging(email: str) -> Dict[str, Any]:
    # Simulação de acesso a banco de dados interno
    # Em produção, usaria SQLAlchemy ou similar
    mock_db = {"admin@empresa.com": {"id": 1, "role": "admin", "status": "active"}}
    return mock_db.get(email, {"error": "User not found"})

def handle_request(request_str: str) -> str:
    """Processa requisições JSON-RPC simplificadas do Claude Code via stdio"""
    try:
        req = json.loads(request_str)
        if req.get("method") == "query_user":
            params = QueryUserRequest(**req.get("params", {}))
            result = get_user_from_staging(params.email)
            return json.dumps({"jsonrpc": "2.0", "id": req.get("id"), "result": result})
    except Exception as e:
        return json.dumps({"jsonrpc": "2.0", "error": str(e)})

if __name__ == "__main__":
    # Servidores MCP frequentemente comunicam-se via stdin/stdout
    for line in sys.stdin:
        response = handle_request(line)
        sys.stdout.write(response + "\n")
        sys.stdout.flush()
```

### 5.2. Scripts de Verificação (Exemplo em Go)

Para garantir que o Claude Code não quebre a build, podemos fornecer scripts de verificação rápidos que ele deve invocar via Hooks ou instruções no `CLAUDE.md`.

```go
// verify_build.go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

// Este script é chamado pelo Claude Code após modificar arquivos Go.
// Ele garante que o código formata, compila e passa nos testes básicos.
func main() {
	steps := []struct {
		name string
		cmd  *exec.Cmd
	}{
		{"Go Fmt", exec.Command("go", "fmt", "./...")},
		{"Go Vet", exec.Command("go", "vet", "./...")},
		{"Go Test", exec.Command("go", "test", "-short", "./...")},
	}

	for _, step := range steps {
		fmt.Printf("Executando: %s...\n", step.name)
		out, err := step.cmd.CombinedOutput()
		if err != nil {
			fmt.Fprintf(os.Stderr, "❌ Falha em %s:\n%s\n", step.name, string(out))
			os.Exit(1) // O exit code 1 sinaliza ao Claude Code que a verificação falhou
		}
		fmt.Printf("✅ %s passou.\n", step.name)
	}
	fmt.Println("🚀 Todas as verificações passaram. Seguro para commit.")
}
```

---

## 6. Anti-Patterns e Armadilhas Comuns

1. **O "God Prompt" no CLAUDE.md**: Colocar 1000 linhas de documentação no `CLAUDE.md`.
   *Solução*: O modelo vai ignorar as instruções do meio (fenômeno *Lost in the Middle*). Use Skills e subagentes para carregar contexto sob demanda.
2. **Loop Infinito no Auto Mode**: O agente tenta rodar um teste, falha, tenta corrigir cegamente, falha novamente, repetindo até esgotar os tokens.
   *Solução*: Estabeleça limites de custo/iteração. Instrua no `CLAUDE.md`: *"Se um teste falhar 3 vezes seguidas, pare e peça ajuda ao usuário."*
3. **Falta de Ground Truth**: Pedir ao agente para refatorar código sem testes unitários cobrindo a funcionalidade.
   *Solução*: A primeira tarefa do agente deve ser escrever testes para o comportamento atual (caracterização) antes de iniciar a refatoração.

---

## See Also
- [[llm-context-windows]] - Aprofundamento sobre como modelos de linguagem gerenciam atenção e o fenômeno "Lost in the Middle".
- [[model-context-protocol]] - Especificação completa do MCP (Model Context Protocol).
- [[agentic-design-patterns]] - Padrões de design para sistemas multi-agentes e delegação de tarefas.
- [[test-driven-development]] - Práticas de TDD, essenciais para fornecer *ground truth* a agentes autônomos.

## Sources
- `raw/articles/claude-code-best-practice-readme.md`
- `raw/articles/claude-code-best-practice-claudemd.md`
- `raw/articles/claude-code-official-best-practices.md`