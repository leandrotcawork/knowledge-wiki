---
domain: domains/ai/agentic-coding-workflows-plugins.md
confidence: low
sources: 5
last_updated: 2026-04-14
---

# Workflows, Plugins e Skills para Codex e Claude Code

A transição de assistentes de código baseados em autocompletar (como as primeiras versões do Copilot) para **agentes autônomos de engenharia de software** (como Claude Code e implementações modernas baseadas em Codex) exigiu uma nova arquitetura de extensibilidade. Em vez de apenas prever a próxima linha, essas ferramentas operam em loops de *Reasoning e Action* (ReAct), necessitando de acesso a ferramentas externas, contexto de repositório e diretrizes de comportamento.

Este documento detalha a arquitetura, os mecanismos de configuração e os padrões de implementação para estender o Claude Code e sistemas baseados em Codex através de Plugins, Skills e Workflows.

---

## 1. Arquitetura de Extensibilidade e MCP

Para que um LLM atue como um agente, ele precisa de "mãos" (ferramentas/skills) e "olhos" (recursos/contexto). A padronização emergente para essa comunicação é o **Model Context Protocol (MCP)**, amplamente adotado pelo ecossistema Claude, enquanto sistemas Codex frequentemente utilizam interfaces de *Function Calling* proprietárias ou adaptadores OpenAPI.

### O Loop de Execução Agêntica

Quando você invoca o Claude Code ou um agente Codex, o fluxo interno segue um padrão de máquina de estados:

```ascii
+-------------------+       +-----------------------+       +-------------------+
|                   |       |                       |       |                   |
|  User Prompt /    | ----> |  Context Assembly     | ----> |  LLM Inference    |
|  Task Definition  |       |  (CLAUDE.md, RAG)     |       |  (Opus/Sonnet)    |
|                   |       |                       |       |                   |
+-------------------+       +-----------------------+       +-------------------+
                                      ^                               |
                                      |                               v
                            +-------------------+           +-------------------+
                            |                   |           |                   |
                            |  Tool Execution   | <-------- |  Tool Call /      |
                            |  (Plugin/Skill)   |           |  Action Decision  |
                            |                   |           |                   |
                            +-------------------+           +-------------------+
                                      |
                                      v
                            +-------------------+
                            |                   |
                            |  Artifact Landing /|
                            |  Commit / Output  |
                            |                   |
                            +-------------------+
```

### Model Context Protocol (MCP)

O MCP é um protocolo bidirecional (geralmente sobre stdio ou SSE/HTTP) que permite que o Claude Code descubra e interaja com ferramentas locais. Um servidor MCP expõe:
1.  **Resources:** Dados estáticos ou dinâmicos (ex: logs, esquemas de banco de dados).
2.  **Prompts:** Templates reutilizáveis.
3.  **Tools (Skills):** Funções executáveis com side-effects (ex: rodar testes, fazer commits, consultar APIs).

---

## 2. Mecanismos de Configuração

Estudos recentes sobre ferramentas de IA agêntica identificam 8 mecanismos principais de configuração. O Claude Code é notável por suportar a gama mais ampla destes mecanismos.

| Mecanismo | Descrição | Exemplo de Uso |
| :--- | :--- | :--- |
| **Context Files** | Arquivos de memória e diretrizes lidos no início da sessão. | `CLAUDE.md`, `.cursorrules`, `AGENTS.md` |
| **Skills / Tools** | Funções atômicas que o modelo pode invocar. | `caveman-compress`, `search_linear_issues` |
| **Subagents** | Delegação de tarefas para modelos menores/mais rápidos. | Enviar linting para Haiku ou Devstral. |
| **Commands** | Macros de CLI que encadeiam prompts e ferramentas. | `/review`, `/plan` |
| **Rules** | Restrições rígidas (frequentemente de segurança). | Bloquear deleção de arquivos `.tf` de produção. |
| **Settings** | Parâmetros do modelo. | `temperature`, `max_tokens` |
| **Hooks** | Scripts executados antes/depois de ações da IA. | Rodar `pre-commit` antes de permitir que a IA faça commit. |
| **MCP** | Servidores externos que injetam contexto e ferramentas. | `Figma MCP`, `Postgres MCP` |

### O Padrão `AGENTS.md` e `CLAUDE.md`

O arquivo `CLAUDE.md` atua como o *system prompt* injetado no diretório de trabalho. Uma antipadrão comum é inchar este arquivo com regras globais.
**Padrão Ouro:** Mantenha o `CLAUDE.md` mínimo. Use-o para *learning loops* (memória de curto prazo do projeto) e delegue regras de arquitetura para arquivos específicos que o agente pode buscar via RAG ou Skills. O `AGENTS.md` está emergindo como um padrão interoperável para definir topologias de subagentes.

---

## 3. Workflows Agênticos

Workflows não são apenas scripts; são metodologias de engenharia de software adaptadas para agentes autônomos.

### A Metodologia "Superpowers"

O framework *Superpowers* (e similares como *Everything*) estrutura a interação em três fases estritas para evitar alucinações e loops infinitos:

1.  **Research (Pesquisa):** O agente usa ferramentas de busca (grep, ast-grep) para mapear o código existente. Nenhuma alteração é permitida.
2.  **Plan (Planejamento):** O agente gera um plano em markdown. **Crucial:** O plano deve ser quebrado em pedaços pequenos (bite-sized).
3.  **Implement (Implementação):** Execução do plano passo a passo.

**Artifact Landing:** Durante a implementação, modelos como Opus ou Sonnet tendem a manter o estado na janela de contexto, o que degrada a performance e aumenta custos. O padrão *Artifact Landing* força o agente a escrever resultados intermediários em arquivos temporários no disco, limpando o contexto da conversa.

### O "Ralph Loop"

Para sessões de codificação de múltiplas horas, a janela de contexto inevitavelmente se polui. O *Ralph Loop* (um padrão oficializado em alguns plugins) resolve isso:
1. O agente completa uma subtarefa.
2. Ele gera um resumo denso do estado atual e do que foi feito.
3. A sessão atual é encerrada (limpando o contexto).
4. Uma nova sessão é iniciada, recebendo apenas o resumo denso como contexto inicial.

---

## 4. Ecossistema de Plugins e Skills

O ecossistema de plugins do Claude Code (acessível via `claude plugin marketplace`) expande drasticamente as capacidades do agente.

### Otimização de Tokens: Caveman

O plugin **Caveman** (`JuliusBrussee/caveman`) é um estudo de caso em eficiência de LLMs. Ele força o modelo a reduzir os tokens de saída em até 75%.
*   **Por que funciona:** LLMs frequentemente geram "AI slop" (texto prolixo, desculpas, explicações desnecessárias).
*   **Níveis:** Lite, Full, Ultra, Wenyan (níveis crescentes de restrição de vocabulário).
*   **Sub-skills:** `caveman-commit` (commits no estilo Unix, curtos e diretos), `caveman-review`, `caveman-compress` (comprime arquivos de memória como `CLAUDE.md` removendo stopwords e formatação desnecessária).

### Integrações de Infraestrutura e Produto

*   **Firecrawl:** Converte documentações web e sites em Markdown limpo, ideal para RAG. Essencial quando o agente precisa ler a documentação de uma biblioteca que foi atualizada após o *knowledge cutoff* do modelo.
*   **Context7:** Fornece documentação atualizada diretamente no contexto.
*   **Linear / Jira:** Permite que o agente leia a issue, entenda os critérios de aceite e mova o ticket automaticamente.
*   **Security Guidance:** Atua como um *Rule/Hook*, bloqueando mudanças em arquivos sensíveis (ex: configurações de CI/CD) sem aprovação humana explícita.
*   **Frontend Design:** Um plugin de *system prompt* que penaliza o modelo por usar designs genéricos (o "estilo Bootstrap de IA"), forçando o uso de sistemas de design específicos do projeto.

---

## 5. Implementação Prática: Criando uma Skill Customizada

Embora você possa escrever skills em bash, para integrações complexas (como consultar um banco de dados interno para dar contexto ao agente), construir um servidor MCP ou uma API de Tool Calling é o caminho ideal.

Abaixo, um exemplo de como implementar um endpoint de Skill usando **Python, FastAPI e Pydantic** que o Claude Code pode consumir. Esta skill permite que o agente consulte o esquema de um banco de dados local.

```python
# main.py
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel, Field
from sqlalchemy import create_engine, inspect
from sqlalchemy.orm import sessionmaker, Session
from typing import List, Dict, Any

# Configuração do Banco de Dados (SQLite para o exemplo, mas aplicável a Postgres via psycopg2)
DATABASE_URL = "sqlite:///./app.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

app = FastAPI(
    title="Claude Code DB Skill",
    description="Skill para permitir que o Claude Code inspecione esquemas de banco de dados.",
    version="1.0.0"
)

# --- Modelos Pydantic para a Interface da Skill ---

class SchemaQueryRequest(BaseModel):
    table_name: str = Field(..., description="Nome da tabela para inspecionar. Use 'all' para listar todas as tabelas.")

class ColumnInfo(BaseModel):
    name: str
    type: str
    primary_key: bool
    nullable: bool

class TableSchemaResponse(BaseModel):
    table_name: str
    columns: List[ColumnInfo]

# --- Dependências ---

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# --- Endpoints da Skill ---

@app.post("/skills/inspect-db", response_model=List[TableSchemaResponse])
def inspect_database_schema(
    request: SchemaQueryRequest, 
    db: Session = Depends(get_db)
) -> Any:
    """
    Endpoint que o Claude Code chamará via Tool Calling/MCP.
    Retorna o esquema do banco de dados para ajudar o agente a escrever queries SQL corretas.
    """
    inspector = inspect(engine)
    tables_to_inspect = inspector.get_table_names() if request.table_name == "all" else [request.table_name]
    
    response = []
    for table in tables_to_inspect:
        if not inspector.has_table(table):
            raise HTTPException(status_code=404, detail=f"Tabela '{table}' não encontrada.")
            
        columns = []
        for col in inspector.get_columns(table):
            columns.append(ColumnInfo(
                name=col["name"],
                type=str(col["type"]),
                primary_key=col.get("primary_key", False) == 1,
                nullable=col.get("nullable", True)
            ))
        
        response.append(TableSchemaResponse(table_name=table, columns=columns))
        
    return response

# Para rodar: uvicorn main:app --host 0.0.0.0 --port 8000
```

### Integrando a Skill no Claude Code

Para que o Claude Code utilize esta API, você a registraria no arquivo de configuração do projeto ou via MCP. Se for via MCP, você precisaria de um *wrapper* MCP ao redor desta API FastAPI (ou usar o SDK oficial do MCP para Python).

Se for via configuração direta de *Tools* (dependendo da versão do CLI):

```json
// claude-tools.json
{
  "tools": [
    {
      "name": "inspect_database_schema",
      "description": "Inspeciona o esquema do banco de dados local. Use isso antes de escrever queries SQL para garantir que as colunas existem.",
      "parameters": {
        "type": "object",
        "properties": {
          "table_name": {
            "type": "string",
            "description": "Nome da tabela, ou 'all' para todas."
          }
        },
        "required": ["table_name"]
      },
      "endpoint": "http://localhost:8000/skills/inspect-db"
    }
  ]
}
```

---

## 6. Padrões e Anti-padrões

### Padrões Recomendados
1.  **Fail-Fast em Skills:** Se uma skill falhar, retorne o erro detalhado (stderr ou JSON de erro) imediatamente para o LLM. O modelo é excelente em ler o erro e corrigir os parâmetros da próxima chamada.
2.  **Subagentes para Tarefas Determinísticas:** Não use o modelo principal (Opus/Sonnet 3.5) para formatar código ou rodar testes. Crie um subagente (usando Haiku ou um modelo local via Ollama) para tarefas de baixo nível.
3.  **Compressão de Contexto:** Use ferramentas como o `caveman-compress` para manter o `CLAUDE.md` enxuto. O LLM perde atenção no meio de contextos muito longos (o problema do *Lost in the Middle*).

### Anti-padrões
1.  **Permissões Globais (God Mode):** Dar ao agente acesso root ao terminal sem um mecanismo de *Human-in-the-Loop* (HITL) para comandos destrutivos (`rm`, `drop table`, `terraform apply`). Use plugins de *Security Guidance* para interceptar essas chamadas.
2.  **Over-prompting no System Prompt:** Tentar prever todos os erros possíveis no `CLAUDE.md`. Em vez disso, deixe o agente errar, ler o log de erro do compilador/linter e se corrigir.
3.  **Ignorar o Artifact Landing:** Deixar o agente gerar 500 linhas de código na resposta do chat e depois pedir para ele "salvar no arquivo". Ele deve usar uma tool `write_file` iterativamente.

---

## See Also

*   [[model-context-protocol]] - Detalhes profundos sobre a especificação MCP.
*   [[agentic-design-patterns]] - Padrões de arquitetura para agentes autônomos (ReAct, Plan-and-Solve).
*   [[llm-context-management]] - Estratégias para lidar com janelas de contexto limitadas e degradação de atenção.
*   [[fastapi-best-practices]] - Padrões para construir APIs robustas em Python.

## Sources

*   `raw/articles/firecrawl-claude-code-plugins.md`
*   `raw/articles/claude-marketplaces-directory.md`
*   `raw/articles/caveman-github-readme.md`
*   `raw/articles/superpowers-linkedin-post.md`
*   `raw/articles/agentic-ai-coding-config-study.md`