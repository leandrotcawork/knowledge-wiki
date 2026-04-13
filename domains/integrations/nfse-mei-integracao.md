```yaml
domain: domains/integrations/nfse-mei-integracao.md
confidence: low
sources: 3
last_updated: 2026-04-13
```

# Integração de NFS-e para MEI (Padrão Nacional)

A emissão de Nota Fiscal de Serviços Eletrônica (NFS-e) para Microempreendedores Individuais (MEI) passou por uma unificação arquitetural drástica. Desde 1º de setembro de 2023 (Resolução CGSN nº 169/2022), a emissão de NFS-e por MEIs deixou de ser feita nos mais de 5.000 webservices municipais (frequentemente fragmentados em padrões como ABRASF, DSF, Ginfes) e passou a ser centralizada no **Ambiente de Dados Nacional (ADN)** da Receita Federal do Brasil (RFB).

Este documento detalha a arquitetura, os fluxos de integração via API, os mecanismos de segurança (mTLS e XMLDSig) e as armadilhas comuns ao integrar sistemas (ERPs, PDVs, plataformas de faturamento) com o Padrão Nacional de NFS-e para MEIs.

---

## 1. Arquitetura do Ambiente de Dados Nacional (ADN)

O ADN atua como um *hub* centralizador. Ele recebe as declarações de serviço, autoriza a geração da NFS-e, armazena os documentos (DF-e) e distribui os eventos para os municípios (SEFINs) envolvidos na operação (município do prestador e do tomador).

### Topologia de Integração

```text
+-----------------+        mTLS + XMLDSig         +-----------------------+
|                 | ----------------------------> |                       |
|  ERP / Sistema  |        (API REST/JSON)        |  API Nacional (ADN)   |
|  do Contribuinte| <---------------------------- |  (adn.nfse.gov.br)    |
|                 |        (Status / DF-e)        |                       |
+-----------------+                               +-----------+-----------+
        |                                                     |
        | (Gerenciamento de Certificados A1)                  | (Sincronização B2B)
        v                                                     v
+-----------------+                               +-----------+-----------+
|  HSM / Vault    |                               |  SEFIN (Municípios)   |
| (Chaves Privadas|                               |  (Prefeituras)        |
+-----------------+                               +-----------------------+
```

### DPS vs. NFS-e

No padrão nacional, o conceito de RPS (Recibo Provisório de Serviço) foi substituído pela **DPS (Declaração de Prestação de Serviço)**.
1. O contribuinte (MEI) assina e transmite uma **DPS** (formato XML).
2. O ADN valida a assinatura, o schema e as regras de negócio.
3. Se válido, o ADN gera a **NFS-e** correspondente, que é o documento fiscal com validade jurídica.

---

## 2. Autenticação e Segurança

A API Nacional não utiliza tokens JWT ou OAuth2 para integrações B2B (machine-to-machine). A segurança é baseada em dois pilares criptográficos: **mTLS** (Mutual TLS) na camada de transporte e **XMLDSig** (XML Digital Signature) na camada de aplicação.

### 2.1. Mutual TLS (mTLS)

Para fechar o handshake TLS com os servidores do Serpro/RFB, o cliente deve apresentar um certificado digital e-CNPJ (ou e-CPF) válido na cadeia ICP-Brasil.

**Atenção Operacional:** Certificados A1 brasileiros são distribuídos no formato `.pfx` ou `.p12` (PKCS#12). A maioria das bibliotecas HTTP em Python e Go exige os formatos PEM separados (certificado e chave privada).

**Comando para extração (OpenSSL):**
```bash
# Extrair a chave privada (sem senha para automação)
openssl pkcs12 -in certificado.pfx -nocerts -out key.pem -nodes

# Extrair o certificado público
openssl pkcs12 -in certificado.pfx -clcerts -nokeys -out cert.pem
```

### 2.2. Assinatura XML (XMLDSig)

O payload da DPS deve ser assinado digitalmente usando o padrão XMLDSig. A assinatura deve ser do tipo *Enveloped* (a assinatura fica dentro do próprio XML assinado), utilizando o algoritmo `RSA-SHA256`. A tag a ser assinada é a `<infDPS>`, e a URI da referência deve apontar para o atributo `Id` dessa tag.

---

## 3. Endpoints e Fluxos da API

A documentação oficial (Swagger) frequentemente apresenta inconsistências. A API é dividida em namespaces para Municípios (`/sefin/`) e Contribuintes (`/contribuintes/`). Para sistemas de emissão MEI, utiliza-se o namespace de contribuintes.

### 3.1. A Armadilha do Endpoint de Consulta (Gotcha)

Conforme relatado em fóruns de integração (Projeto ACBr), o Swagger oficial da API Nacional documenta o endpoint de download de DF-e (Documento Fiscal Eletrônico) de forma incorreta:

*   ❌ **Documentado (Swagger):** `https://adn.nfse.gov.br/contribuinte/DFe/{NSU}`
*   ✅ **Correto (Produção):** `https://adn.nfse.gov.br/contribuintes/DFe/{NSU}` *(Note o "s" em contribuintes)*

O uso da URL documentada resultará em erro `404 Not Found`, mesmo com o mTLS configurado corretamente.

### 3.2. Fluxo de Emissão (Síncrono)

Para MEIs, a emissão geralmente segue o fluxo síncrono, dado o baixo volume de notas por segundo por CNPJ.

1. **POST** `https://adn.nfse.gov.br/contribuintes/DPS`
   * **Payload:** XML da DPS assinado.
   * **Response 201:** Retorna a NFS-e gerada imediatamente no corpo da resposta.
   * **Response 400/422:** Retorna a lista de erros de validação (ex: CFOP inválido, erro de schema).

---

## 4. Exemplos de Implementação

### Padrão em Go: Cliente HTTP com mTLS

Em Go, a configuração do mTLS é feita customizando o `http.Transport` com um `tls.Config`.

```go
package nfse

import (
	"crypto/tls"
	"fmt"
	"io"
	"net/http"
	"time"
)

// NewADNClient cria um HTTP client configurado para mTLS com a API Nacional
func NewADNClient(certPEMBlock, keyPEMBlock []byte) (*http.Client, error) {
	// Carrega o par de chaves extraído do A1
	cert, err := tls.X509KeyPair(certPEMBlock, keyPEMBlock)
	if err != nil {
		return nil, fmt.Errorf("falha ao carregar key pair: %w", err)
	}

	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		// Em produção, NUNCA use InsecureSkipVerify: true
		MinVersion: tls.VersionTLS12,
	}

	transport := &http.Transport{
		TLSClientConfig: tlsConfig,
		// Otimizações para conexões keep-alive
		MaxIdleConns:          10,
		IdleConnTimeout:       60 * time.Second,
		TLSHandshakeTimeout:   10 * time.Second,
	}

	return &http.Client{
		Transport: transport,
		Timeout:   30 * time.Second,
	}, nil
}

// Exemplo de uso para buscar um DF-e
func FetchDFe(client *http.Client, nsu string) ([]byte, error) {
	// ATENÇÃO: Usando a URL corrigida (/contribuintes/ e não /contribuinte/)
	url := fmt.Sprintf("https://adn.nfse.gov.br/contribuintes/DFe/%s", nsu)
	
	req, err := http.NewRequest(http.MethodGet, url, nil)
	if err != nil {
		return nil, err
	}
	
	req.Header.Set("Accept", "application/xml")

	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("erro na API ADN: status %d", resp.StatusCode)
	}

	return io.ReadAll(resp.Body)
}
```

### Padrão em Python: Consulta com HTTPX e FastAPI

Para serviços em Python, o `httpx` suporta mTLS nativamente passando uma tupla com os caminhos dos arquivos PEM.

```python
import httpx
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

router = APIRouter()

class DFeResponse(BaseModel):
    nsu: str
    xml_content: str

# Caminhos para os certificados extraídos via OpenSSL
CERT_PATH = "/vault/secrets/cert.pem"
KEY_PATH = "/vault/secrets/key.pem"

@router.get("/api/v1/nfse/dfe/{nsu}", response_model=DFeResponse)
async def get_dfe(nsu: str):
    """
    Busca um Documento Fiscal Eletrônico (DF-e) no ADN pelo NSU.
    """
    # URL corrigida com 'contribuintes' no plural
    url = f"https://adn.nfse.gov.br/contribuintes/DFe/{nsu}"
    
    try:
        async with httpx.AsyncClient(cert=(CERT_PATH, KEY_PATH), timeout=15.0) as client:
            response = await client.get(url, headers={"Accept": "application/xml"})
            
            if response.status_code == 404:
                raise HTTPException(status_code=404, detail="DF-e não encontrado no ADN")
                
            response.raise_for_status()
            
            return DFeResponse(
                nsu=nsu,
                xml_content=response.text
            )
            
    except httpx.RequestError as exc:
        # Logar a exceção real no APM (Datadog, Sentry, etc)
        raise HTTPException(status_code=502, detail=f"Erro de comunicação com o ADN: {str(exc)}")
```

---

## 5. Regras de Negócio e Particularidades do MEI

Ao montar o XML da DPS para um MEI, existem regras de validação estritas no ADN que diferem de empresas do Lucro Presumido ou Real.

### 5.1. Tributação e ISS
O MEI recolhe impostos de forma unificada via DAS (Documento de Arrecadação do Simples Nacional). Portanto, a NFS-e **não deve** destacar valores de ISS.
*   **Regra:** O campo de "Valor Aproximado dos Tributos" (Decreto 8.264/2014) deve ser configurado como *"Não informar nenhum valor estimado para os Tributos"*.
*   **Tag XML:** O regime especial de tributação deve indicar a opção correspondente ao MEI (geralmente código `6` - Microempreendedor Individual, dependendo da tabela de domínio vigente).

### 5.2. Atualizações de 2025 (CRT e CFOP)
A partir de 2025, o ecossistema fiscal brasileiro passou a exigir maior rigor na identificação do regime tributário:
*   **CRT (Código de Regime Tributário):** O MEI deve utilizar o **CRT 4** (Simples Nacional - Microempreendedor Individual). O uso do CRT 1 (Simples Nacional genérico) para MEIs resultará em rejeição da nota.
*   **CFOP:** Embora o CFOP seja primariamente associado à NF-e (produtos), operações mistas ou regras estaduais específicas exigem a atualização das tabelas de CFOP nos sistemas emissores para refletir as novas diretrizes da Nota Técnica 2021.002 v1.12.

### 5.3. Obrigatoriedade vs. Faculdade
*   **B2B (Para Pessoa Jurídica):** Emissão **obrigatória**.
*   **B2C (Para Pessoa Física):** Emissão **facultativa**, exceto se o consumidor exigir (amparado pelo Código de Defesa do Consumidor).

### 5.4. MEI Caminhoneiro
Possui regras híbridas:
*   Transporte **Municipal**: Emite NFS-e (Serviço).
*   Transporte **Intermunicipal/Interestadual**: Emite CT-e (Conhecimento de Transporte Eletrônico) ou NF-e, não NFS-e.

---

## 6. Tratamento de Erros e Resiliência

Ao integrar com o ADN, implemente os seguintes padrões de resiliência:

1.  **Idempotência na Emissão:** Se a requisição de POST da DPS falhar por *timeout* (o ADN processou, mas a resposta não chegou ao seu ERP), não reenvie a mesma DPS com um novo ID. Consulte primeiro se o RPS/DPS já foi processado para evitar duplicidade de faturamento.
2.  **Circuit Breaker:** O ADN pode sofrer instabilidades. Implemente um *Circuit Breaker* para evitar sobrecarregar suas *threads* aguardando *timeouts* da Receita Federal.
3.  **Fila de Retentativas (Dead Letter Queue):** Para notas que falham por indisponibilidade (HTTP 503/504), coloque-as em uma fila de retentativa com *Exponential Backoff*. Notas rejeitadas por erro de negócio (HTTP 400/422) não devem ser retentadas automaticamente sem intervenção do usuário.

---

## See Also
*   [[xml-digital-signature]] - Detalhes sobre a implementação de XMLDSig (Enveloped).
*   [[mtls-architecture]] - Padrões de arquitetura para Mutual TLS em microsserviços.
*   [[nfe-integracao]] - Integração de Nota Fiscal Eletrônica (Produtos).
*   [[simples-nacional-regras]] - Regras tributárias e códigos CRT.

## Sources
*   `raw/articles/agencia_gov_nfse_mei_centralizacao.md`
*   `raw/articles/esimplesauditoria_nfse_mei_guia_completo.md`
*   `raw/articles/projetoacbr_api_nfse_nacional_adn.md`
```