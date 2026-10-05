# Apuração projetada da eleição presidencial de 2026

Painel que acompanha a apuração do TSE minuto a minuto e corrige o parcial levando em conta a ordem de chegada das seções.

- **Painel:** https://lnmeloni.github.io/apuracao-2026/
- **Noite do 1º turno, congelada:** https://lnmeloni.github.io/apuracao-2026/1turno-2026.html
- **Metodologia:** https://lnmeloni.github.io/apuracao-2026/metodo.html

Autor: Luís Meloni (FEA-USP).

## O problema

O TSE divulga os votos na ordem em que os boletins das seções chegam, e essa ordem não é aleatória. As seções do
Sul e do Sudeste costumam ser totalizadas antes, e as do Norte e do Nordeste, depois. Por isso, no começo da noite,
o parcial é uma amostra enviesada do eleitorado. Em 2022, Bolsonaro liderou o parcial do 2º turno até perto de 68%
apurado e perdeu por 1,8 ponto.

## O que o modelo faz

Para cada município que já começou a apurar, calcula o **delta**, isto é, a parcela atual do candidato menos a parcela
do candidato do mesmo campo na eleição anterior (em 2026: Lula contra Lula 2022, Flávio Bolsonaro contra Jair
Bolsonaro 2022).

- Municípios (ou zonas, nas cidades com mais de uma) que já começaram a apurar são projetadas pelo **próprio parcial**. O número de votos que ainda falta é estimado a partir do comparecimento observado.
- Unidades que ainda não apuraram recebem o resultado da eleição anterior **mais o delta médio do seu estado**,
  ponderado pelos votos já contados.
- Nas 190 cidades com mais de uma zona eleitoral, a unidade é a **zona**. Uma zona sem voto recebe o delta das
  outras zonas da mesma cidade; uma cidade sem voto, o do estado.
- O resultado nacional é a soma das unidades, cada uma ponderada pelos votos válidos esperados.

A margem de erro soma um bootstrap sobre as unidades que já apuraram e um termo de erro sistemático que encolhe
conforme a apuração avança. Abaixo de 2% apurado a estimativa aparece esmaecida, marcada como instável.

Cada arquivo do TSE é conferido antes de ser usado (campos presentes, votos que não diminuem, válidos menores
que o comparecimento, seções coerentes com o eleitorado, horário que não volta). Arquivos inconsistentes são
descartados, e a unidade mantém a última leitura válida.

## Como foi testado

As noites de 2022 foram reconstruídas seção a seção a partir dos boletins de urna do TSE, que registram a hora de
chegada de cada seção. Os dados reconstruídos foram montados no mesmo formato do site de resultados e processados
pelo mesmo código usado ao vivo.

| noite | eleição de comparação | erro médio na parcela de Lula | margem de 95% contém o resultado |
|---|---|---|---|
| 1º turno 2022 (ensaio) | 1º turno 2018 | 0,13 ponto | 100% dos minutos |
| 2º turno 2022 (ensaio) | 1º turno 2022 | 0,17 ponto | 100% dos minutos |
| **1º turno 2026 (noite real)** | **1º turno 2022** | **0,20 ponto** | **100% das leituras** |

O erro médio é calculado de 2% a 99% apurado. Na noite de 04/10/2026, a estimativa ficou a menos de 1 ponto do resultado
final de Lula (45,16%) desde as 17h29, com 2% apurado; o parcial do TSE só atingiu essa distância às 20h38. A partir
de 2% apurado, todas as leituras gravadas indicaram na liderança o candidato que terminou à frente, Flávio
Bolsonaro.

Em 2026 o estimador ficou para trás do gravador no pico da apuração, e a série tem intervalos de 25 a 50
minutos sem estimativa entre 18h48 e 20h04. Os arquivos do TSE foram todos gravados, então esses trechos podem ser
recalculados.

Houve também um ensaio com falhas simuladas, em que 5% das requisições falhavam e 3% dos arquivos chegavam
corrompidos. O erro médio foi de 0,16 ponto.

### O que foi testado e não entrou

- **Regressão do delta com tamanho do município, penalização ridge e ajuste por microrregião.** Teve bom desempenho nas noites
  de 2022, mas errou o dobro da média por estado num ensaio fora da amostra (1º turno de 2018 com base em 2014).
- **Covariáveis do Censo 2022** (urbanização, renda, escolaridade, evangélicos, idade, desigualdade, pobreza),
  contínuas, em quartis ou em células estado × quartil. Melhoram o resultado num par de eleições e pioram em outro,
  porque a relação entre o perfil do município e o delta muda de uma eleição para outra.
- **Pesquisa da véspera como ponto de partida da noite.** Melhora o erro do começo em no máximo 0,05 ponto.

## Limites

- Antes de 2% apurado a estimativa pode errar mais de 4 pontos, e nos primeiros minutos errou mais de 10,
  porque a média de cada estado é calculada com pouquíssimos municípios.
- A margem de erro foi dimensionada nos mesmos ensaios em que o modelo foi escolhido.
- Os números por estado e por município são um passo intermediário e erram mais que o total nacional.

O painel corrige o parcial oficial do TSE com dados públicos. Não usa pesquisa eleitoral.
