# Perguntas

Este documento descreve as questões estudadas no trabalho, seguindo a estrutura pedida no [escopo](escopo.md): questão, objetivo e motivação, método, conclusão e discussão.

Todas as análises estão em [`notebooks/cesta_basica.ipynb`](../notebooks/cesta_basica.ipynb).

## Preparação dos dados (comum às duas perguntas)

- **Mercados com pouco histórico:** foram removidos os estabelecimentos com menos de 28 meses de coleta (de 48 possíveis), para que as comparações não dependam de mercados vistos poucas vezes.
- **Nomes de produto:** os nomes que mudaram em mar/2026 foram unificados (ex.: `Carne Acem` → `Carne Bovina Acem`, `Pão Francês` → `Pão`).
- **Marcas de café:** as grafias diferentes da mesma marca foram padronizadas (ex.: variações de "3 Corações" e "São Joaquim").
- **Duplicatas:** linhas idênticas foram removidas.
- **Valores irreais:** foram removidos, por produto, os preços por kg (`PPK`) acima de 3× ou abaixo de ⅓ da mediana do produto (ex.: carne a R$ 0,00, banana a R$ 1.000/kg).
- **Variável de comparação:** usamos o `PPK` (preço por kg/L/unidade), porque o tamanho das embalagens varia.

---

## Pergunta 1: Café e suas variabilidades

### Questão

Como o preço do café evoluiu ao longo do período (tendência e sazonalidade)? É possível prever o preço médio do próximo mês?

### Objetivo e motivação

O café teve forte alta de preço no período, e é um dos itens com mais marcas diferentes na cesta. Queremos entender o movimento do preço ao longo do tempo e verificar se o histórico permite antecipar o preço do mês seguinte.

Isso tem uma aplicação direta para o consumidor: a previsão do próximo mês ajuda a decidir se vale antecipar ou adiar a compra.

### Método

1. **Exploração:** série do `PPK` médio mensal por marca (gráfico em painéis, com cada marca em destaque contra as demais), para contextualizar as diferenças entre marcas.
2. **Análise da série:** antes de escolher o modelo, verificamos se a série tem tendência, sazonalidade e se é estacionária.
   - **Autocorrelação:** gráficos ACF e PACF (`plot_acf` / `plot_pacf`).
   - **Decomposição STL:** separação do preço médio mensal em tendência, sazonalidade e resíduo (`STL`, com período de 12 meses).
   - **Estacionariedade:** teste ADF (`adfuller`).
3. **Previsão:**
   - **Modelo:** suavização exponencial de Holt com tendência amortecida, implementada com o `ETSModel(trend="add", damped_trend=True)`, comparada com um baseline ingênuo (`shift(1)`: o preço do próximo mês é igual ao do mês atual).
   - **Validação:** validação cruzada respeitando a ordem do tempo, com janela expansível: cada fold treina com os meses até *t* e prevê *t+1*. O menor treino tem 24 meses, o que dá 24 folds (set/2024 a ago/2026). Não usamos o KFold porque ele treinaria com meses do futuro para prever o passado.
   - **Métrica:** RMSE (raiz do erro quadrático médio, em R$/500g, a mesma unidade da série do notebook). O modelo só é considerado útil se superar o baseline ingênuo.

### Conclusão

- **Tendência:** forte. O preço ficou estável em torno de R$ 19 a R$ 20 (500g) até meados de 2024, subiu até cerca de R$ 35 em meados de 2025 e depois passou a cair devagar, chegando perto de R$ 30 em 2026.
- **Sazonalidade:** não encontrada. A ACF não tem pico no lag 12, e no STL a sazonalidade oscila só entre −R$ 1 e +R$ 1, sem repetir o mesmo desenho todo ano.
- **Estacionariedade:** a série não é estacionária (estatística ADF de −1,28, maior que todos os valores críticos; p-value de 0,64). Por isso a suavização exponencial simples não se aplica, e o modelo indicado é o de Holt.
- **Previsão:** na validação cruzada, o ETS teve RMSE de **R$ 1,06**, contra **R$ 1,37** do baseline ingênuo, e errou menos em 17 dos 24 meses. Ou seja, é possível prever o preço do próximo mês com um erro de cerca de R$ 1.
- A previsão para os próximos 12 meses aponta uma queda leve, até cerca de R$ 27 em agosto de 2027, mas ela não foi validada e serve só como ilustração.

### Discussão

- Com 48 meses de série, há no máximo 4 ciclos anuais, o que limita a estimativa de sazonalidade.
- A alta do café no período foi puxada por fatores externos (clima, câmbio, mercado internacional), que não estão nos dados. Os dados da Conab sugerem uma relação entre o tamanho da safra e o preço, mas são só 6 pontos anuais (2026 e 2027 ainda são estimativas). Isso limita qualquer previsão feita só com o histórico de preços.
- A vantagem do ETS vem da fase de alta (set/2024 a jun/2025). No período recente, com o preço caindo devagar, ele empata com o baseline ingênuo: quando o preço muda pouco, repetir o último valor já é uma boa previsão.
- Nem toda marca aparece em todo mercado e em todo mês, então a média mensal pode refletir a composição da amostra, e não só a variação de preço.

---

## Pergunta 2: Mercados de melhores preços

### Questão

Em qual mercado vale mais a pena comprar cada produto? Existem mercados que são consistentemente baratos (ou caros) ao longo dos anos?

### Objetivo e motivação

Identificar os mercados com preços sistematicamente mais baixos, por produto e no geral. A aplicação é direta: orientar onde comprar cada item da cesta básica. Os códigos dos mercados são anonimizados, então o resultado indica o código, não o nome da loja.

### Método

1. **Preço relativo:** para cada produto e mês, dividimos o `PPK` pela mediana do produto naquele mês (`PPK_Relativo`).
   - Interpretação: 1,0 = na mediana; 0,9 = 10% mais barato; 1,2 = 20% mais caro.
   - Motivo: retirar o efeito da inflação. Sem isso, um mercado coletado só nos meses de preço alto pareceria caro.
2. **Agrupamento:** KMeans com k = 3 sobre o `PPK_Relativo`, por produto e por ano. Os grupos são ordenados pelo centroide: 0 = barato, 1 = mediano, 2 = caro.
   - **Por que k = 3:** o número de grupos foi escolhido pela interpretação que queremos dar a eles, não por um critério estatístico. Três grupos correspondem diretamente às categorias que fazem sentido para o consumidor: barato, "normal" e caro. Com k = 2, perderíamos a faixa intermediária; com mais grupos, as categorias deixariam de ter um significado claro.
3. **Ranking por ano:** para cada produto, o mercado que mais aparece no grupo escolhido. Depois, por ano, os mercados que lideram em mais produtos.
4. **Consistência entre anos:** quantas vezes cada mercado foi o primeiro do ranking em cada ano.
5. **Filtro genérico:** repetição da análise agrupando variações do mesmo item (ex.: cortes de carne → `Carne`, `Frango Sobrecoxa` → `Frango`).

**Validação do ranking:** correlação de Pearson (`pearsonr`), como na autocorrelação da Aula 6 de Séries Temporais, entre o nível de preço de cada mercado em um ano e no ano seguinte. O nível de preço é a mediana do `PPK_Relativo` de cada produto dentro do mercado e, depois, a mediana entre os produtos, para que produtos com mais marcas não pesem mais. Se a correlação for alta e significativa (p ≤ 0,05), um mercado barato continua barato no ano seguinte, ou seja, o resultado não é sorte da amostra.

### Conclusão

**Validação:** em todos os pares de anos a correlação foi alta e significativa:

| Anos | r | p-valor |
|---|---|---|
| 2022 → 2023 | 0,89 | < 0,05 |
| 2023 → 2024 | 0,94 | < 0,05 |
| 2024 → 2025 | 0,92 | < 0,05 |
| 2025 → 2026 | 0,94 | < 0,05 |

Ou seja, existem mercados consistentemente baratos ou caros. Pelo nível de preço médio dos anos:

- **mais baratos:** SUP_06, SUP_04, SUP_46, SUP_47 e SUP_01, entre 11% e 13% abaixo da mediana;
- **mais caros:** SUP_34, SUP_07, SUP_27, SUP_15 e SUP_32, entre 7% e 13% acima da mediana.

O ranking feito com as funções de agrupamento chega a um resultado parecido para os baratos, mas não para os caros: o SUP_33 aparecia como um dos mais caros e, pelo nível de preço, não está entre os 5 mais caros. Isso acontece porque essas funções contam quantas linhas de cada mercado caem em cada grupo, e não quanto o mercado cobra a mais ou a menos.

**Por produto:** em 2026, o SUP_04 e o SUP_01 empataram como mercados de menor preço, com 7 produtos cada, mas em itens diferentes:

- **SUP_04:** arroz, café, pão, farinha e frango sobrecoxa;
- **SUP_01:** feijão, açúcar, carne (acém), ovo e macarrão;
- **os dois:** manteiga e óleo baratos.

Ou seja, não existe um único "melhor mercado": o melhor depende do produto.

### Discussão

- **Viés de ocorrência:** mercados coletados mais vezes, ou que vendem mais marcas, aparecem mais no ranking. Mercados que entraram depois de 2022, ou que não foram visitados em algum ano, ficam em desvantagem.
- **Exclusividade:** um mercado pode sair como "o mais barato" de um produto por ser um dos poucos que o vendem.
- **Anos incompletos:** 2022 tem só set–dez e 2026 só jan–ago, então as contagens entre anos não são diretamente comparáveis.
- **Marca:** um mercado que vende só marcas premium aparece como caro mesmo cobrando o preço normal daquelas marcas.
- **Empates:** a contagem do "primeiro mercado do ano" precisa de um critério explícito de desempate (ex.: menor `PPK_Relativo` mediano).
