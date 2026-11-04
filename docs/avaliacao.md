# Como avaliar a entrega

Uso de quem corrige o desafio. Os critérios são os mesmos que a pessoa candidata leu na rubrica: instrumentação, linha de raciocínio e operação.

Leia as duas peças. O relatório sozinho não mostra o raciocínio. As notas sozinhas não mostram se a pessoa sabe entregar algo operável.

## Ordem de leitura

1. **`entregas/notas.md`.** Reconstrua o trabalho em ordem. Marque onde a pergunta nasce, que sinal a enfrenta e qual foi o desfecho (medida, descartada, inconclusiva).
2. **`entregas/relatorio.md`, como quem entra de plantão.** Sem voltar às notas: dá para subir a stack? O resumo diz o que olhar primeiro quando um pagamento falha?
3. **As duas lado a lado.** Todo painel e todo trace do relatório têm pergunta e evidência nas notas? Alguma decisão grave das notas sumiu do relatório? O que o relatório chama de correlacionado é o que as notas mostram?
4. **`entregas/evidencias/`.** O arquivo prova a frase? Segredo de ambiente ficou de fora?
5. **Regras.** Contrato das rotas, alvos do escopo, o que não entra no git. Qualquer quebra vai anotada antes da nota.

## O que anotar em cada eixo

Responda em uma frase, com o trecho da entrega que sustenta.

**Instrumentação.** Dá para ver o `POST /api/v1/payments` no trace, da entrada HTTP até a chamada interna? Latência, throughput, erro e payload existem, com rótulo que não explode? O log do pagamento carrega trace id e span id?

**Raciocínio.** A cadeia pergunta → sinal → decisão → evidência → conclusão está escrita? Há caminho descartado com motivo, ou só o que deu certo? O inconclusivo está marcado como inconclusivo? Relatório e notas concordam?

**Operação.** Outra pessoa sobe Prometheus, Collector, Grafana e Jaeger pelo texto? Os dois dashboards respondem perguntas diferentes? O trace tem passo de reprodução? A retenção está dita?

## Nota

Use a escala de [`rubrica.md`](rubrica.md). Escreva o número e a frase. Não medie os eixos num único “passou” antes de registrar os três: instrumentação boa com documento que não sobe a stack é um resultado, e o contrário também.

| Eixo | Nota | Frase |
| --- | --- | --- |
| Instrumentação | | |
| Linha de raciocínio | | |
| Operação | | |

## Sinais que costumam enganar

- Contador único de request, sem o fluxo de pagamento. Conte como ruído até o caminho do pagamento aparecer.
- Relatório longo com pouca decisão. Tamanho não é instrumentação.
- Notas ricas e relatório que não sobe a stack, ou que descreve um painel que a evidência não mostra.
- Id de pedido, de pagamento ou de cliente como label. Isso pesa na instrumentação.
- Loki, alerta ou carga no lugar de Prometheus, Collector, Grafana e Jaeger. Diferencial não cobre o obrigatório.

## O que a avaliação não pede

Não compare a pessoa com um gabarito de dashboard. O que pesa é um fluxo central do serviço, como o pagamento que falha ou o pagamento duplicado, sem hipótese e sem descarte nas notas. Citar o fluxo e deixá-lo de fora com motivo é raciocínio, mesmo se o motivo for fraco: erre a conclusão nas notas, e registre o erro, sem tratar o descarte como ausência de trabalho.
