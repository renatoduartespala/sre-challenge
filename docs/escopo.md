# Escopo

O alvo abaixo é o único laboratório do desafio. Janela, prazo e branch seguem em branco até a Hous3 abrir a avaliação para uma pessoa.

| Campo | Valor |
| --- | --- |
| Candidato | |
| Janela | |
| Prazo de entrega | |
| Canal de dúvida | [Pedro Cardozo](https://www.linkedin.com/in/pedrocardoz0/) |
| Branch de entrega | |

## Alvos

Liste só o que entra no trabalho. Um alvo que não está aqui fica de fora.

| Alvo | Tipo | Ambiente | Observação |
| --- | --- | --- | --- |
| `src/` | API Node.js | local | Sobe com `npm run dev`. Porta padrão `3000` (`PORT`). |
| Stack de observabilidade | Prometheus, OpenTelemetry Collector, Grafana, Jaeger | local | Loki é diferencial. Roda na máquina de quem faz o desafio. |

## Rotas do serviço

O contrato destas rotas permanece. Latência artificial em `ARTIFICIAL_LATENCY_MS` (padrão 150 ms, mais um jitter até o mesmo valor). Log com pino, nível em `LOG_LEVEL`.

| Método | Rota | Resposta |
| --- | --- | --- |
| `GET` | `/healthz` | `200`, `status: ok` |
| `GET` | `/api/v1/orders` | lista de pedidos |
| `GET` | `/api/v1/payments/:id` | o pagamento, ou `404` |
| `POST` | `/api/v1/payments` | `201`; `400` sem `orderId` ou `amount`; `409` se o pedido já tem pagamento |

## Fora de escopo

- Produção e qualquer serviço que não seja este laboratório.
- Mudar a regra de negócio de pedido e pagamento.
- Exigir conta, cloud ou cluster da Hous3.
