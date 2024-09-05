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
