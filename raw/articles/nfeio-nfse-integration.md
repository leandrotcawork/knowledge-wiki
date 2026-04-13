<!-- source_url: https://nfe.io/docs/documentacao/nota-fiscal-servico-eletronica/primeiros-passos/ -->
# Primeiros Passos para integração de nota fiscal de serviço - NFE.io
Requisitos:
- Empresa com CNPJ e CNAE de serviço.
- Certificado Digital A1 (.pfx).
- Integração via API REST.
Fluxo:
1. Criar empresa (POST /v2/companies).
2. Enviar certificado (POST /v2/companies/{id}/certificate).
3. Emitir nota (POST /v2/companies/{id}/serviceinvoices).
Payload exemplo: description, cityServiceCode, issRate, borrower data.
O processamento é assíncrono.
Ambiente de testes disponível sem necessidade de certificado real inicialmente.