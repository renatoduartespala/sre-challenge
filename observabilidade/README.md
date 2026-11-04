# Observabilidade

Terreno de sinal e de stack. Não há cota de dashboard. Na instrumentação, o que conta é se o fluxo de pagamento ficou correlacionado. No raciocínio, se cada sinal responde uma pergunta. Na operação, se outra pessoa sobe a stack e acha o erro.

## Sinais

### Trace

OpenTelemetry no servidor HTTP, nos handlers e na chamada interna (`simulateLatency`). Um trace do `POST /api/v1/payments` mostra o caminho até o pagamento aceito ou recusado.

### Métrica

Latência, throughput, erro e payload, em endpoint compatível com Prometheus ou via OTLP. Label de rota entra. Identificador de pedido, de pagamento ou de cliente não entra como label.

### Log

Log estruturado no mesmo fluxo, com trace id e span id. A linha `payment processed` passa a ser localizável a partir do trace.

## Stack

| Peça | Função neste laboratório |
| --- | --- |
| Prometheus | Guarda a métrica e alimenta o painel |
| OpenTelemetry Collector | Recebe o sinal da aplicação e distribui |
| Grafana | Pelo menos dois dashboards |
| Jaeger | Um trace ponta a ponta, com o passo para repetir |
| Grafana Loki | Diferencial. Log correlacionado com trace e métrica |

O Collector é o ponto de recepção. O destino de métrica, de trace e, se houver, de log fica explícito na configuração e no relatório. A retenção mínima fica escrita: quanto tempo o laboratório guarda, e onde isso está configurado.

## Painéis

Dois dashboards no mínimo. Cada um responde uma pergunta. Exemplos: saúde do serviço, jornada de pagamento, erro por endpoint. O print vai para `entregas/evidencias/`. O relatório diz qual painel abrir e o que a curva mostra.

## Diferencial

- Loki e um painel de log que aponta para o trace.
- Alerta básico no Alertmanager ou no Grafana Alerting.
- Carga com k6, artillery ou autocannon, para o gráfico se mover.
- Automação para subir o ambiente.

O diferencial não cobre a ausência de Prometheus, Collector, Grafana ou Jaeger.

## O que fica no relatório

Como cada processo sobe, em qual porta responde, como validar métrica, trace e log, e qual decisão ficaria diferente em produção.
