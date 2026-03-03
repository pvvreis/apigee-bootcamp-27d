# API Standards (v1)

Este documento define padrões mínimos para garantir consistência, segurança e facilidade de operação.

## 1) Versionamento e naming
- Todas as rotas expostas devem iniciar com `/v1`.
- Paths devem usar `kebab-case` quando houver palavras compostas.
- Identificadores devem estar no path: `/{id}`.
- Mudanças “breaking” criam uma nova versão (ex.: v2).

## 2) Headers obrigatórios / recomendados
### 2.1 X-Correlation-Id
- O cliente pode enviar `X-Correlation-Id`.
- Se não vier, o gateway (Apigee) deve gerar.
- A API sempre deve devolver `X-Correlation-Id` na resposta.

### 2.2 Content-Type
- Para requests com body: `Content-Type: application/json`
- Para responses com body: `Content-Type: application/json`

### 2.3 Idempotency-Key (recomendado)
- Para `POST /v1/orders`: aceitar header `Idempotency-Key` (string).
- Objetivo: evitar criação duplicada em retries do cliente.

## 3) Status codes (padrão)
- 200 OK: sucesso (GET, PATCH).
- 201 Created: sucesso em criação (POST) + header `Location`.
- 204 No Content: ação concluída sem body (ex.: cancelamento).
- 400 Bad Request: erro de validação ou parâmetros inválidos.
- 401 Unauthorized: sem credencial/token válido.
- 403 Forbidden: autenticado, mas sem permissão.
- 404 Not Found: recurso não encontrado.
- 409 Conflict: conflito de estado ou idempotência.
- 429 Too Many Requests: throttling/limites (quota/spike arrest).
- 5xx: erro interno/integração.

## 4) Paginação (listas)
Para endpoints de listagem:
- Query params: `page` (>= 1) e `pageSize` (1..100).
- Response:
```json
{
  "items": [],
  "pageInfo": { "page": 1, "pageSize": 20, "total": 0 }
}
```

## 5) Padrão único de erro
Erros devem seguir sempre este formato:
```json
{
  "code": "INVALID_ARGUMENT",
  "message": "Campo 'email' inválido",
  "details": [
    { "field": "email", "issue": "must be a valid email" }
  ],
  "correlationId": "abc-123"
}
```

Regras:
- `code`: INVALID_ARGUMENT | NOT_FOUND | UNAUTHORIZED | FORBIDDEN | CONFLICT | RATE_LIMITED | INTERNAL
- `message`: mensagem curta e humana
- `details`: opcional (lista)
- `correlationId`: mesmo valor do `X-Correlation-Id`

## 6) Validações mínimas
- JSON inválido -> `400` com erro padrão.
- Campos obrigatórios ausentes -> `400`.
- Tipos inválidos -> `400`.
- `page` < 1 ou `pageSize` fora de 1..100 -> `400`.

## 7) Regras de resposta
- Não retornar stacktrace/detalhes internos.
- Sucesso com body: retornar JSON.
- Erros: sempre no formato padrão.
- Sempre devolver `X-Correlation-Id` na resposta.

## 8) Observabilidade
- Log mínimo: method, path, status, latency, correlationId.
- Para 4xx/5xx, logar também `code` do erro.
