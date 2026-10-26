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
2. **Decomposição da série:** separação do preço médio mensal em tendência, sazonalidade e resíduo (`seasonal_decompose` / STL).
3. **Previsão:**
   - **Modelo:** suavização exponencial (ETS / Holt), comparada com um baseline ingênuo (o preço do próximo mês é igual ao do mês atual).
   - **Validação:** temporal, com janela expansível (treino com os meses até *t*, previsão de *t+1*), nunca embaralhando os meses.
   - **Métrica:** RMSE (raiz do erro quadrático médio, em R$/500g, a mesma unidade da série do notebook). O modelo só é considerado útil se superar o baseline ingênuo.

### Conclusão

> *A preencher após a modelagem.* Responder: como foi a tendência do preço no período? Há sazonalidade visível? O ETS superou o baseline ingênuo? Com qual erro (RMSE)?

### Discussão

- Com 48 meses de série, há no máximo 4 ciclos anuais, o que limita a estimativa de sazonalidade.
- A alta do café no período foi puxada por fatores externos (clima, câmbio, mercado internacional), que não estão nos dados. Isso limita qualquer previsão feita só com o histórico de preços.
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

**Avaliação do agrupamento:** silhueta média de cada grupo (barato, normal e caro), calculada ponto a ponto (`silhouette_samples`) em cada agrupamento de produto e ano e depois resumida pela média. Mostra se cada faixa usada no ranking é bem definida.

**Próximos passos:**

- calcular um intervalo de confiança do ranking (bootstrap das observações).

### Conclusão

**Qualidade dos grupos:** a silhueta média foi parecida nos três grupos:

| Grupo | Silhueta média |
|---|---|
| Barato | 0,57 |
| Normal | 0,56 |
| Caro | 0,51 |

Valores acima de 0,5 costumam ser lidos como uma estrutura razoável. Ou seja, a divisão em três faixas, escolhida pela interpretação, também se sustenta nos dados. O grupo caro é um pouco menos definido, provavelmente porque os preços altos ficam mais espalhados (uma "cauda" de preços acima da mediana). Por isso, os resultados sobre mercados caros merecem um pouco mais de cautela do que os sobre mercados baratos.

> *Parcial.* Em 2026, o SUP_04 e o SUP_01 empataram como mercados de menor preço, com 7 produtos cada, mas em itens diferentes:
>
> - **SUP_04:** arroz, café, pão, farinha e frango sobrecoxa;
> - **SUP_01:** feijão, açúcar, carne (acém), ovo e macarrão;
> - **os dois:** manteiga e óleo baratos.
>
> Ou seja, não existe um único "melhor mercado": o melhor depende do produto. *Completar com o ranking entre anos e com os intervalos de confiança.*

### Discussão

- **Viés de ocorrência:** mercados coletados mais vezes, ou que vendem mais marcas, aparecem mais no ranking. Mercados que entraram depois de 2022, ou que não foram visitados em algum ano, ficam em desvantagem.
- **Exclusividade:** um mercado pode sair como "o mais barato" de um produto por ser um dos poucos que o vendem.
- **Anos incompletos:** 2022 tem só set–dez e 2026 só jan–ago, então as contagens entre anos não são diretamente comparáveis.
- **Marca:** um mercado que vende só marcas premium aparece como caro mesmo cobrando o preço normal daquelas marcas.
- **Empates:** a contagem do "primeiro mercado do ano" precisa de um critério explícito de desempate (ex.: menor `PPK_Relativo` mediano).
