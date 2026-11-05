# Notas

Este é o caderno. Ele não substitui o relatório ([`relatorio.md`](relatorio.md)).

## Ordem do trabalho

1. **Li o código do serviço.** Mostrou 4 rotas, latência artificial em `simulateLatency` e um único log de negócio (`payment processed`). Decidi tratar `POST /api/v1/payments` como o fluxo principal e não alterar nenhuma resposta, código HTTP ou regra.
2. **Rodei o lint do projeto original.** Falhou com 10 erros, todos em `src/server.ts`. Decidi corrigir só com tipagem e estrutura, antes de qualquer instrumentação, para que o código instrumentado partisse de uma base que passa no lint.
3. **Subi a stack vazia** (API, Collector, Prometheus, Grafana, Jaeger). A API não subiu de primeira: a porta 3000 do computador estava ocupada. Decidi mapear `3002:3000`.
4. **Instrumentei** (métricas, spans, log com `trace_id`) e conferi cada destino: traces no Jaeger, métricas no Prometheus, `trace_id` nas linhas de log. Um erro de compilação (`metricReader`, na versão instalada a opção é `metricReaders`) foi corrigido nessa etapa.
5. **Criei o painel** e gerei tráfego em duas execuções do gerador. O painel mostrou as duas ondas de tráfego e latência em torno de 290 ms no P95.
6. **Fechei a correlação com um caso real:** peguei o `trace_id` de uma linha de log (`bf19e96a726d4356155529b00d965ed5`) e abri o mesmo trace no Jaeger.
7. **Escrevi relatório e notas** só com o que foi visto nas capturas e nos logs.

## Mapa do serviço

**Observado** (no código e nos sinais):

- Rotas: `GET /healthz`, `GET /api/v1/orders`, `GET /api/v1/payments/:id`, `POST /api/v1/payments`.
- A latência nasce em `simulateLatency`: espera entre 1x e 2x o valor configurado antes de devolver os dados. As rotas de pedidos e pagamentos passam por ela; `/healthz` e a rejeição de payload inválido não.
- O `POST /api/v1/payments` responde 400 (sem `orderId` ou `amount`), 409 (`orderId` já existe entre os pagamentos) ou 201.
- O único log de negócio era `payment processed`, emitido só no 201.
- Os dados são fixos em memória; o pagamento criado não é gravado em lugar nenhum.
- Nos traces, uma requisição 400 leva 1,46 ms; as que passam por `simulateLatency` levam cerca de 160 a 260 ms.

**Suposto** (não verificado por execução):

- Que a espera de `simulateLatency` represente "acesso a dados" numa versão real do serviço. É uma leitura minha do nome e da posição no código.
- Que o tratador de erro do Express (3 parâmetros) nunca seja acionado como tratador de erro, pela regra de que o Express exige 4 parâmetros. Li a regra; não executei uma rota inexistente para ver o resultado.

## Hipóteses

| Hipótese | Por que surgiu | O que foi feito | Desfecho |
| --- | --- | --- | --- |
| A latência do `POST /api/v1/payments` vem do acesso aos dados, não do tratamento da requisição | `simulateLatency` é a única espera do fluxo | Span filho `payments.fetch` na etapa de dados | **Medida.** No trace `bf19e96…` o filho cobre quase toda a duração de 162,77 ms; o 400, sem acesso a dados, levou 1,46 ms |
| O pipeline completo funciona: API, Collector, Prometheus, Jaeger | A stack subiu vazia e precisava de prova | Gerei tráfego e consultei cada destino | **Medida.** Traces no Jaeger, métricas no Prometheus e no painel, `trace_id` nos logs |
| Existem respostas 5xx no fluxo | Há um tratador de erro no código | Contador por `status_code` e taxa de erro 5xx no painel | **Inconclusiva.** Nenhum 5xx ocorreu no tráfego gerado e não forcei uma falha; o painel mostra "No data" |
| A latência cresce com a concorrência (saturação) | `http_server_active_requests` existe para isso | Cartão "Requisições em andamento" | **Inconclusiva.** O gerador é sequencial; só vi o valor 0. Não fiz teste de carga |
| O tratador de erro de 3 parâmetros não é acionado como tratador de erro | Regra do Express sobre o número de parâmetros | Nada além de ler o código | **Inconclusiva.** Não executei uma rota inexistente; deixei o código como estava |
| A verificação `import.meta.url === file://${argv[1]}` falha no Windows | Os caminhos do Windows têm formato diferente | Troquei por `pathToFileURL` | **Inconclusiva.** Não rodei a API fora do Docker no Windows para reproduzir; a troca é preventiva |
| O tamanho do payload deveria virar métrica | Era uma dimensão possível do tráfego | Nada | **Descartada.** Nenhuma pergunta operacional deste serviço depende disso |
| `orderId` ou `paymentId` como label de métrica | Seria o jeito direto de achar um pagamento | Nada | **Descartada** por cardinalidade ilimitada; não testei o efeito no Prometheus. O pagamento específico se acha pelo trace e pelo log |

## Decisões

| Pergunta que eu tinha | Decisão | O que isso muda na operação |
| --- | --- | --- |
| Como instrumentar uma aplicação ESM com Express? | Spans e métricas escritos à mão com a API do OpenTelemetry | Controle total dos labels; só enxerga o que foi escrito (chamadas de bibliotecas novas não aparecem sozinhas) |
| A API envia direto aos destinos ou por um intermediário? | Collector no meio, por OTLP | A API não conhece Prometheus nem Jaeger; trocar destino é configuração |
| Como as métricas chegam ao Prometheus? | O Collector as expõe na porta 8889 e o Prometheus coleta a cada 5 s | O alvo do Prometheus é o Collector, não a API; se o Collector cair, a métrica para |
| Que labels as métricas carregam? | `method`, `route` (pelo padrão) e `status_code`; `outcome` nos pagamentos | O número de séries fica limitado; identificar um pagamento exige trace ou log |
| Que fração dos traces guardar? | Nenhum sampling configurado (100%) | Todo trace fica disponível; em volume maior seria preciso amostrar |
| Como medir a latência? | Histograma com buckets de 50 ms a 5 s | Os percentis são estimados dentro de cada bucket; resolução melhor perto de 100 a 500 ms |
| O `/healthz` entra nos sinais? | Não: sem span, sem métrica | Os números refletem só o tráfego de negócio; uma falha do próprio `/healthz` não aparece |
| Onde vive o log? | stdout em JSON, com `trace_id`, sem Loki | A busca é por `docker compose logs`; não há consulta por período nem agregação |
| Quanto tempo guardar os dados? | Não configurei; valem os padrões, sem volumes | Dados somem quando o container é recriado |
| Como o container recebe o sinal de parada? | `CMD ["node", "dist/server.js"]` direto, sem o `npm` | O SDK descarrega traces e métricas pendentes ao parar |

## Sinais, da pergunta à conclusão

### Requisições por rota e status (`http_server_requests_total`)

- **Observação:** o código não media nada; o status só existia na resposta de cada rota.
- **Pergunta:** quantas requisições chegam e quantas falham, por rota?
- **O que confirmaria:** séries separadas por rota e status que acompanhem o tráfego gerado, incluindo 400 e 201.
- **O que derrubaria:** séries misturadas, rotas com o caminho real em vez do padrão, ou ausência de dados com tráfego ativo.
- **O que foi feito:** contador no middleware, com `method`, `route` e `status_code`, e um painel com a taxa por rota e status.
- **Evidência:** `entregas/evidencias/dashboard.png`
- **Conclusão:** as séries aparecem separadas por rota e status (`/api/v1/orders · 200`, `/api/v1/payments · 201`, `/api/v1/payments · 400`) e seguem as duas ondas de tráfego.
- **Confiança:** média. Falta ver um 5xx real e uma rota inexistente (`unmatched`) para provar esses dois caminhos.
- **Por que este sinal e não outro:** é o que separa falha do negócio (4xx) de falha do servidor (5xx), que é a primeira pergunta quando um pagamento falha.

### Latência por rota (`http_server_request_duration_seconds`)

- **Observação:** `simulateLatency` espera entre 1x e 2x o valor configurado; o código não media a duração.
- **Pergunta:** quão lenta está cada rota, e a lentidão é geral ou de poucas requisições?
- **O que confirmaria:** percentis coerentes com as durações dos traces e distinção entre rotas com e sem acesso a dados.
- **O que derrubaria:** percentis incompatíveis com os traces do mesmo período.
- **O que foi feito:** histograma com buckets de 50 ms a 5 s e painéis de P50/P95/P99 e de P95 por rota.
- **Evidência:** `entregas/evidencias/dashboard.png`
- **Conclusão:** P50 de 219 ms e P95 de 292 ms na captura, compatíveis com os traces vistos (162 a 262 ms nas rotas com dados).
- **Confiança:** média. Foram 40 pagamentos, sequenciais; falta volume e concorrência para confiar nos percentis altos.
- **Por que este sinal e não outro:** o histograma permite P95 e P99, que uma média esconderia, e a pergunta sobre lentidão é a segunda a ser feita num incidente.

### Pagamentos por resultado (`payments_total`)

- **Observação:** o 201, o 409 e o 400 só apareciam como código HTTP, misturados com as outras rotas.
- **Pergunta:** como terminam as tentativas de pagamento?
- **O que confirmaria:** as três séries (`created`, `duplicate`, `invalid`) acompanhando o tráfego do gerador.
- **O que derrubaria:** série ausente ou resultado contado na categoria errada.
- **O que foi feito:** contador com o label `outcome`, incrementado no ponto exato de cada desfecho do `POST /api/v1/payments`, e também registrado no span (`payment.outcome`).
- **Evidência:** `entregas/evidencias/dashboard.png`
- **Conclusão:** as três séries existem e seguem o padrão do tráfego.
- **Confiança:** média. Não conferi os valores absolutos contra o que o gerador enviou.
- **Por que este sinal e não outro:** separa o desfecho do negócio da resposta HTTP e evita usar `orderId` como label.

### Trace do `POST /api/v1/payments`

- **Observação:** não havia trace; o único elo entre as etapas era o tempo de resposta.
- **Pergunta:** o que aconteceu nesta requisição e onde ela gastou tempo?
- **O que confirmaria:** um trace por requisição, com o span da rota e um filho para o acesso aos dados.
- **O que derrubaria:** traces sem o span filho ou requisições sem trace.
- **O que foi feito:** span de servidor no middleware (nome pelo padrão da rota) e span filho `payments.fetch`.
- **Evidência:** `entregas/evidencias/trace-jaeger.png`
- **Conclusão:** o trace `bf19e96a726d4356155529b00d965ed5` tem 2 spans e 162,77 ms; o filho ocupa quase toda a duração.
- **Confiança:** alta para este fluxo. Falta ver o trace de uma falha 5xx.
- **Por que este sinal e não outro:** é o único sinal que aponta uma requisição específica e mostra onde o tempo foi gasto.

### Log `payment processed` com `trace_id`

- **Observação:** o log já existia e era estruturado, mas sem ligação com nenhuma requisição.
- **Pergunta:** qual pagamento foi criado neste trace?
- **O que confirmaria:** o `trace_id` do log abrir o mesmo trace no Jaeger.
- **O que derrubaria:** `trace_id` ausente ou diferente do trace da requisição.
- **O que foi feito:** o logger lê o span ativo e acrescenta `trace_id` e `span_id`; a mensagem e o conteúdo do log continuam os mesmos.
- **Evidência:** `entregas/evidencias/log-correlacionado.png`
- **Conclusão:** a busca pelo id `bf19e96a726d4356155529b00d965ed5` devolveu a linha do pagamento `ord_64737`, e o mesmo id abre o trace no Jaeger.
- **Confiança:** alta para o 201. Os casos 400 e 409 não geram log, então não há o que correlacionar neles.
- **Por que este sinal e não outro:** é o elo que leva do trace ao dado de negócio (qual pedido e qual pagamento).

### Requisições em andamento (`http_server_active_requests`)

- **Observação:** o serviço não expunha nada sobre quantas requisições estavam abertas.
- **Pergunta:** há requisições acumulando?
- **O que confirmaria:** o valor subir sob tráfego concorrente e voltar a zero depois.
- **O que derrubaria:** o valor ficar em zero durante rajadas concorrentes, ou não voltar a zero.
- **O que foi feito:** contador up/down incrementado ao abrir e decrementado ao fechar a resposta.
- **Evidência:** `entregas/evidencias/dashboard.png`
- **Conclusão:** só vi o valor 0 no cartão; o gerador é sequencial, então o sinal não foi exercitado.
- **Confiança:** baixa. Para subir, falta um teste com requisições simultâneas.
- **Por que este sinal e não outro:** é o indicador de saturação mais barato de medir neste serviço.

## O que ficou de fora do relatório

- **Tamanho do payload:** pouco valor operacional para este serviço.
- **`/healthz`:** excluído dos sinais para não encobrir o tráfego de negócio.
- **Log nos casos 400 e 409:** vi que esses desfechos têm trace e métrica mas nenhuma linha de log, e não acrescentei um `logger.warn`. Mantive a mudança mínima no código de negócio; fica como a próxima melhoria.
- **O tratador de erro de 3 parâmetros:** não alterei porque corrigir mudaria o comportamento de rotas inexistentes e de JSON inválido, e não verifiquei por execução qual é o comportamento atual.
- **Taxa de erro 5xx como sinal comprovado:** o painel a calcula, mas como nenhum 5xx ocorreu, deixei só como limitação.

## O que mudaria em produção

O laboratório não prova:

- **Sampling:** com volume real, 100% dos traces seria caro; seria preciso amostrar sem perder as falhas.
- **Retenção:** volumes para Prometheus, Grafana e Jaeger (hoje tudo se perde ao recriar) e um armazenamento de traces persistente.
- **Alertas e SLO:** sobre a taxa de erro 5xx e o P95, com limites definidos a partir de tráfego real.
- **Cardinalidade:** com mais rotas e mais status, revisar o número de séries.
- **Mais de uma instância:** acrescentar identificação da instância nas métricas e dimensionar o Collector.
- **Logs agregados:** consulta por período e por `trace_id` sem depender de `docker compose logs`.
- **Ligação métrica → trace:** exemplars, para ir do ponto da curva ao trace.
- **Acesso:** autenticação no Grafana e no Jaeger.

## Parada

Parei quando o fluxo de pagamento já estava correlacionado: o mesmo `trace_id` liga o trace no Jaeger à linha de log do pagamento `ord_64737`, e o painel responde se o fluxo está lento ou falhando. Ficaram de fora, de propósito e por falta de tempo, o teste de carga e de concorrência, a provocação de um erro 5xx para exercitar a taxa de erro, o log dos casos 400 e 409, e o Loki.
