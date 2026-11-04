# Notas

Copie este arquivo para `entregas/notas.md` e preencha. Apague as instruções entre colchetes.

Este é o caderno. Ele não substitui o relatório. Quem avalia usa estas notas para ver a instrumentação e a linha de raciocínio, e confere se o relatório segue daqui.

## Ordem do trabalho

[Lista curta, em ordem. O que você fez primeiro, o que isso mostrou e o que você decidiu fazer depois.]

1.

## Mapa do serviço

[O que a API faz, quais rotas importam, onde a latência nasce, o que o log já emitia. Fato observado separado do que você supôs.]

## Hipóteses

Uma linha por hipótese, tenha virado sinal no relatório ou não.

| Hipótese | Por que surgiu | O que foi feito | Desfecho |
| --- | --- | --- | --- |
| | | | medida, descartada ou inconclusiva |

No descarte e no inconclusivo, uma frase de motivo abaixo da tabela ou na própria célula. “Não deu tempo” serve se for verdade; “não medi” sem dizer o que foi olhado não serve.

## Decisões

| Pergunta que eu tinha | Decisão | O que isso muda na operação |
| --- | --- | --- |
| | | |

Cabe aqui o que muda o sinal: label, sampling, exporter, retenção, o que o Collector reparte, o que ficou só no processo da API. Decisão de não fazer também entra, com o motivo.

## Sinais, da pergunta à conclusão

Repita por item que entrou no relatório.

### [Mesmo título do relatório]

- **Observação:** [o que você viu antes de instrumentar]
- **Pergunta:**
- **O que confirmaria:**
- **O que derrubaria:**
- **O que foi feito:**
- **Evidência:** [caminho do arquivo]
- **Conclusão:**
- **Confiança:** [alta, média ou baixa, e o que falta para subir]
- **Por que este sinal e não outro:**

## O que ficou de fora do relatório

[Ponto que você viu e decidiu não promover. Motivo: pouco valor operacional, prova insuficiente, fora do tempo. Se não houve, escreva isso.]

## O que mudaria em produção

[O que este laboratório não prova. Sampling, retenção, alerta, cardinalidade, mais de uma instância. Separado do que você entregou.]

## Parada

[Onde você parou de aprofundar e por quê. O fluxo de pagamento já estava correlacionado, o prazo acabou, ou um diferencial ficou de fora de propósito.]
