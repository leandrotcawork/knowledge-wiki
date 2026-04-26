```yaml
domain: domains/integrations/nfse-mei-integracao.md
confidence: high
sources: 5
last_updated: 2024-05-20
```

# Integração de NFS-e para MEI (Padrão Nacional)

A emissão de Nota Fiscal de Serviços Eletrônica (NFS-e) para Microempreendedores Individuais (MEI) no Brasil passou por uma unificação arquitetural drástica. Desde 1º de setembro de 2023, por força da Resolução CGSN nº 169/2022, a emissão de NFS-e por MEIs deixou de ser realizada nos mais de 5.000 webservices municipais (frequentemente fragmentados em padrões legados baseados em SOAP, como ABRASF, DSF e Ginfes) e passou a ser obrigatoriamente centralizada no **Ambiente de Dados Nacional (ADN)** da Receita Federal do Brasil (RFB) em parceria com o Serpro.

Este documento detalha a arquitetura do Padrão Nacional, os fluxos de integração via API REST, os mecanismos de segurança criptográfica (mTLS e XMLDSig/JWS) e as diretrizes de engenharia para integração de sistemas (ERPs, PDVs, plataformas de faturamento) com o ecossistema governamental.

---

## 1. Arquitetura do Ambiente de Dados Nacional (ADN)

O ADN atua como um *hub* centralizador de mensageria e armazenamento. Ele recebe as Declarações de Prestação de Serviço (DPS), autoriza a geração da NFS-e, armazena os Documentos Fiscais Eletrônicos (DF-e) e distribui os eventos de forma assíncrona para as Secretarias de Finanças (SEFINs) dos municípios envolvidos na operação (município do prestador e do tomador).

### Topologia de Integração

```text
+-----------------+        mTLS + XMLDSig/JWS     +-----------------------+
|                 | ----------------------------> |                       |
|  ERP / Sistema  |        (API REST/JSON)        |  API Nacional (ADN)   |
|  do Contribuinte| <---------------------------- |  (adn.nfse.gov.br)    |
|                 |        (Status / DF-e)        |                       |
+-----------------+                               +-----------+-----------+
        |                                                     |
        | (Gerenciamento de Certificados A1/A3)               | (Sincronização B2B)
        v                                                     v
+-----------------+                               +-----------+-----------+
|  HSM / Vault /  |                               |  SEFINs Municipais    |
|  KMS Provider   |                               |  (Prefeituras)        |
+-----------------+                               +-----------------------+
```

A mudança arquitetural substitui o modelo *point-to-point* (onde o ERP precisava conhecer a URL e o padrão de cada município) por um modelo de *Gateway* Único.

---

## 2. Padrões de Comunicação e Protocolos

Diferente dos padrões municipais legados que utilizavam SOAP/XML, a API do Padrão Nacional foi desenhada sob princípios RESTful, suportando payloads modernos, mas mantendo o rigor das assinaturas digitais exigidas pela infraestrutura de Chaves Públicas Brasileira (ICP-Brasil).

### 2.1. Declaração de Prestação de Serviço (DPS)
A integração não envia uma "Nota Fiscal" pronta. O sistema emissor transmite uma **DPS**. O ADN valida as regras de negócio (tributação, retenções, cadastro) e, se aprovada, o próprio ADN gera a NFS-e e atribui a ela uma chave de acesso nacional de 50 posições.

A API suporta dois formatos para a DPS:
*   **JSON:** Assinado digitalmente utilizando o padrão JWS (JSON Web Signature).
*   **XML:** Assinado digitalmente utilizando o padrão XMLDSig (XML Digital Signature).

### 2.2. Segurança de Transporte (mTLS)
A comunicação com os endpoints de produção do ADN exige **Mutual TLS (mTLS)**. Isso significa que não basta o servidor (ADN) apresentar um certificado SSL válido; o cliente (ERP) deve apresentar um certificado digital de cliente (e-CNPJ ou e-CPF) válido na cadeia ICP-Brasil durante o *handshake* TLS.

### 2.3. Assinatura de Mensagem (Non-repudiation)
Além do mTLS para o canal, o *payload* (DPS) deve ser assinado digitalmente. A assinatura garante a integridade e o não-repúdio do documento. O certificado utilizado para assinar o payload deve pertencer ao CNPJ/CPF do emissor ou de um procurador legalmente cadastrado na RFB.

---

## 3. Fluxo de Emissão e Ciclo de Vida

O ciclo de vida de uma NFS-e no Padrão Nacional segue um fluxo de estados estrito, gerenciado por eventos.

1.  **Geração e Assinatura:** O ERP compila os dados do serviço prestado em um objeto DPS (JSON ou XML), calcula os *hashes* e aplica a assinatura digital (JWS/XMLDSig).
2.  **Transmissão (POST):** O ERP envia a DPS para o endpoint `/v1/dps` do ADN via conexão mTLS.
3.  **Processamento Síncrono/Assíncrono:** 
    *   *Síncrono:* Para lotes unitários, a API geralmente retorna a NFS-e autorizada (HTTP 201 Created) ou a lista de erros de validação (HTTP 400 Bad Request) na mesma requisição.
    *   *Assíncrono:* Para processamento em lote, a API retorna um número de recibo (HTTP 202 Accepted), exigindo *polling* posterior (GET) para resgatar o resultado.
4.  **Distribuição:** Uma vez autorizada, o ADN notifica via *webhooks* ou filas as prefeituras envolvidas.
5.  **Eventos Posteriores:** Cancelamentos ou substituições são tratados como "Eventos" anexados à NFS-e original, enviados para endpoints específicos (ex: `/v1/nfse/{chave}/eventos`).

---

## 4. Desafios Técnicos e Padrões de Resiliência

A integração com sistemas governamentais exige práticas robustas de engenharia de software para lidar com instabilidades de rede e rigor criptográfico.

### 4.1. Canonicalização e Quebra de Assinatura
O erro mais comum na integração via XMLDSig é a quebra de assinatura devido a modificações no payload após a assinatura. Espaços em branco, quebras de linha (CRLF vs LF) ou reordenação de atributos JSON/XML introduzidos por *middlewares* ou *proxies* invalidam o *digest* criptográfico. É imperativo aplicar algoritmos de canonicalização (ex: `Exclusive XML Canonicalization`) antes de assinar e garantir que o payload seja transmitido como um *stream* binário inalterado.

### 4.2. Idempotência e Tratamento de Timeouts
Devido à natureza da internet e possíveis latências no ADN, requisições podem sofrer *timeout* após a DPS ter sido processada pela Receita, mas antes da resposta chegar ao ERP.
*   **Solução:** O sistema emissor deve implementar mecanismos de **Idempotência**. O ADN utiliza o ID da DPS (gerado pelo emissor) para garantir que reenvios da mesma DPS não gerem notas fiscais duplicadas. O ERP deve consultar a chave da DPS antes de tentar gerar uma nova em caso de falha de rede.

### 4.3. Gestão de Certificados em Nuvem
A exigência de mTLS apresenta desafios para arquiteturas *cloud-native* (Serverless, Containers). Serviços de API Gateway gerenciados (como AWS API Gateway ou Cloudflare) frequentemente terminam o TLS na borda.
*   **Padrão Recomendado:** Utilizar *forwarding* de certificados de cliente via cabeçalhos HTTP internos em redes VPC fechadas, ou delegar a comunicação de saída (egress) para *workers* específicos que possuam acesso seguro a um HSM (Hardware Security Module) ou serviço de cofre (Vault) onde a chave privada do e-CNPJ está armazenada, evitando que a chave privada transite pela rede.

---

## See Also

*   [Mutual TLS (mTLS) Authentication](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/) - Conceitos de segurança de camada de transporte.
*   [XML Signature Syntax and Processing (XMLDSig)](https://www.w3.org/TR/xmldsig-core/) - Especificação W3C.
*   [JSON Web Signature (JWS)](https://datatracker.ietf.org/doc/html/rfc7515) - RFC 7515.
*   [Idempotency in Distributed Systems](https://martinfowler.com/articles/microservices.html) - Padrões de resiliência.

## Sources

1.  **Portal da Nota Fiscal de Serviço Eletrônica (NFS-e)** - Receita Federal do Brasil / Serpro. Documentação Técnica e Manuais de Integração.
2.  **Resolução CGSN nº 169/2022** - Comitê Gestor do Simples Nacional (Base legal da obrigatoriedade para MEI).
3.  **RFC 8705: OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens** - IETF (Referência para implementações de mTLS).
4.  **Manual de Orientação do Contribuinte (MOC) - NFS-e Nacional** - Especificações de leiaute, regras de validação e dicionário de dados.