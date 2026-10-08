# Apuração projetada da eleição presidencial de 2026

Painel que acompanha a apuração do TSE minuto a minuto e estima o resultado final levando em conta a ordem de
chegada das seções. Usa só os dados públicos do TSE. Não usa pesquisa eleitoral e não é página oficial do TSE.

- **Painel:** https://lnmeloni.github.io/apuracao-2026/
- **Noite do 1º turno, congelada:** https://lnmeloni.github.io/apuracao-2026/1turno-2026.html
- **Nota de método:** https://lnmeloni.github.io/apuracao-2026/metodo.html

Autor: Luís Meloni (FEA-USP).

## O problema

O TSE divulga os votos na ordem em que os boletins das seções chegam, e essa ordem não é aleatória. As seções do
Sul e do Sudeste costumam ser totalizadas antes, e as do Norte e do Nordeste, depois. No começo da noite, o
parcial é uma amostra enviesada do eleitorado. Em 2022, Bolsonaro liderou o parcial do 2º turno até perto de 68%
apurado e perdeu por 1,8 ponto.

O parcial do TSE é exato para o que já foi contado. A estimativa trata do que falta contar.

## O que o modelo faz

Para cada município que já começou a apurar, o modelo calcula o **delta**: a parcela atual do candidato menos a
parcela do candidato do mesmo campo na eleição anterior. Em 2026, Lula foi comparado com Lula 2022, e Flávio
Bolsonaro com Jair Bolsonaro 2022.

- Municípios que já começaram a apurar são projetados pelo **próprio parcial**. O número de votos que ainda
  falta neles sai do comparecimento observado.
- Municípios que ainda não apuraram recebem o resultado da eleição anterior **mais o delta médio do seu estado**,
  ponderado pelos votos já contados.
- Nas 190 cidades com mais de uma zona eleitoral, a unidade é a **zona**. Uma zona sem voto recebe o delta das
  outras zonas da mesma cidade; uma cidade sem voto, o do estado.
- O resultado nacional é a soma das unidades, cada uma ponderada pelos votos válidos esperados.

A margem de erro soma um bootstrap sobre as unidades que já apuraram e um termo de erro sistemático que encolhe
conforme a apuração avança. Abaixo de 2% apurado a estimativa aparece esmaecida, marcada como instável. O painel
mostra quem lidera, a diferença em pontos e se ela está dentro da margem de erro. Não mostra probabilidade de
vitória.

Cada arquivo do TSE é conferido antes de ser usado. Os campos têm de estar presentes, os votos não podem diminuir,
os válidos ficam abaixo do comparecimento, as seções têm de ser coerentes com o eleitorado e o horário não pode
voltar. Um arquivo inconsistente é descartado, e a unidade mantém a última leitura válida.

## Como foi testado

As noites de 2022 foram reconstruídas seção a seção a partir dos boletins de urna do TSE, que registram a hora de
chegada de cada seção. Os dados reconstruídos foram montados no mesmo formato do site de resultados e processados
pelo mesmo código usado ao vivo.

| noite | eleição de comparação | erro médio na parcela de Lula | margem de 95% contém o resultado |
|---|---|---|---|
| 1º turno 2022 (ensaio) | 1º turno 2018 | 0,13 ponto | 100% das leituras |
| 2º turno 2022 (ensaio) | 1º turno 2022 | 0,17 ponto | 100% das leituras |
| **1º turno 2026 (noite real)** | **1º turno 2022** | **0,20 ponto** | **100% das leituras** |

O erro médio é calculado minuto a minuto, de 2% a 99% apurado. A nota de método compara versões do modelo em
sete marcos, de 2% a 70%, e por isso os números de lá são maiores.

Na noite de 04/10/2026, a estimativa ficou a menos de 1 ponto do resultado final de Lula (45,16%) desde as 17h29,
com 2% apurado. O parcial do TSE só chegou a essa distância às 20h38. Desde 2% apurado, todas as leituras
gravadas puseram Flávio Bolsonaro à frente, como no resultado final.

No pico da apuração de 2026, o cálculo da estimativa atrasou em relação à coleta dos arquivos, e a série tem
intervalos de 25 a 50 minutos sem estimativa entre 18h48 e 20h04. Todos os arquivos do TSE foram guardados, e
esses trechos podem ser recalculados.

Houve também um ensaio com falhas simuladas, em que 5% das requisições falhavam e 3% dos arquivos chegavam
corrompidos. O erro médio foi de 0,16 ponto.

### O que foi testado e não entrou

- **Regressão do delta com tamanho do município, penalização ridge e ajuste por microrregião.** Foi bem nas noites
  de 2022, mas errou quase o dobro da média por estado num ensaio fora da amostra (1º turno de 2018 com base em 2014).
- **Covariáveis do Censo 2022** (urbanização, renda, escolaridade, evangélicos, idade, desigualdade, pobreza),
  contínuas, em quartis ou em células estado × quartil. Melhoram o resultado num par de eleições e pioram em outro,
  o que sugere que a relação entre o perfil do município e o delta muda de uma eleição para outra.
- **Pesquisa da véspera como ponto de partida da noite.** Melhora o erro do começo em no máximo 0,05 ponto.

## Limites

- Antes de 2% apurado a estimativa pode errar mais de 4 pontos. Nos primeiros minutos do ensaio do 1º turno de
  2022 errou mais de 10, porque a média de cada estado sai de pouquíssimos municípios. Na noite de 2026, o maior
  erro abaixo de 2% foi de 3,2 pontos.
- A margem de erro foi dimensionada nos mesmos ensaios em que o modelo foi escolhido. A noite de 04/10/2026 é o
  primeiro teste que não foi usado para escolher nada.
- O modelo também calcula números por estado e por município, mas o painel não os mostra porque erram mais que o
  total nacional.

---

Desenvolvido por Luis Meloni, com auxílio do Claude Code na implementação. As escolhas metodológicas, dados e
conteúdo são de responsabilidade do autor; eventuais erros ou imprecisões são de minha responsabilidade.
