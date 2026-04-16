```yaml
domain: domains/ai/architecture/agent-skills.md
confidence: high
sources: 5
last_updated: 2024-05-24
```

# Agent Skills e Arquitetura de Agentes Generativos

A arquitetura de **Agent Skills** (Habilidades de Agentes) representa uma mudança de paradigma na engenharia de sistemas de Inteligência Artificial autônomos. Historicamente, o desenvolvimento de agentes baseava-se na criação de **agentes de domínio específico** (monolíticos), onde regras de negócio, *prompts* e integrações de ferramentas (*tools*) eram acoplados rigidamente (*hardcoded*) para resolver problemas restritos (ex: um agente exclusivo para análise financeira ou outro para engenharia de software).

A arquitetura moderna converge para o modelo de **Agentes de Propósito Geral equipados com Skills**. Neste paradigma, o agente base atua como um motor de raciocínio genérico, enquanto a *expertise* de domínio é injetada dinamicamente através de pacotes modulares de conhecimento procedural, denominados *Skills*. O código (frequentemente Python ou Bash) atua como a interface universal entre o agente e o ambiente digital.

## A Metáfora do "LLM OS"

A arquitetura baseada em *Skills* é frequentemente compreendida através da metáfora do "LLM OS" (Sistema Operacional baseado em LLM), popularizada por pesquisadores como Andrej Karpathy. Neste modelo, os componentes tradicionais de computação são mapeados para primitivas de IA:

1. **O Modelo (Processador/CPU):** O Large Language Model (ex: GPT-4, Claude 3) fornece a capacidade de raciocínio bruto, roteamento de lógica e geração de código. Por si só, é *stateless* (sem estado) e isolado.
2. **A Janela de Contexto (Memória RAM):** O espaço de trabalho temporário onde o agente mantém o histórico imediato, resultados de execuções e o estado atual da tarefa.
3. **O Agent Loop (Kernel):** O orquestrador central que gerencia o ciclo de vida da execução, tipicamente utilizando o padrão **ReAct** (*Reasoning and Acting*). Ele decide quando pensar, quando invocar uma ferramenta e quando retornar ao usuário.
4. **O Runtime Environment (Sistema Operacional):** Um ambiente isolado (*sandbox*, geralmente containers Docker ou microVMs) que fornece um sistema de arquivos efêmero e a capacidade de executar código gerado pelo LLM de forma segura.
5. **MCP Servers (Drivers/Periféricos):** O *Model Context Protocol* (MCP) atua como um padrão de "drivers", permitindo que o agente se conecte a recursos externos (bancos de dados PostgreSQL, APIs REST, navegadores web) de forma padronizada.
6. **Agent Skills (Aplicações/Userland):** Módulos de software que ensinam o agente a utilizar o *Runtime* e os *MCPs* para executar fluxos de trabalho específicos.

## Anatomia de uma Agent Skill

Diferente de um simples *prompt*, uma *Agent Skill* é um pacote de software estruturado, frequentemente mantido em controle de versão (Git) e baseado no sistema de arquivos. Uma *Skill* típica é composta por:

*   **System Prompts de Domínio:** Instruções contextuais que definem a persona, as restrições e as heurísticas específicas daquela habilidade.
*   **Tool Schemas (JSON/OpenAPI):** Definições formais das funções que o agente pode invocar.
*   **Código Procedural (Glue Code):** Scripts pré-escritos (ex: módulos Python ou Go) que o agente pode importar e executar dentro do seu *sandbox* para realizar tarefas complexas sem precisar escrever o código do zero.
*   **RAG de Documentação:** Um mini-índice vetorial ou arquivos Markdown contendo a documentação das APIs ou bibliotecas que a *Skill* utiliza, injetados no contexto apenas quando a *Skill* é ativada.

## Diagrama de Arquitetura

Abaixo, a topologia de um sistema de agente de propósito geral utilizando *Skills* e MCP:

```text
+-----------------------------------------------------------------------+
|                          AGENT KERNEL (ReAct Loop)                    |
|                                                                       |
|  +----------------+      +-----------------+      +----------------+  |
|  |  Memory/State  | <--> |  LLM (CPU)      | <--> | Context Window |  |
|  +----------------+      +-----------------+      +----------------+  |
+----------|------------------------|------------------------|----------+
           |                        |                        |
           v                        v                        v
+--------------------+   +--------------------+   +--------------------+
|   AGENT SKILLS     |   | RUNTIME (Sandbox)  |   |    MCP SERVERS     |
|   (User Apps)      |   | (OS Environment)   |   |    (Drivers)       |
|                    |   |                    |   |                    |
| - Data Analysis    |   | - Python Runtime   |   | - GitHub MCP       |
| - Web Scraping     |-->| - Bash/Shell       |-->| - PostgreSQL MCP   |
| - DevOps/K8s       |   | - Ephemeral FS     |   | - Slack API MCP    |
| - Code Review      |   |                    |   |                    |
+--------------------+   +--------------------+   +--------------------+
                                    |                        |
                                    v                        v
                         +---------------------------------------------+
                         |              MUNDO EXTERNO                  |
                         |  (Bancos de dados, APIs, Web, Filesystems)  |
                         +---------------------------------------------+
```

## Vantagens da Abordagem Baseada em Skills

1. **Composição Dinâmica:** Um agente pode carregar a *Skill* de "Engenharia de Software" para escrever código e, no passo seguinte, carregar a *Skill* de "DevOps" para fazer o *deploy* desse código, sem inchar o *System Prompt* inicial.
2. **Manutenibilidade:** Atualizações em uma API externa exigem apenas a modificação da *Skill* correspondente, sem alterar o *core* do agente.
3. **Segurança e Isolamento:** Como as *Skills* operam através da geração de código executado em um *sandbox*, o risco de injeção de prompt afetar sistemas críticos é mitigado pelas permissões do *Runtime* e do MCP.

## See Also

*   [Model Context Protocol (MCP)](mcp-protocol.md)
*   [ReAct (Reasoning and Acting)](react-pattern.md)
*   [Large Language Models (LLMs)](../models/llm-overview.md)
*   [Vector Databases e RAG](../data/rag-architecture.md)

## Sources

1.  Karpathy, A. (2023). *Intro to Large Language Models* (Conceito de LLM OS). YouTube.
2.  Anthropic. (2024). *Model Context Protocol Specification*. GitHub.
3.  Yao, S., et al. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. arXiv.
4.  Fowler, M. (2024). *Architecture of Autonomous AI Agents*. martinfowler.com.
5.  Discussões da comunidade técnica sobre padronização de ferramentas para agentes (Hacker News / r/MachineLearning).