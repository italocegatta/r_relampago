# (PART\*) Parte I  {-}

# Manipulação de dados {-}


## Warm-Up {-}


### Conhecendo os dados {-}

Vamos começar importando os dados do arquivo `filmes_imdb.csv`. Para indicar que o arquivo está dentro de uma pasta, é preciso utilizar a barra `/`, 


```r
library(tidyverse)

df <- read_csv2("input/filmes_imdb.csv")

df
```

Primeiro vamos fazer um histograma para poder ver a distribuição da variável `nota`.


```r
ggplot(data = df, aes(x = nota)) +
  geom_histogram() 
```

Agora, vamos ver a relação entre o `valor_captado` e o `publico` dos filmes.


```r
ggplot(data = df, aes(x = orcamento, y = faturamento)) +
    geom_point()
```



```r
ggplot(df, aes(orcamento, faturamento)) +
  geom_point() +
  geom_smooth()
```


### Qual o lucro médio dos filmes? {-}

Nosso objetivo agora é calcular o lucro médio dos filmes. Primeiro vamos criar uma coluna e calcular o lucro de cada filme.


```r
df2 <- mutate(df, lucro = faturamento - orcamento)
df2
```

Vamos isolar os valores de lucro e colocar em um objeto e em seguida calcular a média.


```r
vec_lucro <- pull(df2, lucro)

mean(vec_lucro)
```

Vamos refazer os 2 primeiros passos unindo os comandos em um só.


```r
vec_lucro <- pull(mutate(df, lucro = faturamento - orcamento))
mean(vec_lucro)
```

Seguindo a mesma ideia, podemos unificar todos os comandos em uma única chamada.


```r
mean(pull(mutate(df, lucro = faturamento - orcamento), lucro))
```

Agora utilizando um operador especial chamado `pipe`, vamos executar as mesmas funções, porém de forma organizada e de fácil interpretação.


```r
df %>% 
  mutate(lucro = faturamento - orcamento) %>% 
  pull(lucro) %>% 
  mean()
```


## Sobre o Tidyverse {-}

Neste curso utilizaremos como referência os pacotes vinculados ao `tidyverse`, grupo de funções que utilizam a mesma filosofia de programação e foram desenvolvidos para atuarem em conjunto. O [tidyverse](https://www.tidyverse.org/) é mantido por um time de desenvolvedores do RStudio e liderado pelo seu idealizador [Hadley Wickham](http://hadley.nz/).

Há diversas funções disponíveis nos pacotes do `tidyverse` que tem um equivalente direto nos pacotes `base` do R, mas com uma implementação mais moderna e consistente que facilita a estruturação do código. No decorrer do curso vamos ter vários exemplos desse comparativo.

A manipulação de dados é, na maioria das vezes, realizado com `data.frames` e por isso iremos ver as principais funções que lidam com essa estrutura de forma rápida e prática.

O pacote `dplyr` é hoje um dos pacotes mais utilizados para esta finalidade. Ele disponibiliza diversas funções que são “equivalentes” às funções básicas do R, mas como melhorias que nos poupam tempo e deixam o código muito mais fácil de interpretar.

Como exemplo, vamos realizar uma análise exploratória dos dados de um inventário na floresta amazônica.


```r
library(tidyverse)
library(readxl)
```


```r
inv_amazonia <- read_excel("input/inv_amazonia.xlsx")

inv_amazonia
```


### Filter {-}

Com a função `filter()` é possível selecionar linhas específicas, de acordo com o fator que se deseja. Podem ser usados um ou vários fatores de seleção.


```r
filter(inv_amazonia, cap > 300)
```


```r
filter(inv_amazonia, cap > 300 & qf > 1)
```


```r
filter(inv_amazonia, cap > 300 | hcom > 15)
```


```r
filter(inv_amazonia, especie == "IPE")
```


```r
filter(inv_amazonia, especie %in% c("COPAIBA", "IPE"))
```


### Arrange {-}

Para ordenar as colunas, podemos usar a função `arrange()`. A hierarquia é dada pela sequência dos fatores que são adicionados como argumentos da função.


```r
arrange(inv_amazonia, cap)
```


```r
arrange(inv_amazonia, -cap)
```


### Select {-}

A função `select()` auxilia-nos na seleção de variáveis (colunas).


```r
select(inv_amazonia, especie, cap)
```


```r
select(inv_amazonia, especie:cap)
```


```r
select(inv_amazonia, -(familia:comercial))
```


### Mutate {-}

Para criar novas variáveis, podemos usar a função `mutate()`. Um diferencial dessa função em relação à função base do R, é que podemos utilizar variáveis criadas dentro do próprio comando.


```r
mutate(
  inv_amazonia,
  dap = cap / pi,
  secao = pi * dap^2 / 4
)
```

Note que se quisermos utilizar os dados calculados no futuro, temos de salvar em um objeto. No caso, vamos salvar no mesmo objeto `inv_amazonia2` de forma que ele será atualizado com as novas colunas.


```r
inv_amazonia2 <- mutate(
  inv_amazonia,
  dap = cap / pi
)
```


### Summarise {-}

A função `summarise` nos permite resumir dados. Também é possível resumir dados em função de vários fatores com o `group_by`.


```r
summarise(inv_amazonia2, dap_medio = mean(dap))
```


```r
summarise(inv_amazonia2, hcom_medio = mean(hcom))
```


```r
summarise(inv_amazonia2, hcom_medio = mean(hcom, na.rm = TRUE))
```


```r
inv_amazonia_by_esp <- group_by(inv_amazonia2, especie)

summarise(inv_amazonia_by_esp, dap_medio = mean(dap))
```


```r
inv_amazonia_by_esp <- group_by(inv_amazonia2, especie)

resumo_esp <- summarise(
  inv_amazonia_by_esp,
  n = n(),
  dap_medio = mean(dap, na.rm = TRUE),
  hcom_medio = mean(hcom, na.rm = TRUE)
)

resumo_esp 
```


```r
filter(resumo_esp, dap_medio > 50)
```


### Operador %>% {-}

O pacote `dplyr` foi desenhado para trabalhar em conjunto que o operador em cadeia `%>%`. O que esse operador faz é aplicar o que está no LHS no primeiro parâmetro da função do RHS. Podemos também direcionar o local onde o conteúdo do LHS será aplicado informando um `.` como argumento.


```r
inv_amazonia2 %>%
  group_by(especie) %>% 
  summarise(
  n = n(),
  dap_medio = mean(dap, na.rm = TRUE),
  hcom_medio = mean(hcom, na.rm = TRUE)
  ) %>% 
  filter(dap_medio > 50)
```


```r
inv_amazonia %>% 
  filter(comercial == "Sim") %>% 
  select(especie, cap) %>% 
  arrange(desc(cap)) %>% 
  slice(1:5)
```


### Gráficos rápidos {-}


```r
inv_amazonia2 %>% 
  group_by(especie) %>% 
  summarise(dap_medio = mean(dap, na.rm = TRUE)) %>% 
  arrange(desc(dap_medio)) %>% 
  slice(1:20) %>% 
  ggplot(aes(especie, dap_medio)) +
    geom_col() +
    coord_flip() +
    theme_bw()
```


```r
inv_amazonia2 %>% 
  group_by(parcela) %>% 
  summarise(n = n()) %>% 
  arrange(desc(n)) %>% 
  ggplot(aes(factor(parcela), n)) +
    geom_col() +
    theme_bw()
```


```r
inv_amazonia2 %>% 
  ggplot(aes(dap, hcom)) +
    geom_point() +
    theme_bw()
```


```r
inv_amazonia2 %>% 
  ggplot(aes(dap, hcom, color = comercial)) +
    geom_point() +
    geom_smooth() +
    theme_bw()
```
