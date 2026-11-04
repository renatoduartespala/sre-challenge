![Desafio Técnico // SRE — Reliability Engineer](docs/cover.jpg)

# Desafio SRE — HOUS3 DIGITAL

Este repositório contém o código-base de um serviço em Node.js com TypeScript. O seu objetivo é evoluir a observabilidade dessa aplicação, construindo um ambiente completo de monitoramento e rastreamento.

## Visão Geral

Você deve provisionar localmente (ou em um ambiente de sua preferência) a seguinte stack de observabilidade:

- Prometheus
- OpenTelemetry Collector
- Grafana
- Jaeger
- Grafana Loki *(diferencial / plus)*

A aplicação expõe rotas REST simplificadas que simulam um contexto de pagamentos. O desafio está em instrumentar o serviço, coletar e correlacionar métricas, logs e traces, e transformar esses sinais em painéis úteis para operação e troubleshooting.

## Objetivos Obrigatórios

1. **Instrumentação da aplicação**
   - Implementar tracing distribuído com OpenTelemetry (HTTP server, handlers e chamadas internas).
   - Expor métricas relevantes (latência, throughput, erros, payloads) em um endpoint compatível com Prometheus ou via OTLP.
   - Enviar logs estruturados e correlacionados com traces (trace/span id).
2. **Stack de observabilidade**
   - Subir Prometheus, OpenTelemetry Collector, Grafana e Jaeger.
   - Configurar o Collector para receber dados da aplicação e distribuir para os destinos apropriados (Prometheus, Jaeger, Grafana, Loki).
   - Garantir retenção mínima e documentação de como os serviços são iniciados.
3. **Painéis e exploração**
   - Criar pelo menos 2 dashboards no Grafana com foco em métricas e traces (por exemplo: visão de saúde do serviço, jornada de pagamento, erros por endpoint).
   - Capturar pelo menos um fluxo de trace end-to-end no Jaeger e explicar como reproduzir.
   - Caso utilize Grafana Loki, demonstrar como os logs se correlacionam com traces e métricas.
4. **Documentação**
   - Registrar no README (ou em um arquivo separado) instruções de como rodar a stack, acessar os painéis e validar as métricas/traces/logs.
   - Incluir decisões técnicas, trade-offs e passos futuros sugeridos.

## Bônus (opcional, mas valorizado)

- Configuração do Grafana Loki e painéis de logs.
- Geração de alertas básicos (Prometheus Alertmanager ou Grafana Alerting).
- Scripts de carga (k6, artillery, autocannon etc.) para demonstrar gráficos em movimento.
- Pipelines CI/CD ou automações para iniciar o ambiente.

## Como avaliar o desafio

Você pode disponibilizar sua solução:

- **Forms de solução** com o código, configurações e documentação.

## Informações de Contato

Em caso de dúvidas, entre em contato com [Pedro Cardozo](https://www.linkedin.com/in/pedrocardoz0/). Durante a entrevista técnica falaremos sobre suas decisões e o que faria diferente em produção.

Boa sorte!

