# Relatório — Reliability Engineer

Este é o documento de operação. A ordem do trabalho, as hipóteses e os descartes ficam em [`notas.md`](notas.md). Aqui entra só o que alguém precisa para subir a stack, priorizar e seguir um pagamento.

## Resumo

Quando um pagamento falha, comece pela taxa de requisições por rota e status. Os status 400 e 409 são respostas do próprio negócio (payload inválido e pedido já pago); um 5xx indica falha do servidor. Em seguida, olhe a latência da rota de pagamento: se o P95 subiu, a espera está no acesso aos dados, que é onde o serviço gasta quase todo o tempo. Para um pagamento específico, abra o trace da requisição e copie o trace id; a linha de log com o mesmo id traz o pagamento (`id` e `orderId`). A métrica diz que algo mudou, o trace diz onde, o log diz qual pagamento.

| Sinal | Onde olhar | Pergunta que responde |
| --- | --- | --- |
| Requisições por rota e status (`http_server_requests_total`) | Grafana `:3001`, painel **Payments — Reliability**, "Requisições por rota e status" | Quantas requisições chegam e quantas falham, por rota? |
| Latência por rota (`http_server_request_duration_seconds`) | Grafana, "Latência P50 / P95 / P99" e "Latência P95 por rota" | Quão lenta está cada rota? |
| Pagamentos por resultado (`payments_total`) | Grafana, "Pagamentos por resultado" | Como terminam as tentativas de pagamento? |
| Trace da requisição | Jaeger `:16686`, serviço `payments-api` | O que aconteceu nesta requisição e onde ela gastou tempo? |
| Log `payment processed` | `docker compose logs api` | Qual pagamento foi criado neste trace? |
| Requisições em andamento (`http_server_active_requests`) | Grafana, cartão "Requisições em andamento" | Há requisições acumulando? |

## Como rodar

Pré-requisito: Docker Desktop em execução.

```powershell
docker compose -f observabilidade\docker-compose.yml up -d --build
```

| Peça | Porta no computador | O que indica que subiu |
| --- | --- | --- |
| API | `3002` (mapeada para a 3000 do container; a 3000 do computador estava ocupada) | `curl.exe http://localhost:3002/healthz` responde 200 |
| Collector | `4317`, `4318` (OTLP), `8889` (métricas) | `docker compose -f observabilidade\docker-compose.yml logs otel-collector` mostra que está pronto, sem erro de exportação |
| Prometheus | `9090` | em `/targets`, o alvo `otel-collector` aparece como UP |
| Jaeger | `16686` | a interface abre e o serviço `payments-api` aparece depois do primeiro tráfego |
| Grafana | `3001` (`admin` / `admin`) | o painel **Payments — Reliability** aparece na pasta **Hous3** |

Para gerar tráfego: `powershell -ExecutionPolicy Bypass -File .\gerar-trafego.ps1`. Para derrubar: `docker compose -f observabilidade\docker-compose.yml down`.

**Retenção.** Não configurei nenhuma; valem os padrões das imagens, e nada tem volume:

- Métrica: Prometheus guarda por 15 dias (padrão), mas os dados vivem no container e somem quando ele é recriado.
- Trace: o Jaeger usa armazenamento em memória e perde tudo ao reiniciar.
- Log: fica no log padrão do Docker enquanto o container da API existir.

## Escopo

Instrumentado: as rotas `GET /api/v1/orders`, `GET /api/v1/payments/:id` e `POST /api/v1/payments`, com foco no fluxo de criação de pagamento. O log existente `payment processed` foi mantido e passou a carregar `trace_id` e `span_id`.

Janela observada: 04/11/2026, em duas execuções de `gerar-trafego.ps1` com 20 rodadas cada, por volta de 21:27 e 21:33 (horário de Brasília). Os logs mostram 40 pagamentos criados nessa janela.

Ficou de fora: `/healthz` (sem span nem métrica), tamanho de payload, agregação de logs, alertas e qualquer teste de carga ou de concorrência.

## Painéis

Do mais útil ao menos útil. As três capturas abaixo estão numa só imagem, `entregas/evidencias/dashboard.png`, porque os painéis ficam no mesmo dashboard.

### Payments — Reliability: requisições por rota e status

- **Pergunta:** quais rotas recebem tráfego e com qual resposta cada uma termina?
- **Onde:** Grafana `http://localhost:3001`, dashboard **Payments — Reliability**, painel "Requisições por rota e status".
- **O que a curva mostra:** duas ondas de tráfego, que coincidem com as duas execuções do gerador. A legenda separa as séries por rota e status: `/api/v1/orders · 200`, `/api/v1/payments · 201` e `/api/v1/payments · 400`, entre outras. Nenhuma série 5xx apareceu no período.
- **Evidência:** `entregas/evidencias/dashboard.png`

### Payments — Reliability: latência P50 / P95 / P99

- **Pergunta:** quão lento está o serviço, e a lentidão é geral ou de poucas requisições?
- **Onde:** Grafana, dashboard **Payments — Reliability**, painel "Latência P50 / P95 / P99" (e "Latência P95 por rota" para separar por rota).
- **O que a curva mostra:** no momento da captura, P50 de 219 ms e P95 de 292 ms. Os traces do mesmo período ficam em torno de 160 a 260 ms nas rotas que acessam dados, e uma requisição rejeitada por payload inválido leva 1,46 ms. A diferença indica que o tempo está no acesso aos dados, não no tratamento da requisição.
- **Evidência:** `entregas/evidencias/dashboard.png`

### Payments — Reliability: pagamentos por resultado

- **Pergunta:** como terminam as tentativas de pagamento?
- **Onde:** Grafana, dashboard **Payments — Reliability**, painel "Pagamentos por resultado".
- **O que a curva mostra:** três séries, `created`, `duplicate` e `invalid`, com o mesmo formato de ondas do tráfego. Não conferi os valores absolutos contra o gerador.
- **Evidência:** `entregas/evidencias/dashboard.png`

## Trace

- **Fluxo:** `POST /api/v1/payments` com resposta 201: a requisição abre um span, consulta os dados (span filho `payments.fetch`) e cria o pagamento.
- **Como repetir:**

```powershell
Invoke-RestMethod -Uri http://localhost:3002/api/v1/payments -Method Post -ContentType 'application/json' -Body '{"orderId":"ord_90001","amount":50.5}'
```

- **Onde abrir:** Jaeger `http://localhost:16686`, serviço `payments-api`, operação `POST /api/v1/payments`. Também dá para colar o trace id no campo "Lookup by Trace ID".
- **Evidência:** `entregas/evidencias/trace-jaeger.png`, trace `bf19e96a726d4356155529b00d965ed5`: 2 spans, 162,77 ms, iniciado em 04/11/2026 às 21:33:58.

## Correlação

O log da aplicação carrega o trace id: o logger (pino) lê o span ativo e acrescenta `trace_id` e `span_id` em cada linha emitida durante uma requisição. Não há Loki; o log sai em JSON no stdout do container.

```powershell
docker compose -f observabilidade\docker-compose.yml logs api --no-log-prefix | Select-String "bf19e96a726d4356155529b00d965ed5"
```

Esse comando devolve a linha do pagamento `ord_64737` (`pay_1793838838936`), do mesmo fluxo do trace acima (`entregas/evidencias/log-correlacionado.png`). O caminho inverso também vale: o `trace_id` de qualquer linha de log abre o trace no Jaeger.

A métrica não carrega o trace id (não usei exemplars). Do painel ao trace, o caminho é por rota, status e horário; do trace ao log, é pelo id.

## Decisões

- **Instrumentação manual** com a API do OpenTelemetry, em vez de automática: a aplicação é ESM e quis controle total sobre os labels.
- **Collector entre a API e os destinos:** a API fala só OTLP; o destino de cada sinal fica em configuração.
- **Exporter:** traces vão do Collector ao Jaeger por OTLP; métricas saem do Collector na porta 8889 e o Prometheus as coleta (a cada 5 s).
- **Cardinalidade:** só `method`, `route` (pelo padrão, como `/api/v1/payments/:id`) e `status_code` viram label; `orderId` e `paymentId` ficam no trace e no log.
- **Sampling:** nenhum configurado, então 100% das requisições são traçadas; adequado ao volume do laboratório.
- **Buckets do histograma** de 50 ms a 5 s, com mais resolução perto da latência observada.
- **Log no stdout, sem Loki:** o requisito de correlação se cumpre com o trace id na linha.

## Limitações

- O volume foi pequeno (40 pagamentos criados) e sequencial; os percentis têm pouca amostra e não houve teste de concorrência.
- Nenhum erro 5xx ocorreu no período, então a taxa de erro 5xx do painel aparece sem dados e **não foi exercitada**.
- Os casos 400 e 409 têm trace e métrica, mas não geram linha de log.
- Os dados não sobrevivem à recriação dos containers (sem volumes).
- Não há alertas nem SLO.
- A API não persiste pagamentos; um pagamento criado não conflita com uma repetição.
- Grafana com credencial padrão, adequado só para uso local.
