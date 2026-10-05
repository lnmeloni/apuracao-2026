# Apuração projetada — eleição presidencial de 2026

Painel que acompanha a apuração do TSE minuto a minuto e corrige o parcial pela ordem de chegada das urnas.

- **Painel:** https://lnmeloni.github.io/apuracao-2026/
- **Noite do 1º turno, congelada:** https://lnmeloni.github.io/apuracao-2026/1turno-2026.html
- **Metodologia:** https://lnmeloni.github.io/apuracao-2026/metodo.html

Autor: Luís Meloni (FEA-USP).

## O problema

O TSE divulga os votos na ordem em que os boletins das seções chegam, e essa ordem não é aleatória. Sul e
Sudeste apuram antes, Norte e Nordeste depois. O parcial do começo da noite é uma amostra torta do país. Em 2022,
Bolsonaro liderou o parcial do 2º turno até perto de 68% apurado e perdeu por 1,8 ponto.

## O que o modelo faz

Para cada município que já começou a apurar, calcula o **delta**: a parcela do candidato agora menos a parcela
do candidato do mesmo campo na eleição anterior (em 2026: Lula contra Lula 2022, Flávio Bolsonaro contra Jair
Bolsonaro 2022).

- Quem já apurou fica com o **próprio parcial**. O tamanho do que falta sai do comparecimento observado.
- Quem ainda não apurou recebe o resultado da eleição anterior **mais o delta médio do seu estado**, ponderado
  pelos votos já contados (um efeito fixo de UF).
- Nas 190 cidades com mais de uma zona eleitoral, a unidade é a **zona**. Uma zona sem voto recebe o delta das
  outras zonas da mesma cidade; uma cidade sem voto, o do estado.
- O país é a soma, com cada lugar pesando pelos votos válidos que deve ter.

A margem de erro soma um bootstrap sobre as unidades que já apuraram e um termo de erro sistemático que encolhe
conforme a apuração avança. Abaixo de 2% apurado a estimativa aparece esmaecida, marcada como instável.

Cada arquivo do TSE é conferido antes de ser usado (campos presentes, votos que não diminuem, válidos menores
que o comparecimento, seções coerentes com o eleitorado, horário que não volta). Arquivo inconsistente é
descartado e a unidade fica com a última leitura boa.

## Como foi testado

As noites de 2022 foram reconstruídas seção a seção a partir dos boletins de urna do TSE, que trazem a hora de
chegada de cada seção, servidas no mesmo formato do site de resultados e processadas pelo mesmo código que roda
ao vivo.

| noite | base | erro médio na parcela de Lula | margem de 95% contém o resultado |
|---|---|---|---|
| 1º turno 2022 (ensaio) | 1º turno 2018 | 0,25 ponto | 99% dos minutos |
| 2º turno 2022 (ensaio) | 1º turno 2022 | 0,24 ponto | 98% dos minutos |
| **1º turno 2026 (ao vivo)** | **1º turno 2022** | **0,20 ponto** | **100% dos minutos** |

Erro médio a partir de 2% apurado. Na noite de 04/10/2026, a estimativa ficou a menos de 1 ponto do resultado
final de Lula (45,16%) desde as 17h29, com 2% apurado; o parcial do TSE só chegou lá às 20h38. A partir de 2%
apurado, a estimativa nunca pôs o candidato errado na frente.

Também houve um ensaio com dados falhos (5% das requisições falhando e 3% dos arquivos chegando corrompidos de
propósito): erro médio de 0,16 ponto, com todos os arquivos corrompidos barrados e nenhum verdadeiro descartado.

### O que foi testado e não entrou

- **Regressão do delta com tamanho do município, penalização ridge e ajuste por microrregião.** Ia bem nas noites
  de 2022, mas errou o dobro da média por estado num ensaio fora da amostra (1º turno de 2018 com base em 2014).
- **Covariáveis do Censo 2022** (urbanização, renda, escolaridade, evangélicos, idade, desigualdade, pobreza),
  contínuas, em quartis ou em células estado × quartil. Melhoram um par de eleições e pioram outro: a relação entre
  o perfil do município e o delta muda de uma eleição para outra.
- **Pesquisa da véspera como ponto de partida da noite.** Melhora o erro do começo em no máximo 0,05 ponto.

## Limites

- Nos primeiros 2% apurados a estimativa erra de 1 a 3 pontos: a média de cada estado sai de pouquíssimos
  municípios.
- A margem de erro foi dimensionada nos mesmos ensaios em que o modelo foi escolhido.
- Os números por estado e por município são um passo intermediário e erram mais que o total nacional.

**Isto não é pesquisa eleitoral.** É uma correção do parcial oficial do TSE, feita com dados públicos.
