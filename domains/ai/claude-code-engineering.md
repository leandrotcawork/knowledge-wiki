```yaml
domain: ai
confidence: high
sources: 5
last_updated: 2026-04-14
```

# Claude Code e Engenharia Agêntica

A **Engenharia Agêntica** (do inglês *Agentic Engineering*) é o paradigma de desenvolvimento de software focado na orquestração de sistemas probabilísticos autônomos (agentes de Inteligência Artificial) para a resolução de problemas complexos. O **Claude Code** é a interface de linha de comando (CLI) agêntica desenvolvida pela Anthropic, projetada para operar diretamente no sistema de arquivos local, executar comandos de terminal, interagir com APIs e iterar sobre bases de código de forma autônoma.

Diferente de assistentes de código baseados em autocompletar (como o GitHub Copilot tradicional), o Claude Code opera em um loop de *ReAct* (Reasoning and Acting), permitindo que ele planeje, execute, verifique erros de compilação/testes e corrija o próprio código sem intervenção humana contínua.

Este documento detalha a arquitetura, os princípios e as melhores práticas para estruturar repositórios e fluxos de trabalho otimizados para o Claude Code em ambientes de produção.

---

## 1. Princípios da Engenharia Agêntica

A transição da engenharia de software imperativa para a agêntica exige a adaptação da arquitetura do projeto para acomodar as limitações e os pontos fortes dos Modelos de Linguagem de Grande Escala (LLMs).

### 1.1. Gerenciamento Agressivo de Contexto
A **Janela de Contexto** (Context Window) atua como a memória RAM do agente. A degradação de performance (alucinações, esquecimento de restrições arquiteturais) é diretamente proporcional à saturação do contexto.
*   **Isolamento:** Evite carregar o repositório inteiro. O Claude Code deve ser instruído a usar ferramentas de busca (como `grep` ou `ast-grep`) para localizar apenas os nós relevantes da árvore de sintaxe abstrata (AST).
*   **Limpeza de Sessão:** Sessões longas acumulam histórico de tentativas falhas. Reiniciar a sessão do Claude Code após a conclusão de uma subtarefa (commit) é uma prática recomendada para restaurar a precisão do modelo.

### 1.2. Verificação Empírica sobre Geração
Agentes LLM são geradores probabilísticos; eles não possuem compreensão determinística de que um código funciona até que ele seja executado.
*   **Feedback Loops:** O Claude Code deve ter acesso a linters, formatadores e suítes de testes automatizados.
*   **TDD Agêntico:** A abordagem mais eficaz é o humano escrever os testes (ou aprovar testes gerados) e delegar ao Claude Code a tarefa de "fazer os testes passarem", utilizando o erro do compilador/interpretador como *prompt* implícito para a próxima iteração.

### 1.3. Divulgação Progressiva (Progressive Disclosure)
Em vez de injetar toda a documentação do domínio no prompt inicial do sistema, repositórios devem ser estruturados para que o agente *descubra* as regras sob demanda.
*   Uso de arquivos `.clauderc` ou `AGENT_INSTRUCTIONS.md` na raiz do projeto contendo apenas ponteiros para documentações mais profundas (ex: "Para regras de banco de dados, leia `docs/db_schema.md`").

---

## 2. Arquitetura de Repositórios Orientada a Agentes

Para que o Claude Code opere com máxima eficiência, a base de código deve ser "legível por máquinas" não apenas no nível do compilador, mas no nível semântico.

### 2.1. Modularidade e Tamanho de Arquivos
Arquivos monolíticos (ex: `utils.ts` com 3.000 linhas) esgotam o contexto rapidamente e aumentam o custo de tokens por iteração.
*   Adote o princípio de Responsabilidade Única (SRP) de forma estrita.
*   Mantenha arquivos curtos (idealmente < 300 linhas).

### 2.2. Tipagem Forte e Contratos Claros
O Claude Code depende de assinaturas de funções e tipagem estática para inferir o uso correto de módulos que não estão em seu contexto imediato.
*   Linguagens com tipagem estática forte (Go, Rust, TypeScript) apresentam taxas de sucesso agêntico significativamente maiores do que linguagens dinâmicas (Python sem Type Hints, JavaScript puro).
*   Documentação inline (JSDoc, Godoc, Docstrings) deve focar no *porquê* e nas *restrições* (ex: "Esta função não é thread-safe"), pois o agente já compreende o *o quê* lendo o código.

### 2.3. Padronização de Ferramentas (Toolchain)
O agente precisa saber como construir e testar o projeto sem adivinhar comandos.
*   Utilize `Makefiles`, `package.json` (scripts) ou `Taskfiles` padronizados.
*   Exemplo de instrução de sistema: *"Sempre execute `make test` após modificar arquivos em `/src`."*

---

## 3. Mecânica de Execução e Segurança

O Claude Code possui a capacidade de executar comandos de terminal (`bash`/`zsh`). Isso introduz vetores de risco significativos.

### 3.1. O Loop de Execução (OODA/ReAct)
1.  **Observe:** O Claude Code lê o prompt do usuário e o estado atual do repositório.
2.  **Orient:** Planeja quais arquivos precisam ser lidos ou modificados.
3.  **Decide:** Seleciona a ferramenta apropriada (ex: `read_file`, `edit_file`, `run_command`).
4.  **Act:** Executa a ação. Se for um comando de terminal, ele analisa o `stdout`/`stderr` e reinicia o loop caso haja erros.

### 3.2. Sandboxing e Permissões
Nunca execute o Claude Code com privilégios de superusuário (`root`) ou em ambientes de produção com acesso irrestrito a bancos de dados reais.
*   **Containers:** Recomenda-se rodar o Claude Code dentro de DevContainers (Docker) para isolar o sistema de arquivos do host.
*   **Variáveis de Ambiente:** Mascare ou utilize cofres de segredos (secrets vaults) para chaves de API. O agente pode acidentalmente imprimir ou logar variáveis de ambiente durante o processo de depuração.

---

## 4. Padrões de Fluxo de Trabalho (Workflows)

### 4.1. O Padrão "Supervisor-Trabalhador"
O engenheiro humano atua como Supervisor (Product Manager / Arquiteto), enquanto o Claude Code atua como Trabalhador (Desenvolvedor).
1.  O humano define a arquitetura e escreve os esqueletos das interfaces/tipos.
2.  O humano invoca o Claude Code: `claude "Implemente a interface X no arquivo Y. Use a biblioteca Z. Rode os testes ao finalizar."`
3.  O humano revisa o `git diff` antes de aceitar as mudanças.

### 4.2. Isolamento de Falhas (Git Checkpoints)
Agentes podem entrar em "loops de alucinação", onde tentam consertar um erro criando refatorações em cascata que quebram o sistema inteiro.
*   **Regra de Ouro:** Faça um commit no Git *antes* de invocar o Claude Code para qualquer tarefa complexa.
*   Instrua o agente a criar branches temporárias para experimentos arquiteturais.

---

## See Also

*   **ReAct Prompting:** O paradigma de raciocínio e ação que fundamenta agentes de CLI.
*   **Test-Driven Development (TDD):** Metodologia que se tornou o mecanismo de validação primário para código gerado por IA.
*   **DevContainers:** Tecnologia de conteinerização essencial para o sandboxing de agentes locais.
*   **Large Language Model Operations (LLMOps):** Práticas de gerenciamento de ciclo de vida de modelos e agentes.

---

## Sources

1.  **Karpathy, Andrej.** *Intro to Large Language Models* e discussões sobre o "LLM OS" (Analogia do Contexto como Memória RAM).
2.  **Fowler, Martin.** *martinfowler.com* - Princípios de Refatoração e TDD (aplicados à verificação empírica de código gerado).
3.  **The Pragmatic Engineer (Gergely Orosz).** Análises sobre o impacto de assistentes de IA e agentes autônomos no fluxo de trabalho de engenharia de software.
4.  **Hacker News (news.ycombinator.com).** Discussões agregadas da comunidade sobre melhores práticas, falhas de segurança e sandboxing de ferramentas CLI agênticas (como Claude Code, Aider, etc).
5.  **Documentação Oficial (Go, TypeScript, Python).** Padrões de tipagem estática e design de repositórios que facilitam a análise estática por ferramentas automatizadas e LLMs.