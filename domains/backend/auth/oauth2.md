```yaml
---
domain: domains/backend/auth/oauth2.md
confidence: low
sources: 3
last_updated: 2026-04-13
---
```

# OAuth2 e Delegação de Autorização

Esqueça a ideia de pedir usuário e senha diretamente na sua aplicação para acessar dados de terceiros. OAuth2 é o framework padrão da indústria para **delegação de autorização**. Ele permite que um *Client* (sua aplicação) acesse recursos em um *Resource Server* (uma API) em nome de um *Resource Owner* (o usuário), sem nunca tocar nas credenciais reais.

**Regra de ouro:** OAuth2 é sobre *Autorização* (o que a aplicação pode fazer). Se você precisa de *Autenticação* (saber quem é o usuário logado), você deve usar **OpenID Connect (OIDC)**, que é uma camada de identidade construída por cima do OAuth2. Na prática, implementamos os dois juntos.

## Os Atores (Quem é Quem)

Para debugar ou implementar OAuth2, você precisa saber exatamente qual papel sua aplicação está jogando:

1. **Resource Owner:** O usuário final. É quem clica em "Permitir" para dar acesso aos dados.
2. **Client:** A aplicação que precisa do acesso (seu frontend React, seu app Mobile, ou seu backend que consome uma API externa).
3. **Authorization Server (AS):** O servidor que autentica o usuário, pede consentimento e emite os tokens (ex: Keycloak, Auth0, Ory, FusionAuth).
4. **Resource Server (RS):** A API que contém os dados. Ela recebe o token, valida e retorna o recurso.

## Qual Grant Type (Fluxo) eu devo usar?

O OAuth2 define vários fluxos ("Grant Types") para obter um token. Como engenheiros, precisamos ser pragmáticos. Eis a matriz de decisão moderna (alinhada com o OAuth 2.1):

### 1. Authorization Code Flow (com PKCE)
**Quando usar:** Aplicações Web (Server-side), SPAs (React/Angular/Vue) e Apps Mobile/Nativos.
**Como funciona:** 
1. O Client redireciona o usuário para o AS.
2. O usuário loga e aprova o acesso.
3. O AS redireciona de volta para o Client com um `code` temporário.
4. O Client troca esse `code` (junto com um secret ou PKCE verifier) por um `access_token` via chamada backend-to-backend.
**Por que usar:** O token de acesso nunca passa pelo navegador do usuário (na URL), reduzindo drasticamente a superfície de ataque.

### 2. Client Credentials
**Quando usar:** Comunicação Machine-to-Machine (M2M). Microserviços chamando microserviços, cronjobs, daemons.
**Como funciona:** Não há usuário envolvido. O Client usa seu próprio `client_id` e `client_secret` para pedir um token diretamente ao AS.
**Por que usar:** É o padrão para autenticação de serviços internos.

### 🚫 Fluxos Depreciados (NÃO USE)
*   **Implicit Grant:** Retornava o token diretamente na URL. Foi morto pelo OAuth 2.1. Use Auth Code + PKCE para SPAs.
*   **Resource Owner Password Credentials (ROPC):** A aplicação pedia a senha do usuário e trocava por um token. Quebra o princípio fundamental do OAuth2 (não tocar na senha). Evite a todo custo.

---

## Padrões de Implementação

Abaixo, exemplos práticos de como configurar OAuth2 no ecossistema Java/Spring Boot, que é comum em nossos serviços.

### 1. Configurando um Resource Server (Sua API)

Sua API precisa validar o token (geralmente um JWT) enviado no header `Authorization: Bearer <token>`.

**Dependência (`pom.xml`):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

**Configuração (`application.yml`):**
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          # URL do Authorization Server para buscar as chaves públicas (JWKS) e validar a assinatura do token
          issuer-uri: https://auth.nosso-dominio.com/realms/internal
```

**Código (Spring Security 6.x):**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                // Exige o escopo 'read:reports' para esta rota
                .requestMatchers("/api/reports/**").hasAuthority("SCOPE_read:reports")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        
        return http.build();
    }
}
```

### 2. Configurando um Client (M2M com Client Credentials)

Quando seu backend precisa chamar outra API protegida por OAuth2.

**Configuração (`application.yml`):**
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          relatorios-api:
            client-id: meu-servico-backend
            client-secret: ${OAUTH2_CLIENT_SECRET}
            authorization-grant-type: client_credentials
            scope: read:reports
        provider:
          relatorios-api:
            token-uri: https://auth.nosso-dominio.com/realms/internal/protocol/openid-connect/token
```

*Dica:* Use o `WebClient` ou `RestClient` do Spring configurado com o `OAuth2AuthorizedClientManager` para que o framework gerencie automaticamente o fetch e o refresh do token antes de cada requisição. Não implemente cache de token na mão se o framework já faz isso.

---

## Considerações de Segurança e Pitfalls

1. **Nunca armazene JWTs no `localStorage` em SPAs:** 
   Isso deixa sua aplicação vulnerável a XSS. O padrão moderno para SPAs é o **BFF (Backend-for-Frontend)**. O SPA fala com seu próprio backend usando cookies de sessão (`HttpOnly`, `Secure`), e o backend guarda o OAuth2 Access Token e faz o proxy das chamadas para o Resource Server.
2. **Valide o `aud` (Audience) e `iss` (Issuer):**
   No Resource Server, não basta validar a assinatura do JWT. Verifique se o token foi emitido pelo AS correto (`iss`) e se foi destinado à sua API (`aud`). Um token válido gerado para a API "A" não deve ser aceito pela API "B".
3. **Princípio do Menor Privilégio (Scopes):**
   Não crie escopos genéricos como `admin` ou `all`. Use escopos granulares como `users:read`, `payments:write`. O Resource Server deve validar esses escopos em cada endpoint.
4. **Tokens de Vida Curta:**
   Access Tokens devem expirar rápido (ex: 5 a 15 minutos). Use Refresh Tokens (que têm vida longa e podem ser revogados pelo AS) para obter novos Access Tokens sem incomodar o usuário.

## See Also
* [[oidc-overview]] - Para entender a camada de identidade e o ID Token.
* [[jwt-validation-patterns]] - Boas práticas na validação de JSON Web Tokens.
* [[bff-pattern]] - Implementação do Backend-for-Frontend para SPAs seguros.

## Sources
* `raw/articles/oauth2-ory-concepts.md`
* `raw/articles/oauth2-igventurelli-indepth.md`
* `raw/articles/oauth2-fusionauth-simplified.md`