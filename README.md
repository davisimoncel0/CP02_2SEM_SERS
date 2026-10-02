# CP02 — APIs, Energias Renováveis e Aprendizado de Máquina

Este projeto foi desenvolvido para o **Checkpoint 02 de SERS — 2º semestre**, com o objetivo de aplicar técnicas de **Aprendizado de Máquina** utilizando dados obtidos de **APIs públicas** relacionadas a energias renováveis e condições meteorológicas.

O trabalho é dividido em duas tarefas independentes:

1. **Classificação da fonte de geração de energia**, utilizando dados da ANEEL.
2. **Regressão da radiação solar**, utilizando dados históricos meteorológicos do Open-Meteo.

Em cada tarefa foram treinados e comparados **três algoritmos diferentes de Machine Learning**.

---

## Integrantes

| Integrante | RM |
|---|---:|
| Augusto de Souza Ávila | 570839 |
| Davi Simoncelo | 571738 |
| João Pedro Sousa | 573962 |
| Matheus Evangelista Silva | 568593 |
| Murilo Lima de Carvalho | 570156 |

---

# Objetivo

O objetivo deste projeto é consultar dados reais disponibilizados por APIs públicas, prepará-los para análise e aplicar algoritmos de aprendizado de máquina em dois tipos diferentes de problema:

- **Classificação multiclasse:** identificar se um empreendimento de geração de energia é Solar, Eólico ou Hidráulico.
- **Regressão:** estimar a radiação solar horizontal em Petrolina (PE) a partir de condições meteorológicas.

Além do treinamento dos modelos, são realizadas análises exploratórias, tratamento e seleção das variáveis, comparação de métricas e interpretação dos resultados obtidos.

---

# Fontes dos dados

## 1. ANEEL — SIGA

Os dados utilizados na tarefa de classificação são provenientes do:

**SIGA — Sistema de Informações de Geração da ANEEL**

Fonte oficial:

https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel

A consulta é realizada por meio da API pública CKAN/DataStore da ANEEL e não exige token de autenticação.

### Dados utilizados

Foram consultados empreendimentos das seguintes categorias:

- `UFV` — Solar Fotovoltaica
- `EOL` — Eólica
- `UHE` — Usina Hidrelétrica
- `PCH` — Pequena Central Hidrelétrica
- `CGH` — Central Geradora Hidrelétrica

As categorias `UHE`, `PCH` e `CGH` foram agrupadas na classe **Hidráulica**.

O notebook consultou:

- 1.200 registros de Solar;
- 1.200 registros de Eólica;
- 221 registros de UHE;
- 537 registros de PCH;
- 718 registros de CGH.

Após a preparação, o conjunto possui **3.876 empreendimentos válidos**.

### Período dos dados

O SIGA é um cadastro de empreendimentos de geração mantido pela ANEEL. A consulta utilizada neste trabalho representa os registros disponíveis no cadastro **no momento da execução da API**, incluindo empreendimentos em diferentes fases.

Portanto, os dados não representam um intervalo temporal específico nem a energia efetivamente produzida pelos empreendimentos.

---

## 2. Open-Meteo

Os dados utilizados na tarefa de regressão são provenientes da:

**Open-Meteo Historical Weather API**

Documentação oficial:

https://open-meteo.com/en/docs/historical-weather-api

A API também pode ser consultada sem token para esta aplicação.

### Local analisado

**Petrolina — Pernambuco**

Coordenadas aproximadas:

- Latitude: `-9.39`
- Longitude: `-40.50`
- Fuso horário: `America/Recife`

### Período analisado

Os dados meteorológicos compreendem o período:

**01/04/2025 a 30/06/2025**

Inicialmente foram recebidos **2.184 registros horários**.

Para esta análise foram mantidas somente observações entre **07h e 17h**, resultando em:

**1.001 registros válidos.**

Os dados históricos da Open-Meteo são provenientes de modelos e reanálises meteorológicas e não representam medições realizadas diretamente por um painel fotovoltaico.

---

# Arquivos gerados

Durante a execução do notebook são gerados dois arquivos CSV:

```text
aneel_classificacao_orange.csv
meteo_regressao_orange.csv
```

O primeiro contém os dados utilizados na tarefa de classificação e o segundo contém os dados utilizados na regressão.

---

# Tarefa 1 — Classificação da fonte renovável

## Objetivo

Classificar a fonte de geração de um empreendimento como:

- Solar
- Eólica
- Hidráulica

utilizando somente sua **potência outorgada e localização geográfica**.

---

## Variáveis utilizadas

### Entradas — X

```text
potencia_kw
latitude
longitude
```

### Variável alvo — y

```text
fonte
```

A variável `fonte` possui três classes:

```text
Solar
Eólica
Hidráulica
```

Informações como nome do empreendimento, código CEG e a própria sigla original da fonte não foram utilizadas como entradas, pois poderiam revelar diretamente a resposta ao modelo.

---

## Análise dos dados

O conjunto final possui:

```text
3876 registros
```

Distribuição das classes:

| Fonte | Registros |
|---|---:|
| Hidráulica | 1476 |
| Solar | 1200 |
| Eólica | 1200 |

Não foram identificados valores ausentes nas variáveis utilizadas.

A potência apresenta uma grande variação entre os empreendimentos, indo de valores inferiores a 1 kW até valores superiores a milhões de kW.

Também foi observada uma distribuição geográfica característica entre as fontes, especialmente pela concentração de empreendimentos eólicos em determinadas regiões do Brasil.

---

## Divisão dos dados

Foi realizada uma divisão de:

```text
80% para treinamento
20% para teste
```

utilizando:

```python
random_state=42
stratify=y
```

A estratificação preserva aproximadamente a proporção das três classes nos conjuntos de treino e teste.

Para os modelos sensíveis à escala, foi utilizado `StandardScaler`, ajustado somente sobre os dados de treinamento para evitar **data leakage**.

---

## Algoritmos utilizados

Foram comparados três classificadores:

1. **K-Nearest Neighbors — KNN**
2. **Logistic Regression**
3. **Random Forest Classifier**

---

## Métricas

Como o problema possui três classes, Precision, Recall e F1 foram calculados utilizando:

```text
average = "macro"
```

Dessa forma, cada classe possui o mesmo peso no cálculo final da métrica.

### Resultados

| Modelo | Accuracy | Precision Macro | Recall Macro | F1 Macro |
|---|---:|---:|---:|---:|
| K-Nearest Neighbors | 0.9652 | 0.9663 | 0.9636 | 0.9648 |
| Logistic Regression | 0.8247 | 0.8282 | 0.8214 | 0.8197 |
| Random Forest | **0.9768** | **0.9777** | **0.9758** | **0.9765** |

Também foram geradas matrizes de confusão individuais para os três classificadores.

---

## Conclusão da classificação

Entre os modelos analisados, o **Random Forest apresentou os maiores valores das métricas avaliadas**, alcançando aproximadamente **97,7% de F1 Macro** e **97,7% de Precision Macro**.

Na matriz de confusão foram encontrados principalmente erros envolvendo a classe Solar:

- 7 empreendimentos solares classificados como Hidráulicos;
- 4 empreendimentos solares classificados como Eólicos;
- 3 empreendimentos Eólicos classificados como Hidráulicos.

Apesar dos resultados obtidos, potência e localização não são suficientes para identificar com certeza a fonte de um empreendimento em uma aplicação real.

Empreendimentos de diferentes fontes podem apresentar características semelhantes de potência e localização. Informações climáticas, geográficas, físicas e técnicas adicionais poderiam aumentar a capacidade de generalização do modelo.

---

# Tarefa 2 — Regressão da radiação solar

## Objetivo

Estimar a **radiação solar global horizontal**, em W/m², para Petrolina (PE), utilizando condições meteorológicas e a hora local.

---

## Variáveis utilizadas

### Entradas — X

```text
temperatura_c
umidade_pct
nuvens_pct
vento_kmh
hora
```

### Variável alvo — y

```text
radiacao_w_m2
```

A coluna `data_hora` é utilizada apenas para identificação e ordenação temporal e não é utilizada diretamente como entrada do modelo.

A variável alvo `radiacao_w_m2` também não é utilizada para gerar nenhuma característica de entrada.

---

## Dados meteorológicos

O conjunto final possui:

```text
1001 registros
```

e não apresenta valores ausentes.

As observações correspondem às horas compreendidas entre:

```text
07:00 e 17:00
```

durante o período entre abril e junho de 2025.

---

## Algoritmos utilizados

Foram treinados três algoritmos de regressão:

1. **Linear Regression**
2. **Random Forest Regressor**
3. **Decision Tree Regressor**

---

## Métricas de avaliação

Os modelos foram comparados por:

- **MAE — Mean Absolute Error**, em W/m²;
- **MSE — Mean Squared Error**, em (W/m²)²;
- **R² — Coeficiente de determinação**.

### Resultados obtidos no notebook

| Modelo | MAE | MSE | R² |
|---|---:|---:|---:|
| Regressão Linear | 121.59 | 24816.37 | 0.6211 |
| Random Forest | **48.97** | **5003.24** | **0.9236** |
| Árvore de Decisão | 70.28 | 11274.06 | 0.8278 |

Também são apresentados gráficos comparando os valores reais e os valores previstos pelos modelos.

---

## Importância da hora do dia

A variável `hora` é especialmente relevante para a previsão de radiação solar.

A quantidade de radiação recebida varia naturalmente ao longo do dia:

- no início da manhã, a radiação tende a ser menor;
- próximo ao meio do dia, ocorre maior incidência solar;
- no final da tarde, a radiação diminui novamente.

Entretanto, somente a hora não é suficiente.

Cobertura de nuvens, umidade, temperatura e demais condições atmosféricas podem alterar significativamente a quantidade de radiação que chega à superfície.

---

## Conclusão da regressão

Nos resultados atualmente gerados pelo notebook, o **Random Forest Regressor apresentou os menores erros e o maior R² entre os três algoritmos**.

O modelo obteve:

```text
MAE = 48.97 W/m²
MSE = 5003.24 (W/m²)²
R² = 0.9236
```

O resultado mostra uma relação forte entre as variáveis meteorológicas escolhidas e a radiação solar registrada no conjunto analisado.

A Árvore de Decisão também apresentou resultados relevantes, enquanto a Regressão Linear apresentou maior dificuldade para representar as relações não lineares existentes entre horário, condições meteorológicas e radiação solar.

---

# Radiação solar não é geração elétrica

A previsão de `radiacao_w_m2` não representa diretamente a quantidade de energia elétrica produzida por um sistema fotovoltaico.

A radiação solar indica a quantidade de potência solar incidente em determinada área.

A geração elétrica real depende também de diversos outros fatores, como:

- área dos painéis;
- eficiência dos módulos fotovoltaicos;
- orientação e inclinação;
- temperatura dos módulos;
- sombreamento;
- perdas elétricas;
- inversores utilizados;
- condições e degradação do sistema.

Portanto, prever radiação solar é apenas uma das etapas necessárias para estimar a geração elétrica de uma instalação fotovoltaica.

---

# Tecnologias utilizadas

O projeto utiliza:

```text
Python
Pandas
NumPy
Matplotlib
Seaborn
Scikit-learn
urllib
json
csv
```

Os principais recursos do Scikit-learn utilizados incluem:

```text
KNeighborsClassifier
LogisticRegression
RandomForestClassifier

LinearRegression
RandomForestRegressor
DecisionTreeRegressor

StandardScaler
train_test_split

accuracy_score
precision_score
recall_score
f1_score
confusion_matrix

mean_absolute_error
mean_squared_error
r2_score
```

---

# Como executar

## 1. Clonar o repositório

```bash
git clone URL_DO_REPOSITORIO
```

Entre na pasta:

```bash
cd CP02_2SEM_SERS
```

---

## 2. Instalar as dependências

É necessário possuir Python instalado.

As principais bibliotecas podem ser instaladas com:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

---

## 3. Abrir o notebook

Execute:

```bash
jupyter notebook
```

Em seguida, abra:

```text
SERS_2SEM_CP02.ipynb
```

O notebook também pode ser executado utilizando o **Google Colab**.

---

## 4. Executar as células

Execute todas as células **na ordem em que aparecem no notebook**.

As primeiras etapas realizam as consultas às APIs e geram automaticamente:

```text
aneel_classificacao_orange.csv
meteo_regressao_orange.csv
```

Depois são executadas:

```text
Análise exploratória
Preparação dos dados
Treinamento dos modelos
Previsões
Cálculo das métricas
Visualizações
Conclusões
```

As APIs utilizadas nas consultas deste projeto não necessitam de token.

---

# Estrutura do projeto

```text
CP02_2SEM_SERS/
│
├── README.md
├── SERS_2SEM_CP02.ipynb
├── aneel_classificacao_orange.csv
└── meteo_regressao_orange.csv
```

---

# Conclusão geral

O projeto demonstrou duas aplicações distintas de aprendizado de máquina utilizando dados relacionados ao setor energético.

Na classificação, foi possível identificar padrões entre **potência, localização e fonte de geração**, com desempenho elevado principalmente nos modelos KNN e Random Forest.

Na regressão, as condições meteorológicas e a hora local permitiram estimar a radiação solar com resultados significativamente melhores nos modelos baseados em árvores do que na regressão linear.

Os resultados também demonstram a importância de compreender as limitações dos dados: a classificação da fonte não deve depender exclusivamente de potência e localização, assim como a estimativa de radiação solar não representa diretamente a geração elétrica de um sistema fotovoltaico.

---

# Referências

- ANEEL — SIGA — Sistema de Informações de Geração  
  https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel

- ANEEL — Recurso utilizado pela API  
  https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel/resource/11ec447d-698d-4ab8-977f-b424d5deee6a

- Open-Meteo — Historical Weather API  
  https://open-meteo.com/en/docs/historical-weather-api
