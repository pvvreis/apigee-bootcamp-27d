# Endpoints v1

Este arquivo define o escopo da API (rotas) e um resumo do contrato.
O contrato formal será feito em OpenAPI no Dia 2.

## Customers

### GET /v1/customers
Lista clientes (paginado).
- Query: `page`, `pageSize`
- 200: `{ items: [...], pageInfo: {...} }`

### POST /v1/customers
Cria um cliente.
- Request (exemplo):
  - `{ "name": "Maria", "email": "maria@exemplo.com", "document": "12345678900" }`
- 201: retorna Customer + header `Location`
- 400: erro padrão

### GET /v1/customers/{id}
Busca cliente por id.
- 200: retorna Customer
- 404: erro padrão

### PATCH /v1/customers/{id}
Atualiza parcialmente o cliente.
- Request (parcial):
  - `{ "name": "Novo Nome" }` ou `{ "email": "novo@exemplo.com" }`
- 200: retorna Customer
- 400/404: erro padrão

## Orders

### GET /v1/orders
Lista pedidos (paginado + filtros opcionais).
- Query: `page`, `pageSize`, `status?`, `customerId?`
- 200: `{ items: [...], pageInfo: {...} }`

### POST /v1/orders
Cria um pedido.
- Header recomendado: `Idempotency-Key`
- Request (exemplo):
  - `{ "customerId": "cst_123", "items": [{ "productId": "prd_9", "qty": 2 }], "notes": "..." }`
- 201: retorna Order + header `Location`
- 400: erro padrão
- 409: conflito (ex.: Idempotency-Key repetida com payload diferente)

### GET /v1/orders/{id}
Busca pedido por id.
- 200: retorna Order
- 404: erro padrão

### POST /v1/orders/{id}/cancel
Cancela um pedido (ação).
- 204: sucesso sem body (padrão recomendado)
- 404: erro padrão
- 409: conflito (ex.: pedido já cancelado ou estado não permite)

---

## Fluxos de exemplo (Dia 1)

### Fluxo A: Criar Customer
1) Request: `POST /v1/customers`
2) Sucesso: `201 Created`
   - Header `Location: /v1/customers/{id}`
   - Header `X-Correlation-Id: <id>`
3) Erro de validação: `400`
```json
{
  "code": "INVALID_ARGUMENT",
  "message": "Campo 'email' inválido",
  "details": [{ "field": "email", "issue": "must be a valid email" }],
  "correlationId": "abc-123"
}
```

### Fluxo B: Listar Orders paginado
1) Request: `GET /v1/orders?page=1&pageSize=20`
2) Sucesso: `200 OK`
```json
{
  "items": [],
  "pageInfo": { "page": 1, "pageSize": 20, "total": 0 }
}
```

### Fluxo C: Cancelar Order
1) Request: `POST /v1/orders/{id}/cancel`
2) Sucesso: `204 No Content`
3) Caso já cancelado: `409 Conflict`
```json
{
  "code": "CONFLICT",
  "message": "Pedido já está cancelado",
  "details": [],
  "correlationId": "abc-123"
}
```
