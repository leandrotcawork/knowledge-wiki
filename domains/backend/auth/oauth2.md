---
domain: domains/backend/auth/oauth2.md
confidence: medium
sources: 4
last_updated: 2026-04-13
---

O **OAuth2** (Open Authorization 2.0) é um framework de autorização padrão da indústria que permite a aplicações de terceiros obter acesso limitado a recursos de um usuário hospedados em um serviço HTTP, sem a necessidade de expor as credenciais (como usuário e senha) do proprietário do recurso. Ao desacoplar a autenticação da autorização, o OAuth2 fornece uma maneira segura e padronizada de delegar acesso, mitigando riscos de segurança, aumentando o controle do usuário e resolvendo problemas de escalabilidade no gerenciamento de credenciais.

## Atores do OAuth2

O fluxo de autorização do OAuth2 é orquestrado pela interação de quatro atores principais (roles) definidos na especificação:

*   **Proprietário do Recurso (Resource Owner):** Geralmente o usuário final. É a entidade capaz de conceder acesso a um recurso protegido (como dados de perfil, arquivos ou e-mails). O proprietário tem controle total sobre quem pode acessar sua conta e qual o nível de acesso concedido (escopo).
*   **Cliente (OAuth2 Client):** A aplicação (web, mobile, desktop ou dispositivo conectado) que requer acesso aos recursos protegidos em nome do proprietário do recurso. Para acessar os dados, o cliente deve primeiro obter um *Access Token* (token de acesso).
*   **Servidor de Recursos (Resource Server):** O servidor (frequentemente uma API) que hospeda e protege os recursos do usuário. Ele recebe as requisições do cliente, valida o token de acesso apresentado e, se autorizado, retorna os recursos solicitados.
*   **Servidor de Autorização (Authorization Server):** O servidor responsável por autenticar o usuário, obter seu consentimento e emitir os tokens de acesso para o cliente. Ele expõe endpoints críticos, como o *Authorization Endpoint* (para interação com o usuário) e o *Token Endpoint* (para emissão de tokens). Em muitos ecossistemas, atua em conjunto com um Provedor de Identidade (IdP).

## Tipos de Concessão (Grant Types)

O OAuth2 define diferentes "tipos de concessão" (fluxos) para acomodar variados casos de uso e arquiteturas de clientes. A escolha do fluxo depende da natureza da aplicação (ex: server-side, client-side, máquina-a-máquina).

### Fluxos Atuais e Recomendados

*   **Authorization Code Grant:** Utilizado principalmente por aplicações web tradicionais (server-side) e aplicações nativas/mobile. Neste fluxo, o cliente redireciona o usuário para o servidor de autorização. Após o consentimento, o cliente recebe um "código de autorização" temporário, que é posteriormente trocado por um token de acesso em uma comunicação segura de back-end (server-to-server).
*   **Client Credentials Grant:** Utilizado para comunicação sistema-a-sistema (máquina-a-máquina), onde não há envolvimento de um usuário final. O cliente solicita um token de acesso diretamente ao servidor de autorização usando suas próprias credenciais.

### Fluxos Descontinuados (Deprecated)

Com a evolução das práticas de segurança e a publicação da **RFC 9700** (OAuth 2.0 Security Best Current Practice), alguns fluxos originais foram considerados inseguros e estão oficialmente descontinuados:

*   **Implicit Grant:** Anteriormente usado para aplicações client-side (como Single Page Applications). Neste fluxo, o token de acesso era retornado diretamente na URL de redirecionamento, sem a etapa de troca de código. Foi descontinuado devido a vulnerabilidades de vazamento de tokens no histórico do navegador e interceptação.
*   **Resource Owner Password Credentials Grant:** Permitia que o cliente coletasse diretamente o usuário e a senha do proprietário do recurso para trocá-los por um token. Este fluxo quebra o princípio fundamental do OAuth2 de não compartilhar credenciais com terceiros e foi totalmente descontinuado.

## Segurança e Melhores Práticas (RFC 9700)

A **RFC 9700** (também conhecida como BCP 240, publicada em janeiro de 2025) atualiza as especificações originais (RFC 6749, RFC 6750 e RFC 6819) para incorporar experiências práticas e mitigar novas ameaças. As principais recomendações de segurança incluem:

*   **Proteção de Fluxos Baseados em Redirecionamento:** Exigência de validação estrita das URIs de redirecionamento (Redirect URI) para prevenir ataques de injeção e roubo de códigos de autorização.
*   **Prevenção contra Replay de Tokens:** Implementação de mecanismos para garantir que *Access Tokens* e *Refresh Tokens* não possam ser reutilizados maliciosamente caso sejam interceptados.
*   **Restrição de Privilégios (Least Privilege):** Os tokens de acesso devem ter seus privilégios e escopos estritamente limitados ao necessário para a operação, reduzindo o impacto em caso de vazamento.
*   **Autenticação Forte de Clientes:** Recomendação de métodos robustos para a autenticação do cliente junto ao servidor de autorização durante a troca de tokens.

## Veja Também

*   [[OpenID Connect]]
*   [[JSON Web Token (JWT)]]
*   [[Autenticação vs Autorização]]
*   [[Single Sign-On (SSO)]]

## Fontes

*   `raw/articles/oauth2-overview-devto.md`
*   `raw/articles/oauth2-concepts-ory.md`
*   `raw/articles/oauth2-best-practice-oauthnet.md`
*   `raw/articles/oauth2-rfc9700.md`