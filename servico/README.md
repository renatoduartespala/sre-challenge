# Serviço

Terreno da API de pagamentos. O código está em [`src/`](../src). Não há outro alvo. Na instrumentação, o que conta é se o mapa destas rotas gerou o sinal. No raciocínio, se cada fluxo virou uma pergunta. No relatório, se outra pessoa sobe o processo e reconhece a rota.

## O que já existe

- [`src/server.ts`](../src/server.ts) sobe o Express e define as rotas. O comentário no final do arquivo marca o ponto de instrumentação.
- [`src/settings.ts`](../src/settings.ts) lê `PORT` e `ARTIFICIAL_LATENCY_MS`.
- [`src/utils.ts`](../src/utils.ts) devolve pedidos e pagamentos fixos e aplica a latência artificial. Essa chamada é o trecho interno que o trace precisa atravessar.
- O log usa pino. O `POST /api/v1/payments` já emite `payment processed` quando aceita o pagamento.

## Rotas

| Método | Rota | O que observar |
| --- | --- | --- |
| `GET` | `/healthz` | processo no ar |
| `GET` | `/api/v1/orders` | lista, com a latência artificial |
| `GET` | `/api/v1/payments/:id` | encontrado e `404` |
| `POST` | `/api/v1/payments` | `201`, `400` sem `orderId` ou `amount`, `409` quando o pedido já tem pagamento |

Pedido, pagamento aceito, pagamento inexistente, payload inválido e pagamento duplicado são fluxos diferentes. Um contador único de request não cobre o terreno.

## Como subir

```bash
npm install
npm run dev
```

`GET /healthz` responde `status: ok`. A porta padrão é `3000`.
