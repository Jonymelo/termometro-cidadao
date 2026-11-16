# Termômetro Cidadão

**Dashboard ao vivo: https://jonymelo.github.io/termometro-cidadao/**

Solução contra o calor extremo que cruza o **tipo de calor de cada dia** com a **vulnerabilidade de cada município** para descobrir onde o alerta precisa virar prioridade. Global Solution 2026.2 (CitizenTech), FIAP. Grupo Cluster 2.0, turma 2TSCOR.

O calor extremo não derruba casas nem alaga ruas, então quase não vira notícia, mas desidrata, agrava doenças do coração e dos rins e mata, principalmente idoso. O alerta que existe hoje chega para quem já está conectado; quem mais precisa costuma ser quem menos recebe. A ideia aqui é separar o calor em tipos (o seco desidrata, o abafado sobrecarrega o corpo porque o suor não evapora) e, onde um calor perigoso encontra muita gente vulnerável, acionar busca ativa em vez de só mandar um SMS.

## Ver o dashboard

O arquivo `index.html` é a página inteira, sem dependência de servidor. Duas formas de abrir:

- **Local**: baixe e abra o `index.html` no navegador.
- **GitHub Pages (ao vivo)**: https://jonymelo.github.io/termometro-cidadao/

O dashboard tem as telas de tipos de calor, vulnerabilidade, mapa (com busca por município), simulador de alerta e linha do tempo. Funciona no celular.

## Estrutura

```
.
├── index.html                      dashboard (GitHub Pages)
├── notebook/
│   └── termometro_cidadao.ipynb    pipeline completo, do INMET bruto ao data warehouse
└── dados/
    ├── estacoes_diario.csv.gz      INMET consolidado por estação e dia (entrada)
    ├── censo2022_idosos.csv        IBGE Censo 2022, idosos por município (entrada)
    ├── municipios_ibge.csv         malha municipal com coordenadas
    ├── estados.csv                 UFs e regiões
    ├── tabela9514_ibge_censo2022.xlsx   fonte original do IBGE
    └── dw/                         data warehouse gerado pelo notebook (fato + 8 dimensões)
```

## O que o notebook faz

1. Consolida os dados diários do INMET e calcula o índice de calor (o que o corpo "sente").
2. Cria atributos de persistência (dias quentes e noites quentes seguidas).
3. Agrupa os dias com k-means. O cotovelo e a silhueta apontam **5 tipos** (silhueta 0,323).
4. Batiza os tipos pelo perfil do centroide e define 3 níveis de alerta.
5. Traz a vulnerabilidade do Censo 2022 (proporção de idosos) e liga cada município à estação mais próxima (Haversine).
6. Monta o data warehouse (modelo floco de neve, grão município x mês x tipo de dia) e exporta os CSVs.

## Rodar o notebook

```
pip install pandas numpy scikit-learn matplotlib jupyter
jupyter notebook notebook/termometro_cidadao.ipynb
```

Ele lê os arquivos de `dados/` e regrava o data warehouse em `dados/dw/`.

## Resultados

- 458.167 dias-estação analisados, de 599 estações do INMET (2023 a 2025).
- 5 tipos de dia: frio ou ameno (15,7%), ameno e seco (17,0%), quente e úmido (35,4%), calor seco (17,2%) e calor abafado (14,8%).
- 6,1 milhões de idosos vivem em municípios onde mais de um terço dos dias foi de calor de risco alto, de um total de 22,1 milhões no país.
- O Norte tem cerca de 8 vezes mais dias perigosos por município que o Sul; o Nordeste concentra o maior volume de exposição.

## Fontes

- INMET, estações meteorológicas automáticas, 2023 a 2025.
- IBGE, Censo 2022 (tabela 9514, população por idade).
