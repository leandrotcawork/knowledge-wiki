---
domain: domains/ai/architecture/agent-skills.md
confidence: low
sources: 1
last_updated: 2026-04-14
---

# Agent Skills e Arquitetura de Agentes Generativos

A engenharia de agentes autônomos passou por uma mudança de paradigma fundamental. Inicialmente, a abordagem padrão era construir **agentes de domínio específico** — um agente para finanças, outro para engenharia de software, outro para RH — cada um com seu próprio scaffolding, prompts e ferramentas hardcoded. 

A arquitetura moderna converge para um modelo diferente: **Agentes de Propósito Geral equipados com Skills**. 

Neste modelo, o código é a interface universal para o mundo digital. O agente base é genérico (um loop de raciocínio acoplado a um ambiente de execução de código), e a *expertise de domínio* é injetada dinamicamente através de **Agent Skills** — pacotes modulares de conhecimento procedural baseados em sistema de arquivos.

## A Arquitetura do "LLM OS"

Para entender o papel das Skills, precisamos olhar para a arquitetura emergente de agentes como um sistema operacional.

1. **O Modelo (Processador):** O LLM (ex: Claude, GPT-4) fornece a capacidade de raciocínio bruto e manipulação de linguagem/código. Sozinho, é stateless e isolado.
2. **O Agent Loop (Kernel):** Gerencia o estado, a memória, o roteamento de tokens e o ciclo de ReAct (Reasoning and Acting).
3. **O Runtime Environment (Sistema Operacional):** Um ambiente isolado (sandbox) que fornece um sistema de arquivos e a capacidade de executar código (Python, Bash).
4. **MCP Servers (Drivers/Periféricos):** O *Model Context Protocol* (MCP) padroniza a conexão com o mundo externo (APIs, bancos de dados, navegadores).
5. **Agent Skills (Aplicações):** O conhecimento procedural e as regras de negócio que ensinam o agente a usar o Runtime e os MCPs para resolver problemas específicos.

### Diagrama de Arquitetura

```text
                               +---------------------------------------+
                               |        Agent Loop (Orquestrador)      |
                               | - Context Management                  |
                               | - Token Routing                       |
                               +---------------------------------------+
                                     ^                           |
                                     | (Prompt/History)          | (Tool Calls/Code)
                                     v                           v
+-------------------+      +-----------------------------------------------+
|                   |      | Runtime Environment (Sandbox / ex: E2B)       |
|   LLM (Cérebro)   | <--> | - File System (Workspace)                     |
|                   |      | - Code Execution Engine (Python, Node, Bash)  |
+-------------------+      +-----------------------------------------------+
                                     |                           |
        +----------------------------+                           +-----------------------------+
        |                                                                                      |
        v                                                                                      v
+------------------------------------------------+      +------------------------------------------------+
| Agent Skills (Expertise / Conhecimento)        |      | MCP Servers (I/O / Conectividade Externa)      |
| - skill_analise_financeira/                    |      | - mcp-github (Leitura/Escrita de PRs)          |
|   |- skill.md (Instruções core)                |      | - mcp-notion (Acesso a base de conhecimento)   |
|   |- format_report.py (Script utilitário)      |      | - mcp-postgres (Acesso a dados estruturados)   |
| - skill_code_review/                           |      | - mcp-browserbase (Automação web)              |
+------------------------------------------------+      +------------------------------------------------+
```

## O que são Agent Skills?

**Skills são coleções organizadas de arquivos que empacotam conhecimento procedural combinável para agentes.** 

Em termos práticos: **Skills são pastas.**

Essa simplicidade é intencional. Ao usar o sistema de arquivos como primitiva, as Skills herdam décadas de tooling de engenharia de software: podem ser versionadas no Git, zipadas, compartilhadas e testadas via CI/CD.

### Skills vs. Ferramentas Tradicionais (Tools)

A abordagem tradicional de "Tools" (ex: OpenAI Function Calling) injeta schemas JSON massivos no *System Prompt*. Isso apresenta três problemas críticos:
1. **Poluição de Contexto:** Injetar 100 ferramentas consome milhares de tokens e degrada a capacidade de atenção do modelo (Lost in the Middle).
2. **Rigidez:** Se a ferramenta falha ou a instrução é ambígua, o modelo não pode modificar o código da ferramenta em tempo de execução.
3. **Cold Start:** O modelo precisa inferir como usar a ferramenta apenas pela sua assinatura, sem exemplos ricos ou scripts auxiliares.

**Skills resolvem isso movendo as ferramentas para o sistema de arquivos.** Como o agente tem acesso a um Runtime com execução de código, uma Skill pode conter scripts Python complexos que o agente invoca via shell, em vez de depender de endpoints de API hardcoded no prompt.

### A Estrutura de uma Skill

Uma Skill típica contém metadados, instruções em Markdown e scripts utilitários.

```text
skills/
└── relatorio_tributario_br/
    ├── metadata.json       # Descrição curta para descoberta
    ├── skill.md            # Instruções detalhadas, edge cases, heurísticas
    ├── templates/          # Assets estáticos (ex: template_dre.docx)
    └── utils/
        └── calc_irpj.py    # Script Python que o agente pode executar ou modificar
```

## Progressive Disclosure (Descoberta Progressiva)

Para suportar milhares de Skills sem estourar a janela de contexto, a arquitetura utiliza **Progressive Disclosure** (Divulgação Progressiva).

1. **Fase de Descoberta:** No início do loop, o agente recebe apenas um índice leve (metadados) das Skills disponíveis.
2. **Fase de Carregamento:** Quando o agente decide que precisa de uma Skill específica para a tarefa atual, ele usa uma ferramenta do sistema para ler o `skill.md` correspondente.
3. **Fase de Execução:** O agente lê as instruções detalhadas e executa os scripts contidos na pasta da Skill.

### Implementação: Gerenciamento de Skills em Python

Abaixo, um padrão de implementação usando `Pydantic` para gerenciar o carregamento progressivo de Skills em um Agent Loop.

```python
import os
import json
from typing import List, Dict, Optional
from pydantic import BaseModel, Field

class SkillMetadata(BaseModel):
    id: str
    name: str
    description: str
    triggers: List[str] = Field(description="Palavras-chave ou intenções que ativam esta skill")

class SkillRegistry:
    def __init__(self, skills_dir: str):
        self.skills_dir = skills_dir
        self._index: Dict[str, SkillMetadata] = {}
        self._build_index()

    def _build_index(self):
        """Constrói o índice leve para a Fase de Descoberta."""
        for skill_id in os.listdir(self.skills_dir):
            meta_path = os.path.join(self.skills_dir, skill_id, "metadata.json")
            if os.path.exists(meta_path):
                with open(meta_path, 'r') as f:
                    data = json.load(f)
                    self._index[skill_id] = SkillMetadata(**data)

    def get_available_skills_summary(self) -> str:
        """Injetado no System Prompt inicial."""
        summary = "Available Skills (Load them using `load_skill` tool if needed):\n"
        for skill in self._index.values():
            summary += f"- {skill.id}: {skill.description}\n"
        return summary

    def load_skill_content(self, skill_id: str) -> str:
        """Ferramenta chamada pelo LLM para ler o conteúdo profundo da skill."""
        if skill_id not in self._index:
            return f"Error: Skill {skill_id} not found."
        
        skill_md_path = os.path.join(self.skills_dir, skill_id, "skill.md")
        try:
            with open(skill_md_path, 'r') as f:
                return f.read()
        except FileNotFoundError:
            return "Error: skill.md missing."

# Exemplo de uso no Agent Loop
registry = SkillRegistry("/var/agent/skills")
system_prompt = f"""
You are a general purpose agent with access to a sandboxed file system and code execution.
{registry.get_available_skills_summary()}
"""
```

## MCP vs. Skills: A Fronteira de Responsabilidade

É crucial não confundir MCP (Model Context Protocol) com Agent Skills. Eles são complementares, mas resolvem problemas diferentes.

*   **MCP Servers fornecem Conectividade (O "O Quê"):** Eles expõem dados e ações de sistemas externos de forma padronizada. Exemplo: Um MCP do GitHub permite ler e criar Pull Requests. O MCP não sabe *como* fazer um bom code review.
*   **Skills fornecem Expertise (O "Como"):** Uma Skill de "Code Review Sênior" contém as heurísticas da empresa (ex: "Sempre verifique vazamento de memória em loops Go", "Garanta que a cobertura de testes não caia"). A Skill orquestra o MCP do GitHub para baixar o código, usa o Runtime para rodar linters locais, e usa o MCP novamente para postar os comentários.

## Continuous Learning e Evolução

O aspecto mais poderoso de modelar Skills como arquivos é o **Continuous Learning** (Aprendizado Contínuo). 

Agentes tradicionais são amnésicos; eles começam do zero a cada sessão. Com Skills, o agente pode ser instruído a **escrever suas próprias Skills**.

Se você pede repetidamente para o agente formatar slides de uma maneira específica, você pode dizer: *"Salve esse script Python e essas regras de formatação como uma nova Skill chamada `formatacao_slides_acme`"*. 

O agente cria a pasta, escreve o `metadata.json`, o `skill.md` e o script Python. Na próxima sessão (Dia 30), o agente carrega essa Skill e executa a tarefa perfeitamente, sem precisar deduzir os princípios do zero. O agente no Dia 30 é fundamentalmente mais capaz que o agente no Dia 1.

## Considerações de Segurança e Infraestrutura

Como as Skills dependem fortemente da execução de código gerado pelo LLM (e de scripts contidos nas próprias Skills), a segurança do Runtime Environment é inegociável.

1. **Sandboxing Estrito:** Nunca execute o Runtime do agente na máquina host ou em containers Docker não privilegiados sem isolamento adicional. Use microVMs (ex: AWS Firecracker) ou plataformas de sandboxing focadas em IA (ex: E2B, Daytona).
2. **Network Egress Control:** O ambiente de execução de código deve ter acesso de rede restrito. O agente deve se comunicar com o mundo externo preferencialmente através de MCP Servers auditáveis, e não fazendo `curl` arbitrário de dentro do script Python da Skill, a menos que explicitamente permitido.
3. **Skill Lineage e Versionamento:** Trate Skills como código de produção. Elas devem residir em repositórios Git. Mudanças em `skill.md` devem passar por revisão (humana ou automatizada via LLM-as-a-Judge) para garantir que a alteração de uma heurística não degrade o comportamento do agente em outras tarefas.

## See Also

*   [[model-context-protocol]]
*   [[react-agent-loop]]
*   [[llm-sandboxing-e2b]]
*   [[function-calling-vs-tools]]

## Sources

*   raw/transcripts/anthropic-agent-skills-talk.md