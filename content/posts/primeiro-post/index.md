---
title: "Otimização de roteiro através de IA"
date: 2026-08-14
summary: "Roteirização com técnica DBSCAN, OR-Tools e Open Street Map"
showHero: true
tags: ["DBSCAN", "Inteligência Artificial", "Otimização", "Roteirização"]
---

Problemas de roteirização são um tema clássico em Supply Chain, Pesquisa Operacional, com ampla aplicação no segmento de distribuição de utilities. Em linhas gerais, esses problemas buscam determinar rotas otimizadas para atendimento de um conjunto de pontos, considerando restrições financeiras/operacionais e objetivos como a otimização de receita, redução de custos e eficiência operacional.

Para esse projeto pessoal, utilizei LLM para geração de dados sintéticos de georreferenciamento no município de João Pessoa-PB. O objetivo principal é otimizar a rota simultânea de três equipes para a realização de serviços de suspensão de energia. Como dado, além da simulação das informações de posição (latitude e longitude), solicitei geração de dados referentes ao valor devido e o valor esperado de recuperação (que pode vir, a princípio, de modelagem preditiva).

![base_dados](base_dados.png)

O problema, no fundo, é uma variação do Prize-Collecting Travelling Salesman Problem: em vez de visitar todos os pontos, cada equipe parte de um ponto inicial (semente), definindo o roteiro e as paradas que valem a pena dentro de *bounds* de distância máxima percorrida, a capacidade de serviços e a penalização pela própria distância percorrida. A função objetivo maximiza o valor total recuperado menos o custo do deslocamento, respeitando as restrições e sem sobreposição de área entre equipes de mesma natureza (equipes que possuem mesma característica como carro e ferramentas).

Entre as técnicas de roteirização, utilizei para o cálculo da matriz de distância um arquivo do OSM (Open Street Map), o que permite definir rotas com sobreposição da malha, simulando as distâncias da rua e não distâncias euclidianas. Optei pelo OR-Tools como solver para definição das melhores rotas e a segunda etapa de seleção de rotas sem sobreposição. 

O motor de otimização (OR-Tools) recuperou R$ 39 mil contra R$ 26 mil de uma heurística mais simples testada como comparação - cerca de 53% a mais. Já na escolha dos pontos de partida das rotas, troquei a amostragem aleatória por DBSCAN: em vez de sortear candidatos ou elevar muito o número de candidatos sorteados (mais processamento e tempo), ele varre a base inteira em busca de todo ponto que atende à regra de densidade. O ganho nesse caso foi menor: cerca de 1,1% acima da técnica aleatória, mas esse resultado deve ser interpretado considerando que trabalhei com dados mais concentrados e base pequena. Esse resultado tem potencial em uma base maior e mais dispersa, mais próximo da realidade dos serviços.

Outro diferencial desse otimizador está na abertura para conexão com APIs de georreferenciamento. Devido ao limite de requisições gratuitas para essa PoC, foi feito o download da malha viária e processamento offline dos dados (link para download: https://download.geofabrik.de/south-america/brazil.html). Essa técnica faz com que o otimizador consiga calcular a distância sobrepondo a malha viária em vez de considerar apenas a distância euclidiana utilizando a técnica de Haversine, como em métodos mais tradicionais.

Abaixo segue as três rotas definidas para cada equipe.

![Roteiro otimizado](roteiro.png)

Ao final, temos um modelo de roteirização, com a possibilidade de parametrização de dados como o raio para definição de sementes, a distância percorrida, o custo do km percorrido etc. A utilização das técnicas acima permitiu a definição de um roteiro otimizado que potencializa os resultados.

Link para o projeto no meu repositório Github: https://github.com/alonsoguimaraesmarcos/roteirizacao-servicos

Observações:

1. Todo o processo foi realizado e encapsulado dentro de um ambiente virtual, com as bibliotecas definidas para evitar qualquer incompatibilidade entre versões, estando disponível no arquivo de requirements.txt

2. Um detalhe para colocar projetos desse porte para rodar: uma das bibliotecas usadas para processamento dos dados cartográficos depende de um pacote sem instalador pronto para Windows (Visual C++ Build Tools). A saída mais simples foi migrar para o Conda com o Miniforge (sugestão da Claude), que já traz a dependência compilada. 