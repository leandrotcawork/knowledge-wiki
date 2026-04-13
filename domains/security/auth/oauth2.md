domain: security
confidence: medium
sources: 4
last_updated: 2026-04-13

# OAuth2

OAuth2 é um framework de **autorização delegada**, não de autenticação. Ele permite que uma aplicação (Client) acesse recursos protegidos em nome de um usuário (Resource Owner) ou de si mesma, sem expor credenciais. Se você precisa saber *quem* é o usuário, você precisa de [[openid-connect]] (OIDC), que roda sobre o OAuth2.

Nunca construa seu próprio Authorization Server a menos que seja o core business da sua empresa. Use soluções consolidadas (Ory, Keycloak, Auth0, AWS Cognito). Seu código deve focar em atuar como **Client** (solicitando tokens) ou **Resource Server** (validando tokens).

## Atores do Ecossistema

*   **Resource Owner:** O usuário ou sistema dono dos dados.
*   **Client:** A aplicação que precisa de acesso (seu frontend SPA, app mobile ou microsserviço).
*   **Authorization Server (AS):** O motor que autentica o usuário, gerencia o consentimento e emite os tokens.
*   **Resource Server (RS):** A API que protege os dados. Recebe o Access Token, valida sua integridade/permissões e serve a requisição.

## Qual Grant Type usar? (Decisões de Arquitetura)

A RFC 9700 (BCP 240) atualizou drasticamente as melhores práticas de segurança do OAuth2. Esqueça o que você aprendeu antes de 2020.

### 1. Authorization Code com PKCE (Padrão Ouro)
**Quando usar:** Aplicações Web (Server-side), SPAs (React/Vue) e Apps Mobile.
**Como funciona:** O Client redireciona o usuário para o AS. Após o login, o AS redireciona de volta com um `code`. O Client troca esse `code` por um Access Token em uma chamada backend. O PKCE (Proof Key for Code Exchange) garante que quem iniciou o fluxo é o mesmo que está trocando o código, prevenindo ataques de interceptação.

### 2. Client Credentials
**Quando usar:** Comunicação Machine-to-Machine (M2M), CRON jobs, daemons.
**Como funciona:** Não há usuário envolvido. O Client usa seu próprio `client_id` e `client_secret` para pedir um token diretamente ao AS.

### 🚫 Fluxos Depreciados (NÃO USE)
A RFC 9700 proíbe explicitamente o uso dos seguintes fluxos em novas implementações:
*   **Implicit Grant:** Retornava o token diretamente na URL. Vulnerável a vazamentos no histórico do browser e injeção de tokens. Substituído pelo Auth Code + PKCE.
*   **Resource Owner Password Credentials (ROPC):** O Client pedia usuário e senha e os repassava ao AS. Quebra o princípio fundamental do OAuth2 de não compartilhar credenciais com o Client.

## Implementação: Resource Server (FastAPI)

O Resource Server não se importa com *como* o token foi obtido. Ele apenas recebe o token via header `Authorization: Bearer <token>` e o valida.

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt # PyJWT
from pydantic import BaseModel

app = FastAPI()
security = HTTPBearer()

# Configurações do Authorization Server (na prática, carregue via env vars)
JWKS_URL = "https://seu-tenant.auth0.com/.well-known/jwks.json"
AUDIENCE = "https://api.suaempresa.com"
ISSUER = "https://seu-tenant.auth0.com/"

class UserContext(BaseModel):
    sub: str
    scopes: list[str]

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)) -> UserContext:
    token = credentials.credentials
    try:
        # Em produção, faça cache do JWKS (chaves públicas do AS)
        # O PyJWT possui PyJWKClient para buscar e fazer cache das chaves automaticamente
        jwks_client = jwt.PyJWKClient(JWKS_URL)
        signing_key = jwks_client.get_signing_key_from_jwt(token)
        
        payload = jwt.decode(
            token,
            signing_key.key,
            algorithms=["RS256"],
            audience=AUDIENCE,
            issuer=ISSUER
        )
        
        # Extrai os scopes (geralmente vêm como uma string separada por espaços)
        scopes = payload.get("scope", "").split()
        return UserContext(sub=payload.get("sub"), scopes=scopes)
        
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Token expirado")
    except jwt.InvalidTokenError as e:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail=f"Token inválido: {str(e)}")

@app.get("/api/protected")
def protected_route(user: UserContext = Depends(verify_token)):
    if "read:data" not in user.scopes:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Scope insuficiente")
    
    return {"message": "Acesso concedido", "user_id": user.sub}
```

## Implementação: Client Credentials (M2M)

Quando seu microsserviço precisa chamar outra API, ele atua como Client. Use o fluxo `client_credentials`.

```python
import httpx
from pydantic_settings import BaseSettings
from fastapi import BackgroundTasks

class OAuth2Settings(BaseSettings):
    client_id: str
    client_secret: str
    token_url: str

async def get_m2m_token(settings: OAuth2Settings) -> str:
    """
    Busca um token M2M. Em produção, você DEVE fazer cache deste token
    até que ele expire (verifique a claim 'expires_in' da resposta)
    para não sobrecarregar o Authorization Server.
    """
    async with httpx.AsyncClient() as client:
        response = await client.post(
            settings.token_url,
            data={
                "grant_type": "client_credentials",
                "client_id": settings.client_id,
                "client_secret": settings.client_secret,
                # Opcional: solicitar scopes específicos
                # "scope": "write:reports"
            },
            headers={"Content-Type": "application/x-www-form-urlencoded"}
        )
        response.raise_for_status()
        return response.json()["access_token"]
```

## Considerações de Segurança (RFC 9700)

1.  **Validação Estrita de Redirect URIs:** O Authorization Server deve exigir correspondência exata (exact match) das URIs de redirecionamento. Nada de wildcards (`*`) no path. Isso previne ataques de Open Redirect que roubam o `code`.
2.  **Token Replay Prevention:** Access tokens devem ter vida curta (ex: 5 a 15 minutos). Para manter a sessão, use Refresh Tokens com **Rotation** (cada vez que um refresh token é usado, ele é invalidado e um novo é emitido).
3.  **Access Token Privilege Restriction:** Aplique o princípio do menor privilégio. Limite os `scopes` e sempre valide a claim `aud` (Audience) para garantir que o token foi emitido especificamente para a API que o está recebendo.
4.  **Sender-Constrained Tokens:** Para sistemas de altíssima segurança, considere usar mTLS (Mutual TLS) ou DPoP (Demonstrating Proof-of-Possession) para atrelar o token ao cliente que o solicitou, tornando-o inútil se for roubado.

## See Also
*   [[openid-connect]]
*   [[jwt-validation]]
*   [[m2m-authentication]]
*   [[api-security-best-practices]]

## Sources
*   `raw/articles/oauth2-best-practice.md`
*   `raw/articles/rfc9700-oauth2-security.md`
*   `raw/articles/ory-oauth2-concepts.md`
*   `raw/articles/devto-oauth2-overview.md`