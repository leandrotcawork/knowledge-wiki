```yaml
---
domain: domains/backend/auth/oauth2.md
confidence: high
sources: 6
last_updated: 2026-04-13
---
```

# OAuth 2.0

**OAuth 2.0** (Open Authorization) é um framework padrão da indústria para delegação de autorização, definido primariamente pela **RFC 6749**. Ele permite que uma aplicação de terceiros obtenha acesso limitado a um serviço HTTP em nome do proprietário do recurso (usuário) ou em seu próprio nome, orquestrando uma interação de aprovação entre o proprietário e o serviço HTTP.

O objetivo principal do OAuth 2.0 é eliminar a necessidade de os usuários compartilharem suas credenciais (como usuário e senha) diretamente com aplicações de terceiros.

**Aviso Arquitetural Crítico:** OAuth 2.0 **não** é um protocolo de autenticação. Ele foi desenhado exclusivamente para *autorização* (o que a aplicação pode fazer), não para provar a identidade do usuário (quem o usuário é). Para casos de uso de autenticação, a indústria utiliza o **OpenID Connect (OIDC)**, uma camada de identidade construída sobre o OAuth 2.0.

---

## 1. Atores e Componentes (Roles)

O ecossistema OAuth 2.0 é estritamente dividido em quatro papéis fundamentais:

1. **Resource Owner (Proprietário do Recurso):** A entidade capaz de conceder acesso a um recurso protegido. Quando o proprietário é uma pessoa, é referido como usuário final.
2. **Client (Cliente):** A aplicação que requer acesso ao recurso protegido em nome do *Resource Owner*. Os clientes são classificados em dois tipos baseados em sua capacidade de manter credenciais seguras:
   * **Confidential Clients:** Aplicações que podem manter o sigilo de suas credenciais (ex: aplicações web tradicionais com backend em Node.js, Go, Python).
   * **Public Clients:** Aplicações incapazes de manter credenciais seguras (ex: Single Page Applications - SPAs em React/Vue, ou aplicativos mobile nativos).
3. **Authorization Server (Servidor de Autorização):** O servidor que autentica o *Resource Owner*, obtém seu consentimento e emite os tokens de acesso para o *Client*. (ex: Auth0, Keycloak, AWS Cognito).
4. **Resource Server (Servidor de Recursos):** O servidor que hospeda os recursos protegidos (APIs) e é capaz de aceitar e responder a requisições que contenham um *Access Token* válido.

---

## 2. Tokens e Escopos

O OAuth 2.0 utiliza tokens para representar a autorização concedida.

* **Access Token (Token de Acesso):** Uma credencial de curta duração usada para acessar recursos protegidos. Pode ser um token opaco (uma string aleatória validada via introspecção no Authorization Server) ou um token estruturado, como um **JWT (JSON Web Token)**.
* **Refresh Token (Token de Atualização):** Uma credencial de longa duração opcional, usada pelo *Client* para obter novos *Access Tokens* quando o atual expira, sem exigir nova interação do usuário.
* **Scopes (Escopos):** Mecanismo para limitar o acesso concedido. O *Client* solicita escopos específicos (ex: `read:email`, `write:files`), e o *Access Token* emitido será restrito apenas a essas permissões.

---

## 3. Fluxos de Autorização (Grant Types)

O OAuth 2.0 define diferentes métodos (fluxos) para um *Client* obter um *Access Token*, dependendo do tipo de cliente e do caso de uso.

### 3.1. Authorization Code Flow (Fluxo de Código de Autorização)
O fluxo mais comum e seguro, ideal para **Confidential Clients**.
1. O *Client* redireciona o usuário para o *Authorization Server*.
2. O usuário se autentica e aprova o acesso.
3. O *Authorization Server* redireciona o usuário de volta ao *Client* com um `code` (código de autorização) de uso único e curta duração.
4. O *Client* (no backend) troca este `code` e seu `client_secret` por um *Access Token* diretamente com o *Authorization Server*.

### 3.2. Authorization Code com PKCE
Extensão do fluxo anterior (RFC 7636), obrigatória para **Public Clients** (SPAs, Mobile) que não possuem um `client_secret`.
* Utiliza o **PKCE (Proof Key for Code Exchange)**. O *Client* gera um `code_verifier` (string aleatória) e envia seu hash (`code_challenge`) na requisição inicial.
* Ao trocar o código pelo token, o *Client* envia o `code_verifier` original. O servidor valida o hash, garantindo que quem iniciou o fluxo é o mesmo que o está concluindo, mitigando ataques de interceptação de código.

### 3.3. Client Credentials Flow (Credenciais do Cliente)
Utilizado para comunicação **Machine-to-Machine (M2M)**, onde não há um usuário final envolvido. O *Client* usa suas próprias credenciais (`client_id` e `client_secret`) para obter um *Access Token* e acessar recursos sob seu próprio controle.

### 3.4. Fluxos Obsoletos (Deprecated)
As melhores práticas atuais (e o rascunho do OAuth 2.1) desencorajam fortemente o uso dos seguintes fluxos devido a vulnerabilidades de segurança:
* **Implicit Flow:** Retornava o token diretamente na URL. Substituído pelo *Authorization Code com PKCE*.
* **Resource Owner Password Credentials (ROPC):** O *Client* coletava a senha do usuário diretamente. Viola o princípio fundamental do OAuth de não compartilhar credenciais com terceiros.

---

## 4. Considerações de Segurança

* **TLS/HTTPS Obrigatório:** Como os tokens são credenciais de portador (Bearer tokens), qualquer pessoa que os possua pode usá-los. O tráfego deve ser sempre criptografado.
* **Proteção contra CSRF:** O parâmetro `state` deve ser utilizado nas requisições de autorização para manter o estado entre a requisição e o callback, prevenindo ataques de *Cross-Site Request Forgery*.
* **Armazenamento de Tokens:** Em aplicações web, recomenda-se armazenar tokens no backend (BFF - Backend for Frontend) e usar cookies `HttpOnly` para a sessão do navegador, mitigando ataques XSS (Cross-Site Scripting) que poderiam roubar tokens do `localStorage`.

---

## See Also

* [[OpenID Connect (OIDC)]] - Camada de identidade sobre o OAuth 2.0.
* [[JSON Web Tokens (JWT)]] - Formato comum para Access Tokens.
* [[Autenticação vs Autorização]] - Diferenças conceituais fundamentais.
* [[Single Sign-On (SSO)]] - Padrão de arquitetura frequentemente implementado via OIDC/OAuth2.

---

## Sources

1. **IETF RFC 6749:** *The OAuth 2.0 Authorization Framework*. A especificação oficial do protocolo.
2. **IETF RFC 7636:** *Proof Key for Code Exchange by OAuth Public Clients (PKCE)*.
3. **MDN Web Docs:** *Authorization and Authentication*.
4. **Martin Fowler:** *Securing Microservices with OAuth2*. Discussões arquiteturais sobre delegação de tokens em sistemas distribuídos.
5. **Auth0 / Okta Developer Documentation:** Guias de implementação e melhores práticas da indústria para fluxos OAuth 2.0.
6. **OAuth 2.0 Security Best Current Practice (IETF Draft):** Documento em evolução que define a obsolescência do fluxo Implícito e a obrigatoriedade do PKCE.