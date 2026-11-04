# Regras de engajamento

O desafio só existe neste repositório e na stack que você sobe para ele. Tudo fora do [`escopo.md`](escopo.md) fica de fora.

## Permitido

- Instrumentar o serviço com OpenTelemetry: trace no servidor HTTP, nos handlers e na chamada interna.
- Expor métrica de latência, throughput, erro e payload em endpoint compatível com Prometheus ou via OTLP.
- Log estruturado no mesmo fluxo, com trace id e span id.
- Subir Prometheus, OpenTelemetry Collector, Grafana e Jaeger localmente.
- Loki, alerta, script de carga e automação de subida, como diferencial.

## Proibido

- Mudar o contrato das rotas para produzir gráfico. A resposta de cada rota continua a mesma.
- Alvo que não esteja em `escopo.md`.
- Commit de `.env`, volume de Prometheus, Grafana ou Loki, token e senha de painel.

## Entrega

O material fica neste repositório, na branch e no prazo combinados no escopo. Duas peças em Markdown: o documento de operação e as notas com a linha de raciocínio. Evidência sem segredo.
