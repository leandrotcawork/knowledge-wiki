domain: domains/integrations/nfse-mei.md
confidence: low
sources: 3
last_updated: 2026-04-13

# Integração de NFS-e para MEI (Padrão Nacional)

A emissão de Nota Fiscal de Serviço Eletrônica (NFS-e) no Brasil historicamente foi um dos maiores pesadelos de integração para engenheiros de software. Com mais de 5.500 municípios, existiam centenas de padrões diferentes (ABRASF, Ginfes, Betha, etc.), schemas XML incompatíveis e webservices baseados em SOAP com implementações de segurança arcaicas.

O **Padrão Nacional da NFS-e**, gerido pela Receita Federal e pelo Serpro, introduziu o **ADN (Ambiente de Dados Nacional)**. Desde 1º de setembro de 2023, tornou-se **obrigatório** para todos os Microempreendedores Individuais (MEIs) emitirem suas notas de serviço (para pessoas jurídicas) através deste padrão unificado, desobrigando-os do uso dos sistemas municipais.

Este documento detalha a arquitetura, os fluxos de integração, modelagem de dados e padrões de resiliência necessários para integrar a emissão de NFS-e do MEI em sistemas de software.

---

## 1. Arquitetura e Conceitos Core

Para integrar com o Padrão Nacional, é fundamental entender a separação entre a intenção de faturamento e o documento fiscal consolidado.

### 1.1. DPS vs. NFS-e
*   **DPS (Declaração de Prestação de Serviço)**: É o documento (payload) que o seu sistema gera e envia para a API. Ele contém os dados do prestador, tomador, serviço realizado e valores.
*   **NFS-e (Nota Fiscal de Serviço Eletrônica)**: É o documento fiscal com validade jurídica, gerado pelo ADN (Serpro) *após* a validação e autorização da DPS.
*   **DANFSE (Documento Auxiliar da NFS-e)**: É a representação visual (geralmente em PDF) da NFS-e, utilizada para entrega ao cliente final.

### 1.2. Topologia de Integração

Existem duas abordagens principais para integração: **Direta (Serpro)** e via **Gateway/Broker (APIs de Terceiros)**.

```text
+-------------------+       JSON / REST        +-------------------+
|                   |  (Autenticação via API   |                   |
|  Sistema Interno  |   Key ou OAuth2)         |  Gateway Fiscal   | (NFE.io, TecnoSpeed, etc.)
|  (ERP / SaaS)     +------------------------->+  (Broker)         |
|                   |                          |                   |
+--------+----------+                          +---------+---------+
         |                                               |
         |                                               | XML Assinado (XMLDSig)
         |                                               | mTLS (Certificado A1)
         |                                               v
         |                                     +-------------------+
         |  Integração Direta (Hard Mode)      |                   |
         +------------------------------------>+   ADN / Serpro    | (Ambiente de Dados Nacional)
            XML Assinado + mTLS                |                   |
                                               +---------+---------+
                                                         |
                                                         | Sincronização Assíncrona
                                                         v
                                               +-------------------+
                                               |                   |
                                               |    Prefeituras    | (Sistemas Municipais)
                                               |                   |
                                               +-------------------+
```

#### Decisão de Engenharia: Direta vs. Gateway
*   **Integração Direta**: Exige implementação de assinatura digital XML (XMLDSig) no padrão ICP-Brasil, gestão de certificados digitais (A1/A3) em memória, e lidar diretamente com as instabilidades do Serpro. Recomendado *apenas* se você é um ERP de grande porte onde o custo por nota em um gateway inviabiliza o negócio.
*   **Integração via Gateway**: Você envia um payload JSON simples. O gateway gerencia o certificado digital do cliente (armazenado em cofre seguro), assina o XML, gerencia a fila de retentativas (circuit breakers para o Serpro) e devolve webhooks. **Fortemente recomendado** para 99% dos casos de uso.

---

## 2. Autenticação e Segurança

### 2.1. Certificados Digitais (A1)
Para emissão via API (seja direta ou via gateway), o MEI precisa de um **Certificado Digital A1** (arquivo `.pfx` ou `.p12` - PKCS#12). Este arquivo contém a chave pública e a chave privada.

Se você estiver construindo um gateway ou integração direta, precisará extrair a chave privada e o certificado para estabelecer a conexão **mTLS (Mutual TLS)** com os servidores do governo e para assinar o XML.

### 2.2. Login Gov.br (Selo Prata/Ouro)
Para emissão manual via portal ou app, o MEI utiliza o OAuth2 do Gov.br. Algumas APIs do governo permitem a emissão delegada via tokens OAuth2 do Gov.br, mas para integrações *system-to-system* (background jobs, faturamento em lote), o uso do Certificado A1 via mTLS é o padrão da indústria.

---

## 3. Modelagem de Dados (Payload da DPS)

A emissão para MEI é simplificada. O MEI recolhe impostos via DAS (Documento de Arrecadação do Simples Nacional), portanto, a NFS-e do MEI **não retém ISS** na fonte e não possui cálculos complexos de alíquotas na nota.

### 3.1. Estrutura de Dados (Python / Pydantic)

Abaixo, uma modelagem moderna utilizando `FastAPI` e `Pydantic` para receber uma requisição de faturamento no seu sistema e prepará-la para envio a um Gateway (ex: NFE.io ou TecnoSpeed).

```python
from datetime import date
from typing import Optional
from pydantic import BaseModel, Field, field_validator

class Endereco(BaseModel):
    codigo_municipio: str = Field(..., description="Código IBGE do município (7 dígitos)")
    cep: str = Field(..., pattern=r"^\d{8}$")
    logradouro: str
    numero: str
    bairro: str
    uf: str = Field(..., min_length=2, max_length=2)

class Tomador(BaseModel):
    cpf_cnpj: str = Field(..., description="CPF ou CNPJ do cliente, apenas números")
    razao_social: str
    email: Optional[str] = None
    endereco: Endereco

class Servico(BaseModel):
    codigo_tributacao_nacional: str = Field(
        ..., 
        description="Código NBS ou código de serviço nacional"
    )
    descricao: str = Field(..., max_length=2000)
    valor_servico: float = Field(..., gt=0)
    
    # Para MEI, a tributação é específica e o ISS não é retido na nota
    iss_retido: bool = False
    
class EmissaoDPSRequest(BaseModel):
    id_integracao: str = Field(..., description="ID único no seu sistema para idempotência")
    data_competencia: date
    tomador: Tomador
    servico: Servico

    @field_validator('tomador')
    def validate_cpf_cnpj(cls, v):
        if len(v.cpf_cnpj) not in (11, 14):
            raise ValueError('CPF deve ter 11 dígitos ou CNPJ 14 dígitos')
        return v

# Exemplo de uso em um endpoint FastAPI
from fastapi import FastAPI, HTTPException

app = FastAPI(title="API de Faturamento MEI")

@app.post("/v1/faturamento/emitir")
async def emitir_nota(payload: EmissaoDPSRequest):
    """
    Recebe a intenção de faturamento, salva no banco e enfileira para processamento.
    """
    # 1. Salvar no banco de dados com status 'PENDENTE'
    # 2. Enviar para fila (RabbitMQ/Kafka/SQS) para processamento assíncrono
    # 3. Retornar 202 Accepted
    
    return {
        "status": "processando",
        "id_integracao": payload.id_integracao,
        "mensagem": "DPS enfileirada para emissão no Padrão Nacional"
    }
```

---

## 4. Fluxo Assíncrono e Resiliência

A comunicação com sistemas governamentais (SEFAZ, Serpro) é inerentemente instável. Picos de acesso no início e fim do mês causam timeouts frequentes. **Nunca bloqueie a thread do usuário aguardando a autorização de uma nota.**

O fluxo deve ser estritamente assíncrono:
1. Seu sistema gera a DPS e envia para a API (Governo ou Gateway).
2. A API retorna um número de recibo ou protocolo.
3. Um *Worker* em background faz *polling* do status do protocolo, ou seu sistema recebe um *Webhook* quando a nota for autorizada/rejeitada.

### 4.1. Implementação de Worker de Polling (Go)

Go é excelente para construir workers de alta concorrência que lidam com I/O bound tasks (como fazer polling em APIs lentas).

```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"time"
)

// InvoiceTask representa uma nota pendente de processamento
type InvoiceTask struct {
	IntegrationID string
	Protocol      string
	Attempt       int
}

// GatewayResponse simula a resposta de consulta de um gateway (ex: NFE.io)
type GatewayResponse struct {
	Status string `json:"status"` // "PROCESSING", "AUTHORIZED", "ERROR"
	PDFUrl string `json:"pdf_url,omitempty"`
	Errors string `json:"errors,omitempty"`
}

func processInvoiceStatus(ctx context.Context, task InvoiceTask, results chan<- string) {
	// Backoff exponencial simples baseado na tentativa
	backoff := time.Duration(task.Attempt*2) * time.Second
	time.Sleep(backoff)

	url := fmt.Sprintf("https://api.gatewayfiscal.com.br/v1/invoices/%s/status", task.Protocol)
	req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
	req.Header.Set("Authorization", "Bearer YOUR_API_KEY")

	client := &http.Client{Timeout: 10 * time.Second}
	resp, err := client.Do(req)
	if err != nil {
		log.Printf("[Worker] Erro de rede ao consultar %s: %v", task.Protocol, err)
		// Re-enfileirar task na vida real
		return
	}
	defer resp.Body.Close()

	var result GatewayResponse
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		log.Printf("[Worker] Erro no decode: %v", err)
		return
	}

	switch result.Status {
	case "AUTHORIZED":
		log.Printf("[Worker] Sucesso! Nota %s autorizada. PDF: %s", task.IntegrationID, result.PDFUrl)
		// Atualizar banco de dados, disparar email para o cliente
		results <- task.IntegrationID
	case "ERROR":
		log.Printf("[Worker] Rejeição na Sefaz/Serpro para %s: %s", task.IntegrationID, result.Errors)
		// Atualizar banco de dados com o erro
	case "PROCESSING":
		log.Printf("[Worker] Nota %s ainda processando. Tentativa %d", task.IntegrationID, task.Attempt)
		// Na vida real: colocar de volta na fila (ex: RabbitMQ com delay)
	}
}

func main() {
	// Canal para simular uma fila de tarefas
	tasks := make(chan InvoiceTask, 100)
	results := make(chan string, 100)

	// Iniciar pool de workers
	for w := 1; w <= 5; w++ {
		go func(workerID int) {
			for task := range tasks {
				log.Printf("[Worker %d] Processando protocolo %s", workerID, task.Protocol)
				processInvoiceStatus(context.Background(), task, results)
			}
		}(w)
	}

	// Simulando a inserção de tarefas (notas que foram enviadas e aguardam processamento)
	tasks <- InvoiceTask{IntegrationID: "INV-1001", Protocol: "PROT-9991", Attempt: 1}
	tasks <- InvoiceTask{IntegrationID: "INV-1002", Protocol: "PROT-9992", Attempt: 1}

	// Aguardar resultados (simplificado para o exemplo)
	for i := 0; i < 2; i++ {
		<-results
	}
	close(tasks)
}
```

---

## 5. Tratamento de Erros e Edge Cases

Ao integrar com o Padrão Nacional, você encontrará cenários específicos que exigem tratamento robusto:

### 5.1. Idempotência
Sempre envie uma chave de idempotência (geralmente o ID da sua transação interna) ao criar a DPS. Se houver um timeout na requisição de criação, você pode tentar novamente com o mesmo ID. O gateway ou o Serpro deve reconhecer que a DPS já foi recebida e retornar o protocolo existente, evitando a emissão em duplicidade (o que geraria bitributação ou necessidade de cancelamento).

### 5.2. Cancelamento de NFS-e
O cancelamento só é permitido dentro de um prazo específico (geralmente até o fechamento da competência mensal) e, em muitos casos, exige um código de justificativa.
*   **Fluxo**: Envia-se um evento de cancelamento referenciando a chave de acesso da NFS-e.
*   **Atenção**: Se a nota já foi paga ou o imposto já foi recolhido no DAS, o cancelamento pode exigir processo administrativo.

### 5.3. Indisponibilidade do ADN (Serpro)
Quando o Ambiente de Dados Nacional cai, não há fallback automático.
*   **Estratégia**: Implemente um *Circuit Breaker*. Se a taxa de erro 5xx do gateway/Serpro ultrapassar um limite, pause o envio de novas DPS, acumule-as em uma fila (Dead Letter Queue ou fila de retry) e exiba um aviso no painel do usuário ("Sistema do Governo temporariamente indisponível. Suas notas estão na fila e serão emitidas automaticamente").

---

## 6. Diferenças Práticas: MEI vs. Lucro Presumido/Real

Se o seu sistema atende MEIs e outros regimes tributários, isole a lógica de negócio. O Padrão Nacional simplificou o MEI, mas a complexidade permanece para outros regimes:

| Característica | MEI (Padrão Nacional) | Outros Regimes (Lucro Presumido/Real) |
| :--- | :--- | :--- |
| **Retenção de ISS** | Não aplicável (pago no DAS) | Pode haver retenção pelo tomador |
| **Alíquota ISS** | Fixa/Isenta na nota | Variável (2% a 5%) dependendo do município |
| **Obrigatoriedade** | Nacional (ADN) | Depende do município (alguns já no ADN, outros em sistemas próprios) |
| **Campos Obrigatórios**| CNPJ, Valor, Código Serviço | + Inscrição Municipal, + CNAE, + Regras de Retenção (PIS/COFINS/CSLL) |

---

## See Also
*   [[oauth2]] - Para entender o fluxo de delegação do Gov.br.
*   [[mtls]] - Mutual TLS, essencial para comunicação direta com o Serpro usando certificados A1.
*   [[circuit-breaker]] - Padrão de resiliência vital para lidar com instabilidades de APIs governamentais.
*   [[message-queues]] - Arquitetura recomendada para processamento assíncrono de notas fiscais.

## Sources
*   raw/articles/gov-nfse-mei.md
*   raw/articles/nfeio-nfse-integration.md
*   raw/articles/tecnospeed-nfse-nacional.md