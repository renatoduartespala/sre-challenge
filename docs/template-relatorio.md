# Relatório — Reliability Engineer

Copie este arquivo para `entregas/relatorio.md` e preencha. Apague as instruções entre colchetes.

Este é o documento de operação. A ordem do trabalho, as hipóteses e os descartes ficam em [`template-notas.md`](template-notas.md). Aqui entra só o que alguém precisa para subir a stack, priorizar e seguir um pagamento.

## Resumo

[Cinco a dez linhas. O que olhar primeiro quando um pagamento falha. Sem lista de ferramenta.]

| Sinal | Onde olhar | Pergunta que responde |
| --- | --- | --- |
| | | |

## Como rodar

[Serviço e stack. Comando, porta e o que indica que cada peça subiu. Retenção: quanto tempo o laboratório guarda métrica, trace e, se houver, log.]

## Escopo

[O que foi instrumentado, a janela e o que ficou de fora.]

## Painéis

Do mais útil ao menos útil. Pelo menos dois. Repita o bloco.

### [Nome do dashboard]

- **Pergunta:**
- **Onde:** [Grafana: porta e caminho do painel]
- **O que a curva mostra:**
- **Evidência:** [caminho em `entregas/evidencias/`]

## Trace

[Um fluxo ponta a ponta. Rota, o que fazer para repetir, onde abrir no Jaeger e qual evidência prova o caminho.]

- **Fluxo:**
- **Como repetir:**
- **Onde abrir:**
- **Evidência:**

## Correlação

[Como o log, a métrica e o trace do mesmo pagamento se encontram. Se não houver Loki, diga como o log da aplicação carrega o trace id.]

## Decisões

[Trade-off que muda a operação: sampling, cardinalidade, exporter, retenção. Uma frase cada.]

## Limitações

[O que o laboratório não permite afirmar. Sem transformar limitação em sinal.]
