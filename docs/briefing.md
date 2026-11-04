# Briefing

O laboratório é o serviço deste repositório: uma API de pagamentos em Node.js e TypeScript. A stack sobe na máquina de quem faz o desafio. Escopo, prazo e canal de dúvida estão em [`escopo.md`](escopo.md).

A Hous3 olha três coisas, nesta ordem de importância prática: como você instrumenta o serviço, se a linha de raciocínio fecha, e se outra pessoa consegue operar o que você entregou. Quantidade de painel não ganha de nenhuma das três.

## O que entregar

1. **Relatório** em `entregas/relatorio.md`, a partir de [`template-relatorio.md`](template-relatorio.md). É o documento que outra pessoa usaria para subir a stack, abrir o painel e seguir um pagamento até o trace. Sem diário de bordo aqui.
2. **Notas** em `entregas/notas.md`, a partir de [`template-notas.md`](template-notas.md). É o caderno: ordem do trabalho, mapa do serviço, pergunta operacional, o que você mediu, o que descartou e por quê. A conclusão do relatório tem que caber nestas notas.
3. **Evidência** em `entregas/evidencias/`: print de painel, um trace e uma linha de log do mesmo fluxo. Sem segredo de ambiente.

## O que uma boa entrega mostra

No relatório:

- Dá para subir o serviço e a stack seguindo o texto, com porta e o sinal de que cada peça subiu.
- Dá para saber o que olhar primeiro quando um pagamento falha, sem ler o caderno.
- Há pelo menos dois dashboards, cada um com a pergunta que responde.
- Há um trace ponta a ponta e o passo para reproduzir.
- Métrica, log e trace do mesmo fluxo se encontram. Se Loki entrou, o log aponta para o trace.

Nas notas:

- Dá para reconstruir a ordem do trabalho.
- Cada sinal do relatório começa numa pergunta, não numa lista de exporter.
- O que ficou de fora aparece, com o motivo.
- Trade-off de cardinalidade, sampling, retenção ou caminho do sinal está explícito.
- O que mudaria em produção está separado do que o laboratório faz.

## Dúvida

Dúvida de escopo vai pelo canal escrito em `escopo.md`. Durante a conversa técnica, as notas e o relatório são o ponto de partida: o que você mediu e o que faria diferente em produção.
