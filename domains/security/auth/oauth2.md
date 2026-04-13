domain: domains/security/auth/oauth2.md
confidence: low
sources: 3
last_updated: 2026-04-13

# OAuth 2.0

OAuth 2.0 (Open Authorization) é um framework de delegação de autorização. Ele permite que uma aplicação (Client) obtenha acesso limitado a um recurso protegido (Resource Server) em nome do dono desse recurso (Resource Owner), sem que as credenciais (senha) do dono sejam expostas à aplicação.

**Aviso de Arquitetura:** OAuth 2.0 **não** é um protocolo de autenticação. Ele não foi desenhado para provar *quem* o usuário é, mas sim *o que* a aplicação tem permissão para fazer. Para autenticação, utiliza-se o **OpenID Connect (OIDC)**, que é uma camada de identidade construída sobre o OAuth 2.0.

---

## 1. Atores e Componentes (Roles)

O ecossistema OAuth 2.0 é composto por quatro atores fundamentais:

1.  **Resource Owner (RO):** A entidade capaz de conceder acesso a um recurso protegido. Geralmente, é o usuário final.
2.  **Client:** A aplicação que requer acesso ao recurso protegido em nome do Resource Owner. Pode ser um SPA (Single Page Application), um app mobile, ou um serviço backend.
3.  **Authorization Server (AS):** O servidor que autentica o Resource Owner, obtém seu consentimento e emite os *Access Tokens* para o Client. (ex: Auth0, Keycloak, Google OAuth).
4.  **Resource Server (RS):** O servidor que hospeda os recursos protegidos e aceita requisições que contenham um *Access Token* válido.

### Diagrama de Interação Macro

```text
+--------+                               +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------|
|        |
|        |                               +---------------|
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------|
|        |
|        |                               +---------------|
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------|
```

---

## 2. Tokens

O OAuth 2.0 opera primariamente com dois tipos de tokens:

### Access Token
Credencial de vida curta (short-lived) usada para acessar o Resource Server.
*   **Opaque Tokens:** Strings aleatórias que não contêm informação legível. O Resource Server precisa chamar um endpoint do Authorization Server (ex: `/introspect`) para validar e obter os dados do token.
*   **JWT (JSON Web Tokens):** Formato auto-contido. O Resource Server pode validar a assinatura criptográfica (usando a chave pública do AS) e ler as permissões (`scopes`) sem fazer requisições de rede adicionais.

### Refresh Token
Credencial de vida longa (long-lived) usada exclusivamente pelo Client para obter novos Access Tokens junto ao Authorization Server quando o Access Token atual expira. Nunca deve ser enviado ao Resource Server.

---

## 3. Fluxos de Concessão (Grant Types)

A forma como um Client obtém um Access Token é chamada de *Grant Type*. Com as atualizações de segurança da **RFC 9700 (OAuth 2.0 Security Best Current Practice - Jan 2025)**, o cenário de fluxos recomendados foi drasticamente simplificado.

### 3.1. Authorization Code Flow com PKCE (O Padrão Ouro)
Este é o fluxo mandatório para aplicações web, mobile e SPAs. A RFC 9700 **recomenda fortemente** o uso de PKCE (Proof Key for Code Exchange) até mesmo para *Confidential Clients* (backends que conseguem guardar segredos).

**Como o PKCE funciona:**
O PKCE mitiga ataques de interceptação de código de autorização (Authorization Code Injection).
1. O Client gera um `code_verifier` (string aleatória de alta entropia).
2. O Client calcula o `code_challenge` = `BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))`.
3. O Client envia o `code_challenge` e o método (`S256`) na requisição inicial de autorização.
4. O AS guarda o challenge. Após o login do usuário, o AS retorna um `authorization_code`.
5. O Client faz a troca do código pelo token enviando o `code_verifier` original em texto plano.
6. O AS refaz o hash do `code_verifier` e compara com o `code_challenge` guardado. Se bater, emite o token.

#### Diagrama de Sequência: Auth Code + PKCE

```text
  Browser/App (Client)                                Authorization Server
          |                                                   |
          | 1. Gera code_verifier e code_challenge            |
          | 2. GET /authorize?response_type=code              |
          |    &client_id=...&code_challenge=...              |
          |    &code_challenge_method=S256                    |
          |-------------------------------------------------->|
          |                                                   |
          | 3. Autentica o Usuário e pede Consentimento       |
          |<--------------------------------------------------|
          |                                                   |
          | 4. Redireciona de volta com `code`                |
          |<--------------------------------------------------|
          |                                                   |
          | 5. POST /token                                    |
          |    grant_type=authorization_code                  |
          |    &code=...&code_verifier=...                    |
          |-------------------------------------------------->|
          |                                                   |
          | 6. AS valida o code_verifier contra o challenge   |
          | 7. Retorna Access Token (+ Refresh Token)         |
          |<--------------------------------------------------|
```

### 3.2. Client Credentials Grant
Usado para comunicação Machine-to-Machine (M2M), onde não há um usuário final envolvido (ex: um microserviço acessando outro, ou um cronjob). O Client autentica-se diretamente no AS usando seu `client_id` e `client_secret` (ou mTLS/Private Key JWT) e recebe um Access Token.

### 3.3. Fluxos Depreciados (NÃO USE)
A RFC 9700 proíbe explicitamente o uso dos seguintes fluxos em novas implementações:
*   **Implicit Grant (`response_type=token`):** Retornava o token diretamente no fragmento da URL (`#access_token=...`). Altamente vulnerável a vazamentos no histórico do browser, open redirectors e injeção de tokens. Substituído pelo Auth Code + PKCE.
*   **Resource Owner Password Credentials (ROPC):** O Client coletava o usuário e senha e os enviava ao AS. Quebra o paradigma de delegação, treina usuários a sofrerem phishing e impossibilita o uso de MFA (Multi-Factor Authentication) ou WebAuthn.

---

## 4. Implementação Prática

### 4.1. Client: Trocando o Código pelo Token (Go)

Exemplo de um backend em Go recebendo o callback do Authorization Server e trocando o `code` pelo `access_token`.

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"net/url"
	"os"
	"strings"
)

type TokenResponse struct {
	AccessToken  string `json:"access_token"`
	RefreshToken string `json:"refresh_token"`
	ExpiresIn    int    `json:"expires_in"`
	TokenType    string `json:"token_type"`
}

// Handler para o redirect_uri (ex: /callback)
func oauthCallbackHandler(w http.ResponseWriter, r *http.Request) {
	code := r.URL.Query().Get("code")
	state := r.URL.Query().Get("state")
	
	// 1. Validar o state (CSRF protection)
	if !validateState(state) {
		http.Error(w, "Invalid state parameter", http.StatusBadRequest)
		return
	}

	// 2. Preparar a requisição de troca de token
	tokenURL := "https://oauth2.googleapis.com/token"
	data := url.Values{}
	data.Set("grant_type", "authorization_code")
	data.Set("code", code)
	data.Set("redirect_uri", "https://meu-app.com/callback")
	data.Set("client_id", os.Getenv("OAUTH_CLIENT_ID"))
	data.Set("client_secret", os.Getenv("OAUTH_CLIENT_SECRET"))
	
	// Se estiver usando PKCE (Recomendado), adicione:
	// data.Set("code_verifier", getStoredCodeVerifier(state))

	req, err := http.NewRequest("POST", tokenURL, strings.NewReader(data.Encode()))
	if err != nil {
		http.Error(w, "Failed to create request", http.StatusInternalServerError)
		return
	}
	req.Header.Add("Content-Type", "application/x-www-form-urlencoded")

	// 3. Executar a requisição
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil || resp.StatusCode != 200 {
		http.Error(w, "Failed to exchange token", http.StatusInternalServerError)
		return
	}
	defer resp.Body.Close()

	// 4. Parsear a resposta
	var tokenRes TokenResponse
	if err := json.NewDecoder(resp.Body).Decode(&tokenRes); err != nil {
		http.Error(w, "Failed to parse token", http.StatusInternalServerError)
		return
	}

	// Sucesso: Armazenar token de forma segura e criar sessão do usuário
	fmt.Fprintf(w, "Token obtido com sucesso!")
}

func validateState(state string) bool {
	// Implementar verificação contra o state salvo no cookie/sessão antes do redirect
	return true
}
```

### 4.2. Resource Server: Validando o Access Token (Python / FastAPI)

O Resource Server precisa interceptar a requisição, extrair o Bearer token e validá-lo. Se for um JWT, a validação é feita localmente verificando a assinatura (via JWKS), a expiração (`exp`) e a audiência (`aud`).

```python
from fastapi import FastAPI, Depends, HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt
from jwt import PyJWKClient

app = FastAPI()
security = HTTPBearer()

# URL onde o Authorization Server expõe suas chaves públicas
JWKS_URL = "https://meu-auth-server.com/.well-known/jwks.json"
jwks_client = PyJWKClient(JWKS_URL)

def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    token = credentials.credentials
    try:
        # Obtém a chave pública correta baseada no 'kid' do header do JWT
        signing_key = jwks_client.get_signing_key_from_jwt(token)
        
        # Valida assinatura, expiração e audiência
        payload = jwt.decode(
            token,
            signing_key.key,
            algorithms=["RS256"],
            audience="https://api.meu-app.com", # Prevenção de Privilege Escalation
            options={"require": ["exp", "iss", "aud"]}
        )
        return payload
        
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expirado")
    except jwt.InvalidAudienceError:
        raise HTTPException(status_code=403, detail="Audiência inválida")
    except jwt.InvalidTokenError as e:
        raise HTTPException(status_code=401, detail=f"Token inválido: {str(e)}")

@app.get("/api/recurso-protegido")
def get_protected_data(token_payload: dict = Depends(verify_token)):
    user_id = token_payload.get("sub")
    return {"data": "Informação sensível", "user": user_id}
```

---

## 5. Security Best Current Practices (RFC 9700)

A implementação de OAuth 2.0 exige rigor. As seguintes práticas são mandatórias para sistemas modernos:

### 5.1. Exact Redirect URI Matching
O Authorization Server **DEVE** comparar a `redirect_uri` fornecida na requisição com a URI registrada usando correspondência exata de strings (exact string matching). Expressões regulares ou wildcards abrem brechas para ataques de Open Redirect, onde um atacante rouba o `code` redirecionando o fluxo para um subdomínio controlado por ele.

### 5.2. Prevenção de CSRF e Mix-Up Attacks
*   **CSRF:** O Client deve enviar um parâmetro `state` (um token criptograficamente seguro) na requisição de autorização e validá-lo no retorno. Alternativamente, o uso de PKCE ou do parâmetro `nonce` (no OIDC) também mitiga CSRF.
*   **Mix-Up Attacks:** Ocorre quando um Client suporta múltiplos Authorization Servers e um atacante o engana para enviar o `code` de um AS legítimo para um AS malicioso. **Mitigação (RFC 9207):** O Authorization Server deve incluir o parâmetro `iss` (Issuer) na resposta de autorização, e o Client deve validar se o `iss` retornado corresponde ao AS que ele pretendia contatar.

### 5.3. Sender-Constrained Tokens (DPoP e mTLS)
Tokens Bearer tradicionais sofrem do problema de "quem porta o token, tem o poder". Se um token vaza (ex: via logs, XSS), o atacante pode usá-lo livremente.
Para sistemas de alta segurança (Open Banking, eHealth), a RFC 9700 recomenda *Sender-Constrained Tokens*:
*   **mTLS (Mutual TLS - RFC 8705):** O token é vinculado ao certificado TLS do Client. O Resource Server rejeita o token se a conexão TLS não for estabelecida com o mesmo certificado.
*   **DPoP (Demonstrating Proof-of-Possession - RFC 9449):** O Client gera um par de chaves no nível da aplicação. Ao pedir o token, ele prova a posse da chave privada. O AS embute o hash da chave pública no Access Token. Ao chamar o RS, o Client assina a requisição HTTP (DPoP Header). O RS valida se a assinatura bate com a chave pública atrelada ao token.

### 5.4. Access Token Privilege Restriction
Siga o princípio do menor privilégio:
1.  **Audience Restriction (`aud`):** O token deve ser emitido para um Resource Server específico. Se o Client pedir um token para a API de Pagamentos, esse token não deve ser aceito pela API de Usuários.
2.  **Scopes:** Limite as ações (ex: `read:invoices`, `write:invoices`).

### 5.5. Autenticação Forte de Client
Para *Confidential Clients*, evite usar `client_secret` (simétrico) se possível. Prefira métodos baseados em criptografia assimétrica:
*   `private_key_jwt` (RFC 7523): O Client assina um JWT com sua chave privada e o envia ao AS para se autenticar no endpoint `/token`.

---

## 6. Padrões de Arquitetura e Armazenamento

### Backend For Frontend (BFF)
Em SPAs (React, Vue, Angular), armazenar tokens no `localStorage` expõe a aplicação a ataques XSS (Cross-Site Scripting).
A arquitetura recomendada é o **BFF**:
1. O SPA não lida com tokens OAuth.
2. O SPA se comunica com um backend dedicado (o BFF) usando cookies de sessão tradicionais (`HttpOnly`, `Secure`, `SameSite=Strict`).
3. O BFF atua como o *Confidential Client* OAuth 2.0. Ele executa o fluxo Auth Code, guarda o Access/Refresh Token em memória ou banco de dados (criptografado), e anexa o token nas requisições proxyadas para os Resource Servers reais.

---

## See Also
*   [[openid-connect]] - Camada de identidade construída sobre OAuth 2.0.
*   [[jwt]] - JSON Web Tokens, formato comum para Access Tokens.
*   [[pkce]] - Proof Key for Code Exchange (RFC 7636).
*   [[dpop]] - Demonstrating Proof-of-Possession at the Application Layer (RFC 9449).
*   [[session-management]] - Como gerenciar sessões de usuário de forma segura.

## Sources
*   `raw/articles/rfc9700-oauth2-security.md` - RFC 9700: Best Current Practice for OAuth 2.0 Security (Jan 2025).
*   `raw/articles/oauth2-best-practice.md` - Resumo das melhores práticas de segurança.
*   `raw/articles/infisical-oauth2-implementation.md` - Guia prático de implementação e exemplos de integração.