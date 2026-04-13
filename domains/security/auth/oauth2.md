---
domain: domains/security/auth/oauth2.md
confidence: low
sources: 3
last_updated: 2026-04-13
---

# OAuth 2.0

OAuth 2.0 (Open Authorization) é um framework de delegação de autorização padrão da indústria. Ele permite que aplicações de terceiros obtenham acesso limitado a um serviço HTTP em nome de um proprietário de recurso (usuário) ou em seu próprio nome. 

Embora originalmente definido na RFC 6749 e RFC 6750, o ecossistema OAuth 2.0 evoluiu significativamente. A **RFC 9700 (OAuth 2.0 Security Best Current Practice - BCP 240)** atualiza o modelo de ameaças e deprecia fluxos inteiros (como o *Implicit Grant*) em favor de padrões mais rígidos, como o uso universal de PKCE e validação estrita de URIs.

## Atores (Roles)

O ecossistema OAuth 2.0 é composto por quatro atores fundamentais:

1. **Resource Owner (Proprietário do Recurso)**: A entidade capaz de conceder acesso a um recurso protegido. Geralmente, é o usuário final.
2. **Resource Server (Servidor de Recursos - RS)**: O servidor que hospeda os recursos protegidos, capaz de aceitar e responder a requisições de recursos usando *Access Tokens*.
3. **Client (Cliente)**: A aplicação que faz requisições de recursos protegidos em nome do Resource Owner. Pode ser confidencial (capaz de manter credenciais seguras, como um backend) ou público (SPAs, apps nativos).
4. **Authorization Server (Servidor de Autorização - AS)**: O servidor que emite *Access Tokens* para o Client após autenticar o Resource Owner e obter sua autorização.

## Anatomia dos Tokens

OAuth 2.0 utiliza primariamente dois tipos de tokens:

*   **Access Token**: Credencial usada para acessar recursos protegidos. Representa a autorização concedida. Historicamente, são *Bearer Tokens* (quem o possui, pode usá-lo), mas as melhores práticas atuais (RFC 9700) recomendam **Sender-Constrained Tokens** (ex: via mTLS ou DPoP - Demonstrating Proof-of-Possession) para evitar ataques de *Token Replay* caso o token seja vazado.
*   **Refresh Token**: Credencial usada para obter novos Access Tokens quando o atual expira, sem necessidade de nova interação do usuário. Para clientes públicos, a RFC 9700 exige que utilizem *Refresh Token Rotation* ou sejam *Sender-Constrained*.

### Restrição de Privilégios (Audience e Scopes)
Tokens devem seguir o princípio do menor privilégio. Além dos `scopes` (que definem *o que* pode ser feito), a RFC 9700 recomenda fortemente o uso de **Audience-Restricted Access Tokens**. O AS associa o token a um RS específico (ou pequeno grupo). O RS deve validar se o token foi emitido para ele, mitigando o impacto caso um RS seja comprometido e tente usar o token em outro serviço.

## Fluxos de Concessão (Grant Types)

### 1. Authorization Code Grant com PKCE (Padrão Ouro)

O fluxo de Código de Autorização é o fluxo primário do OAuth 2.0. A RFC 9700 **exige** que a extensão PKCE (Proof Key for Code Exchange - RFC 7636) seja utilizada por **todos** os clientes (públicos e confidenciais) para prevenir injeção de código de autorização e CSRF.

#### Como o PKCE funciona internamente:
1. O Client gera um `code_verifier` (uma string aleatória de alta entropia).
2. O Client calcula o `code_challenge` usando SHA-256: `BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))`. O método `S256` é obrigatório; o método `plain` não deve ser usado.
3. O `code_challenge` é enviado na URL de autorização.
4. Ao trocar o código pelo token, o Client envia o `code_verifier` original. O AS refaz o hash e compara. Se bater, o cliente é legítimo.

#### Diagrama de Arquitetura (Authorization Code + PKCE)

```text
+----------+
| Resource |
|  Owner   |
| (Usuário)|
+----------+
     |
    (A) Inicia fluxo de login/autorização
     v
+----------+                                +---------------+
|          |--(B) Redireciona para o AS --->|               |
|  Client  |      (client_id, state,        | Authorization |
|  (App)   |       code_challenge, S256)    |    Server     |
|          |<-(C) Redireciona de volta -----|     (AS)      |
+----------+      (code, state, iss)        |               |
     |                                      +---------------+
    (D) POST /token (code, code_verifier)           ^
     |----------------------------------------------|
     |<-(E) Access Token (+ Refresh Token) ---------|
     v
+----------+                                +---------------+
|          |--(F) GET /api (Access Token) ->|   Resource    |
|  Client  |                                |    Server     |
|          |<-(G) Dados Protegidos ---------|     (RS)      |
+----------+                                +---------------+
```

### 2. Client Credentials Grant

Usado para comunicação máquina-a-máquina (M2M), onde o Client acessa recursos sob seu próprio controle, sem um Resource Owner humano. Comum em microsserviços ou bots (ex: bots do Discord acessando APIs administrativas). O Client autentica-se diretamente no `/token` endpoint usando `client_id` e `client_secret` (ou mTLS/JWT assertions).

### Fluxos Depreciados (RFC 9700)

*   **Implicit Grant (`response_type=token`)**: **DEPRECIADO**. Retornava o Access Token diretamente no fragmento da URL de redirecionamento. Vulnerável a vazamento via histórico do navegador, cabeçalhos `Referer` e injeção de token. Deve ser substituído pelo Authorization Code com PKCE.
*   **Resource Owner Password Credentials**: **DEPRECIADO**. Exigia que o Client coletasse a senha do usuário em texto plano. Quebra o paradigma de delegação e expõe credenciais.

## Segurança e Melhores Práticas (RFC 9700 / BCP 240)

A implementação de OAuth 2.0 moderna exige defesas em profundidade contra vetores de ataque sofisticados.

### Validação Estrita de Redirect URI
Ataques de *Open Redirection* permitem que invasores roubem códigos de autorização.
*   **Regra**: O AS **DEVE** utilizar correspondência exata de strings (Exact String Matching) ao comparar a URI de redirecionamento da requisição com a URI pré-registrada.
*   **Exceção**: Apenas a porta pode variar para URIs `localhost` em aplicativos nativos.

### Proteção contra CSRF e Injeção de Código
*   O parâmetro `state` deve conter um token CSRF de uso único, criptograficamente seguro e vinculado à sessão do *User Agent*.
*   O uso de PKCE por si só já fornece proteção robusta contra CSRF e injeção de código, pois o `code_verifier` está vinculado à transação iniciada pelo cliente.

### Defesa contra Mix-Up Attacks
Em cenários dinâmicos onde um Client interage com múltiplos Authorization Servers, um AS malicioso pode interceptar a requisição e confundir o Client, fazendo-o enviar o código de autorização (ou credenciais) para o AS errado.
*   **Mitigação**: O AS deve retornar o parâmetro `iss` (Issuer) na resposta de autorização (RFC 9207). O Client deve validar se o `iss` retornado corresponde ao AS com o qual ele pretendia se comunicar.
*   **Alternativa**: Usar `redirect_uri`s distintas e exclusivas para cada AS configurado no Client.

### Prevenção de Vazamento de Tokens
*   **Referer Headers**: AS e Clients devem suprimir o cabeçalho `Referer` em páginas que contêm códigos ou tokens (usando a política `Referrer-Policy: no-referrer`).
*   **Browser History**: Evitar expor tokens em URLs. É por isso que o *Implicit Grant* foi banido.

## Exemplo de Implementação (Python / FastAPI)

Abaixo, um exemplo de um Client confidencial implementando o callback do fluxo de Authorization Code com validação de `state` e troca de código por token, seguindo as diretrizes de segurança.

```python
import os
import secrets
from fastapi import FastAPI, Request, HTTPException, Depends
from fastapi.responses import RedirectResponse
import httpx

app = FastAPI()

CLIENT_ID = os.getenv("OAUTH_CLIENT_ID")
CLIENT_SECRET = os.getenv("OAUTH_CLIENT_SECRET")
TOKEN_URL = "https://discord.com/api/oauth2/token"
REDIRECT_URI = "https://api.meuapp.com/callback"

# Em produção, use Redis ou banco de dados para armazenar o state vinculado à sessão
pending_states = set()

@app.get("/login")
async def login():
    """Inicia o fluxo gerando state e PKCE (simplificado sem PKCE para brevidade, 
    mas obrigatório em produção)"""
    state = secrets.token_urlsafe(32)
    pending_states.add(state)
    
    # Nota: Em uma implementação real BCP 240, você geraria code_verifier e code_challenge aqui.
    auth_url = (
        f"https://discord.com/oauth2/authorize"
        f"?response_type=code&client_id={CLIENT_ID}"
        f"&redirect_uri={REDIRECT_URI}&state={state}&scope=identify"
    )
    return RedirectResponse(auth_url)

@app.get("/callback")
async def oauth2_callback(request: Request, code: str, state: str, iss: str = None):
    """Endpoint de redirecionamento (Redirect URI)"""
    
    # 1. Validação de CSRF (State)
    if state not in pending_states:
        raise HTTPException(status_code=400, detail="State inválido ou expirado (CSRF detectado)")
    pending_states.remove(state)

    # 2. Defesa contra Mix-Up Attack (RFC 9207)
    # Se o Client suporta múltiplos AS, deve validar o 'iss'
    expected_issuer = "https://discord.com"
    if iss and iss != expected_issuer:
        raise HTTPException(status_code=400, detail="Issuer mismatch (Mix-Up Attack detectado)")

    # 3. Troca do Authorization Code pelo Access Token
    # O Content-Type DEVE ser application/x-www-form-urlencoded
    data = {
        "grant_type": "authorization_code",
        "code": code,
        "redirect_uri": REDIRECT_URI,
        # Se estivéssemos usando PKCE, enviaríamos o 'code_verifier' aqui
    }
    
    async with httpx.AsyncClient() as client:
        response = await client.post(
            TOKEN_URL,
            data=data,
            auth=(CLIENT_ID, CLIENT_SECRET), # Basic Auth para Client Confidencial
            headers={"Content-Type": "application/x-www-form-urlencoded"}
        )
        
        if response.status_code != 200:
            raise HTTPException(status_code=400, detail="Falha na troca do token")
            
        token_data = response.json()
        
        # O token_data contém: access_token, token_type (Bearer), expires_in, refresh_token, scope
        return {"message": "Autenticação bem-sucedida", "access_token": token_data["access_token"]}
```

## Integrações Específicas: Discord OAuth2

Provedores como o Discord implementam o OAuth2 com extensões específicas para seu ecossistema:
*   **Scopes Específicos**: Além de escopos de identidade (`identify`, `email`), possuem escopos de ação (`guilds.join`, `applications.commands`).
*   **Bot Authorization**: O fluxo de autorização pode incluir a adição de um Bot a um servidor (Guild). Isso é controlado pelo escopo `bot` e pelo parâmetro `integration_type` (0 para GUILD_INSTALL, 1 para USER_INSTALL).
*   **Webhooks**: O escopo `webhook.incoming` gera um webhook atrelado ao canal selecionado pelo usuário durante o fluxo de autorização, retornado junto com o Access Token.

## See Also
*   [[jwt]] - JSON Web Tokens, formato frequentemente usado para Access Tokens.
*   [[openid-connect]] - Camada de identidade construída sobre o OAuth 2.0.
*   [[csrf]] - Cross-Site Request Forgery, mitigado pelo parâmetro `state` e PKCE.
*   [[mtls]] - Mutual TLS, usado para Sender-Constrained Tokens.

## Sources
*   raw/articles/rfc9700-oauth2-security.md
*   raw/articles/oauth2-best-practice.md
*   raw/articles/discord-oauth2-docs.md