# Rubrica

Três eixos, cada um de 0 a 3. A nota lê o relatório, as notas de raciocínio e a evidência juntos. Volume de painel e volume de configuração não sobem nota.

| Nota | Significado |
| --- | --- |
| 0 | Eixo ausente, ou a entrega quebra a regra de engajamento. |
| 1 | Há material, mas não dá para confiar: a instrumentação não se reconstrói, o raciocínio não liga pergunta a sinal, ou o documento não serve para operar. |
| 2 | O eixo fecha. Outra pessoa do time entende o que foi medido, por que a conclusão é essa e como subir a stack. |
| 3 | Além do item anterior: a pessoa prioriza, explicita o que não sabe e separa o laboratório do que faria em produção. O relatório e as notas contam a mesma história. |

## 1. Instrumentação

A pergunta é se o serviço passou a ser operável, não se a dependência foi instalada.

Olhamos o fluxo de pagamento. Trace no servidor HTTP, nos handlers e na chamada interna. Métrica de latência, throughput, erro e payload. Log estruturado com trace id e span id no mesmo fluxo. A rota continua respondendo o que respondia antes.

Nota 1: biblioteca presente, sinal genérico, ou log sem correlação. Nota 3: dá para ir do erro no painel ao trace e ao log do mesmo pagamento, e a cardinalidade não cresce com id de pedido, de pagamento ou de cliente.

## 2. Linha de raciocínio

A pergunta é se a conclusão está sustentada.

O caminho esperado, nas notas e refletido no relatório: pergunta operacional, sinal que a responderia, o que foi feito, o que foi descartado, trade-off. Conclusão pode ser um sinal que ficou, um caminho abandonado ou um ponto inconclusivo.

Nota 1: a stack aparece pronta, sem origem, ou a evidência não mostra o que o texto afirma. Nota 3: o que ficou de fora está escrito, a confiança declara o que falta, e nenhum painel do relatório fica órfão das notas. Contradição entre as duas peças segura este eixo em 1, mesmo que o texto de cada uma pareça bom sozinho.

## 3. Operação

A pergunta é se a Hous3 conseguiria usar o que foi entregue.

O relatório sobe a stack. Há pelo menos dois dashboards, cada um com uma pergunta (saúde do serviço, jornada de pagamento, erro por endpoint). Há um trace ponta a ponta e o passo para repetir. A retenção mínima está dita. Se Loki entrou, o log aponta para o trace e para a métrica.

Nota 1: stack sem dado, painel vazio, ou “acesse o Grafana” sem porta e sem caminho. Nota 3: quem entra de plantão acha o pagamento que falhou pelo resumo e chega no span sem perguntar de novo onde olhar.

## Sinal que encerra a avaliação

Segredo no git, ou mudança do contrato da API para produzir gráfico. O eixo afetado vai a 0. Se o vazamento é a entrega inteira, a avaliação para.
