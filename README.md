![Desafio técnico — Reliability Engineer](assets/cover.jpg)

# Desafio — Reliability Engineer

Repositório do desafio técnico da vaga de Reliability Engineer na Hous3.

O desafio avalia três coisas:

1. **Como a pessoa instrumenta o serviço.** Métrica, log e trace no fluxo de pagamento, correlacionados, com o contrato da API intacto.
2. **A linha de raciocínio.** Cada sinal nasce de uma pergunta operacional. O que foi medido, o que ficou de fora e o trade-off estão escritos.
3. **Como entrega a operação.** A stack sobe, o painel responde uma pergunta e o texto serve para outra pessoa rodar, achar um erro e seguir um trace.

Painel sem sinal correlacionado não sustenta nenhuma das três. Stack no ar sem dizer o que olhar pesa contra.

## Terreno técnico

As habilidades da vaga são o terreno do teste, não uma lista para marcar.

| Terreno | Onde aparece na condução |
| --- | --- |
| Serviço | API de pagamentos em Node.js e TypeScript: rotas, latência artificial e o log que já existe |
| Observabilidade | OpenTelemetry, métrica compatível com Prometheus, log com trace id; Prometheus, Collector, Grafana e Jaeger |

O detalhe de cada terreno está na pasta correspondente. A nota está em [`docs/rubrica.md`](docs/rubrica.md). Quem corrige a entrega segue [`docs/avaliacao.md`](docs/avaliacao.md).

## Como o desafio funciona

1. Ler o [briefing](docs/briefing.md) e as [regras de engajamento](docs/regras-de-engajamento.md).
2. Confirmar o serviço, a stack e o canal de dúvida em [`docs/escopo.md`](docs/escopo.md). Fora dessa lista, não há trabalho.
3. Instrumentar a aplicação e subir a stack localmente.
4. Entregar duas peças:
   - [`entregas/relatorio.md`](entregas/relatorio.md), a partir de [`docs/template-relatorio.md`](docs/template-relatorio.md) — o documento de operação.
   - [`entregas/notas.md`](entregas/notas.md), a partir de [`docs/template-notas.md`](docs/template-notas.md) — o caderno, com a ordem das decisões.
5. Evidência em `entregas/evidencias/`: painel, trace e log do mesmo fluxo.

## Layout

```
docs/               briefing, regras, escopo, rubrica, avaliação e templates
servico/            terreno da API de pagamentos
observabilidade/    terreno de sinal e de stack
entregas/           relatório, notas e evidências
src/                código-base do serviço
```

## O que não entra no git

`.env`, volume de Prometheus, Grafana e Loki, e segredo de painel. O [`.gitignore`](.gitignore) já cobre o caso comum. Evidência mostra o sinal e omite segredo.
