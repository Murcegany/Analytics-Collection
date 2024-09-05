# Documentação
Este case visa analisar os dados de arrecadação, identificando padrões e anomalias, e gerando insights valiosos para tomada de decisão. O objetivo principal é fornecer uma visão clara das diferentes categorias de arrecadação, detectando anomalias e exibindo as principais métricas por meio de dashboards visuais.

## Transformação e Limpeza de Dados
1. Limpeza de Dados: Remoção de duplicatas, tratamento de valores nulos e ajuste dos tipos de dados (datas, números, textos).
2. Detecção de Anomalias (Python): Foi aplicado o algoritmo Isolation Forest para detectar anomalias nos valores de arrecadação agrupados por UF, Ano e Tipo de Arrecadação. Anomalias foram visualizadas em um gráfico de dispersão.
3. Unificação de Colunas: Diferentes tipos de arrecadação foram consolidados em uma única coluna chamada Tipo de Arrecadação.
4. Criação de Categoria Geral: Adicionada uma nova coluna que classifica as arrecadações em categorias amplas, como "Impostos" e "IPI", facilitando a análise.
   
Script utilizado para a detectação de anomalias:
```bash
import pandas as pd
from sklearn.ensemble import IsolationForest
import matplotlib.pyplot as plt
import seaborn as sns

# Supondo que dataset é seu DataFrame original
df = dataset

# Agrupar os dados
df_grouped = df.groupby(['UF', 'Ano', 'Tipo de arrecadação'])[['Valor']].sum().reset_index()

# Preparar dados para Isolation Forest
X = df_grouped[['Valor']]

# Aplicar Isolation Forest
iso_forest = IsolationForest(contamination=0.1, random_state=42)
df_grouped['Anomalia'] = iso_forest.fit_predict(X)

# Mapear -1 para 'Anômalo' e 1 para 'Normal'
df_grouped['Anomalia'] = df_grouped['Anomalia'].map({-1: 'Anômalo', 1: 'Normal'})

# Plotar gráfico com informações de Tipo de arrecadação
plt.figure(figsize=(14, 8))
sns.scatterplot(
    data=df_grouped,
    x='UF',
    y='Valor',
    hue='Anomalia',
    style='Tipo de arrecadação',
    palette={'Anômalo': 'red', 'Normal': 'blue'},
    marker='o',
    s=100  # Tamanho dos pontos
)
plt.xlabel('UF')
plt.ylabel('Valor')
plt.title('Detecção de Anomalias na Arrecadação por Tipo de Arrecadação')
plt.legend(title='Anomalia', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.grid(True)
plt.tight_layout()
plt.show()

# Exibir o DataFrame agrupado
print(df_grouped)
```

## Medidas Utilizadas
1. Tipos de Arrecadação (qtde)
Esta medida calcula o número distinto de tipos de arrecadação presentes no dataset, fornecendo uma visão geral da diversidade de tipos de arrecadação.
```bash
Tipos de Arrecadação (qtde) = DISTINCTCOUNT('Arrecadacao-Estado'[Tipo de arrecadação])
```

2. Total de Anomalias (qtde)
Esta medida conta o número total de registros classificados como "Anômalo" na tabela de anomalias, indicando a quantidade de valores que foram identificados como fora do padrão.
```bash
Total de Anomalias (qtde) = COUNTROWS(FILTER('Arrecadacao-Anomalia', 'Arrecadacao-Anomalia'[Anomalia] = "Anômalo"))
```

3. R$ Média de Arrecadação (reais)
Esta medida calcula a média dos valores de arrecadação, proporcionando uma visão sobre o valor médio arrecadado.
```bash
R$ Média de arrecadação (reais) = AVERAGE('Arrecadacao-Estado'[Valor])
```

4. R$ Valor Máximo Arrecadado (reais)
Esta medida retorna o valor máximo de arrecadação registrado, ajudando a identificar o maior valor individual de arrecadação.
```bash
R$ Valor Máximo Arrecadado (reais) = MAX('Arrecadacao-Estado'[Valor])
```

5. R$ Total Arrecadado (reais)
Esta medida calcula o total de arrecadação somando todos os valores, oferecendo uma visão geral do total arrecadado.
```bash
R$ Total Arrecadado (reais) = SUM('Arrecadacao-Estado'[Valor])
```

## Gráficos Utilizados
- Gráfico de Total Arrecadado por Categoria
Este gráfico exibe o total arrecadado dividido por categoria. É útil para visualizar quais categorias contribuem mais para o total de arrecadação e identificar as principais fontes de receita.

- Gráfico de Total Arrecadado por Ano
Este gráfico mostra a arrecadação total agrupada por ano. Ele ajuda a analisar as tendências ao longo do tempo e identificar padrões sazonais ou anuais.

- Gráfico de Pareto
Este gráfico é utilizado para aplicar o princípio de Pareto (80/20), mostrando a contribuição acumulada de cada categoria para o total arrecadado. Ele ajuda a identificar as categorias que mais contribuem para a receita total.
Códidgo utilzado para a criação em R:
```bash
library(ggplot2)
library(dplyr)

# dataset <- data.frame(Ano, Categoria, Valor)
# dataset <- unique(dataset)

total_arrecadado <- sum(dataset$Valor, na.rm = TRUE)

dataset_summary <- dataset %>%
  group_by(Categoria) %>%
  summarise(Valor = sum(Valor, na.rm = TRUE)) %>%
  arrange(desc(Valor)) %>%
  mutate(
    Contribuição = Valor / total_arrecadado * 100,
    Cumulativo = cumsum(Contribuição)
  )

ggplot(dataset_summary, aes(x = reorder(Categoria, -Valor))) +
  geom_col(aes(y = Valor, fill = Valor), color = "black") +
  geom_line(aes(y = total_arrecadado * Cumulativo / 100, group = 1), color = "green", size = 1) +
  geom_point(aes(y = total_arrecadado * Cumulativo / 100), color = "green", size = 2) +
  scale_y_continuous(sec.axis = sec_axis(~ . / total_arrecadado * 100, name = "Porcentagem Acumulada (%)")) +
  scale_fill_gradient(low = "lightgreen", high = "darkgreen") +
  labs(
    title = "Gráfico de Pareto - Contribuição Cumulativa por Categoria",
    x = "Categoria",
    y = "Valor",
    fill = "Valor"
  ) +
  theme_light() +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1),
    plot.title = element_text(hjust = 0.5)
  )
```

# Objetivo dos Gráficos
- Gráfico de Total Arrecadado por Categoria: Identifica quais categorias são mais lucrativas e possibilita a análise do desempenho de cada tipo de arrecadação.

- Gráfico de Total Arrecadado por Ano: Ajuda a observar as variações anuais na arrecadação e a identificar tendências e ciclos sazonais.

- Gráfico de Pareto: Destaca as principais categorias que contribuem significativamente para o total arrecadado, ajudando a focar nos principais contribuintes de receita.
# Requisitos

## Power BI (versão desktop)
- Utilzado a versão de desktop fornecida pela play store.
## Figma (versão web)
- Utilizado a versão web padrão.
## R
Pacotes Necessários:
- ggplot2 (para visualizações gráficas)
- dplyr (para manipulação e transformação de dados)
- tidyr (para limpeza e organização de dados)
## Python
Pacotes Necessários:
- pandas (para manipulação de dados)
- matplotlib (para visualização de dados)
- numpy (para cálculos matemáticos)
