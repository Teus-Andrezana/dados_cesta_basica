# Análise de Cesta Básica

![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-3.0-150458?logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-notebook-F37626?logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/status-em%20andamento-yellow)

Análise exploratória e modelagem de **4 anos de preços de itens da cesta básica** coletados mês a mês em dezenas de supermercados. São mais de 46 mil registros de preço, de café e arroz a carne e pão francês.

O objetivo é responder perguntas práticas para quem faz compras: **como o preço se comporta ao longo do tempo** e **em qual mercado vale mais a pena comprar cada produto**.

## Perguntas

| # | Pergunta | Abordagem |
|---|----------|-----------|
| 1 | **Café e suas variabilidades**: o que explica a variação do preço do café (tempo, marca ou mercado)? Dá para prever o próximo mês? | Decomposição da série, dispersão entre mercados, ETS contra um baseline ingênuo, validação temporal |
| 2 | **Mercados de melhores preços**: em qual mercado vale mais a pena comprar cada produto? | Preço relativo à mediana do mês, agrupamento por tipos de mercado (caro, médio, barato), controle por marca, ranking com intervalo de confiança |

A descrição completa de cada pergunta (objetivo, método, conclusão e discussão) fica em [`docs/perguntas.md`](docs/perguntas.md).

## Dados

[`data/raw/ICB_DADOS_2026.xlsx`](data/raw/ICB_DADOS_2026.xlsx)

| | |
|---|---|
| Registros | 46.264 |
| Período | set/2022 a ago/2026 (48 meses) |
| Estabelecimentos | 51, com códigos anonimizados (`SUP_01`, `ALT_26`, …) |
| Produtos | 14 com série completa; frango, ovo, acém, pernil e macarrão a partir de mar/2025 |

| Coluna | Descrição |
|--------|-----------|
| `Mes_Ano_Data` | Mês de referência da coleta |
| `Data_Coleta` | Dia da coleta (ausente a partir de nov/2025) |
| `Estab` | Código do estabelecimento |
| `Produto` | Item da cesta básica |
| `Marca` | Marca do produto (vazia para itens in natura) |
| `Preco` | Preço da embalagem, em R$ |
| `Quantidade` | Tamanho da embalagem (kg, L ou unidades) |
| `PPK` | Preço por kg/L/unidade (`Preco / Quantidade`). **É a variável usada nas comparações**, já que as embalagens variam |

## Estrutura

```
.
├── data/
│   └── raw/                    # planilha original, sem alterações
├── docs/
│   ├── escopo.md               # escopo e critérios do trabalho
│   └── perguntas.md            # perguntas, métodos e conclusões
├── notebooks/
│   └── cesta_basica.ipynb      # limpeza e análises
└── requirements.txt
```

## Como rodar

```bash
git clone https://github.com/Teus-Andrezana/dados_cesta_basica.git
cd dados_cesta_basica

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter notebook notebooks/cesta_basica.ipynb
```

## Andamento

- [x] Remoção de mercados com pouco histórico (menos de 28 meses)
- [x] Padronização das marcas de café
- [x] Exploração do preço do café por marca
- [ ] Unificação dos nomes de produto que mudaram em mar/2026
- [ ] Remoção de duplicatas e outliers
- [ ] Pergunta 1: variabilidade e previsão do preço do café
- [ ] Pergunta 2: ranking de mercados por produto

## Limitações

- São só 48 meses de série, o que limita modelos sazonais.
- Os mercados são anônimos: o ranking indica *qual código* é mais barato, não o nome da loja.
- A cobertura muda ao longo do tempo: produtos entraram em 2025 e nem todo mercado é coletado todo mês.

## Equipe

Projeto desenvolvido como atividade de extensão em Ciência de Dados ([escopo](docs/escopo.md)).

- Giulia Garrido
- Mateus Andrade Antezana
- William Lana
- Thomaz Dacorso
