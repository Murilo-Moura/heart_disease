# Heart Disease Prediction
 
Análise exploratória e modelagem preditiva para identificação de risco de doença cardíaca a partir de variáveis clínicas, usando Python e Scikit-learn.
 
## Sobre o projeto
 
Segundo a OMS, as doenças cardíacas isquêmicas foram responsáveis por cerca de 9,1 milhões de mortes em 2021, sendo a principal causa de mortalidade global. Este projeto nasceu dessa constatação epidemiológica com um objetivo que vai além de "treinar um modelo de Machine Learning": responder se é possível, a partir de variáveis clínicas de fácil acesso, identificar com confiança pacientes em risco de doença cardíaca.
 
O notebook percorre o ciclo completo de um projeto de ciência de dados aplicado à saúde — da exploração dos dados brutos até a comparação final de modelos — com decisões de engenharia sempre justificadas por raciocínio clínico, e não apenas por performance estatística.
 
> Este repositório contém a versão final do notebook de modelagem. O dashboard interativo (Power BI / Streamlit) faz parte da próxima etapa do projeto e será publicado separadamente.
 
##  Sobre o dataset
 
O dataset combina 5 bases de dados cardíacos independentes (Cleveland, Hungria, Suíça, Long Beach e Statlog), totalizando **918 observações** e **11 variáveis clínicas**, sendo um dos maiores conjuntos públicos disponíveis para pesquisa em doença cardíaca.
 
| Variável | Descrição |
|---|---|
| `Age` | Idade do paciente |
| `Sex` | Sexo (M/F) |
| `ChestPainType` | Tipo de dor no peito (TA, ATA, NAP, ASY) |
| `RestingBP` | Pressão arterial em repouso (mm Hg) |
| `Cholesterol` | Colesterol sérico (mm/dl) |
| `FastingBS` | Glicemia em jejum > 120 mg/dl (1/0) |
| `RestingECG` | Resultado do eletrocardiograma em repouso |
| `MaxHR` | Frequência cardíaca máxima atingida |
| `ExerciseAngina` | Angina induzida por exercício (S/N) |
| `Oldpeak` | Depressão do segmento ST induzida por exercício |
| `ST_Slope` | Inclinação do segmento ST no pico do exercício |
| `HeartDisease` | Variável alvo (1: doença, 0: normal) |
 
## Estrutura do notebook
 
1. **Introdução** — contexto epidemiológico e apresentação do dataset
2. **Bibliotecas e Pré-processamento** — setup e tratamento inicial dos dados (data munging)
3. **EDA — Análise Exploratória** — investigação das relações entre variáveis clínicas e o desfecho
4. **Pré-processamento para o Modelo** — encoding, normalização e construção do pipeline
5. **Baseline: Regressão Logística** — primeiro modelo, usado como referência interpretável
6. **Interpretação dos Dados** — importância das variáveis e coeficientes
7. **Calibração do Modelo** — ajuste de thresholds de decisão
8. **Comparação Final** — Regressão Logística vs. Random Forest vs. Gradient Boosting
9. **Conclusão** — síntese dos aprendizados e próximos passos
## Metodologia
 
**Tratamento de dados**
- 172 registros (~19%) apresentavam `Cholesterol = 0`, um valor clinicamente impossível, tratado como dado ausente e imputado pela mediana segmentada por grupo (`HeartDisease = 0` e `HeartDisease = 1`), preservando as diferenças de distribuição entre os grupos.
**Pipeline de pré-processamento**
Construído com `ColumnTransformer` + `sklearn.Pipeline`, garantindo que todas as transformações aprendidas no treino sejam reproduzidas automaticamente na inferência:
- `StandardScaler` → variáveis numéricas contínuas (`Age`, `RestingBP`, `Cholesterol`, `MaxHR`, `Oldpeak`)
- `OneHotEncoder` (sem `drop_first`) → variáveis categóricas nominais, preservando interpretabilidade clínica
- `OrdinalEncoder` com ordem `[Up → Flat → Down]` → `ST_Slope`, por existir uma progressão clínica real entre os valores
- `passthrough` → `FastingBS`, variável binária já em formato adequado
**Modelagem**
- Divisão treino/teste 80/20 estratificada
- Validação cruzada estratificada (`StratifiedKFold`)
- Três modelos treinados e comparados: Regressão Logística, Random Forest e Gradient Boosting
- **Recall** adotado como métrica principal de decisão, priorizando a redução de falsos negativos — em contexto clínico, deixar de detectar uma doença é mais grave do que um falso alarme
## Resultados
 
| Modelo | Accuracy | F1 | ROC-AUC | PR-AUC | Brier |
|---|---|---|---|---|---|
| Regressão Logística | 0.8696 | 0.8857 | 0.9079 | 0.9016 | 0.1111 |
| **Random Forest** | **0.8913** | **0.9029** | **0.9426** | **0.9450** | 0.0954 |
| Gradient Boosting | 0.8913 | 0.9010 | 0.9376 | 0.9414 | **0.0848** |
 
O **Random Forest** foi selecionado como modelo final por apresentar o maior Recall (~94%) e ROC-AUC (~94%). As variáveis de maior peso preditivo foram `ST_Slope`, `ExerciseAngina`, `Oldpeak` e `ChestPainType` — todas relacionadas à resposta cardiovascular ao esforço físico, o que confere ao modelo coerência clínica além da performance estatística.
 
## Tecnologias utilizadas
 
- **Python 3.12**
- **Pandas** e **NumPy** — manipulação e análise de dados
- **Matplotlib** e **Seaborn** — visualização de dados
- **Scikit-learn** — pré-processamento, pipelines e modelagem
  - `ColumnTransformer`, `Pipeline`, `StandardScaler`, `OneHotEncoder`, `OrdinalEncoder`
  - `LogisticRegression`, `RandomForestClassifier`, `GradientBoostingClassifier`
  - `StratifiedKFold`, `cross_val_score`
  - Métricas: `roc_auc_score`, `recall_score`, `f1_score`, `precision_recall_curve`, `brier_score_loss`
- **Jupyter Notebook** — ambiente de desenvolvimento
## Principais aprendizados
 
- **Recall como prioridade clínica**: em problemas de saúde, a métrica de sucesso deve refletir o custo real dos erros — um falso negativo (doença não detectada) tem consequência muito mais grave que um falso positivo.
- **Pipelines evitam vazamento e inconsistência**: encapsular pré-processamento e modelo em um único `Pipeline` garante que a mesma transformação aplicada no treino seja replicada exatamente na inferência, evitando descompasso entre scaler/encoder e modelo em produção.
- **Encoding com propósito clínico, não só técnico**: variáveis ordinais como `ST_Slope` foram tratadas com `OrdinalEncoder`, respeitando a progressão clínica real entre os valores, enquanto variáveis nominais mantiveram `OneHotEncoder` sem `drop_first` para preservar a interpretabilidade de cada categoria.
- **Imputação sensível ao contexto**: valores impossíveis (colesterol zero) foram tratados com mediana segmentada por classe, preservando diferenças de distribuição entre pacientes doentes e saudáveis.
- **Performance estatística não substitui coerência clínica**: a validação do modelo passou tanto por métricas quantitativas quanto pela checagem de que as variáveis mais importantes fazem sentido do ponto de vista médico.
## Próximos passos
 
O objetivo declarado do projeto é transformar este pipeline em uma ferramenta de apoio à decisão clínica: um dashboard interativo onde médicos possam inserir os dados de um paciente e receber, em tempo real, a probabilidade de risco cardíaco. Duas abordagens estão sendo avaliadas:
 
- **Streamlit**: aplicação web leve, com o pipeline serializado via `joblib`, formulário com as 11 variáveis clínicas e explicação da predição via SHAP values.
- **Power BI + Python**: extensão do dashboard já em construção, executando o pipeline treinado diretamente sobre parâmetros ajustados via slicers.
## Estrutura do repositório
 
```
├── notebook.ipynb   # Notebook completo com EDA, pré-processamento e modelagem
├── heart.csv         # Dataset utilizado (918 pacientes, 11 variáveis clínicas)
└── README.md         # Este arquivo
```
 
## Fonte dos dados
 
Dataset combinado a partir de 5 bases públicas (Cleveland, Hungria, Suíça, Long Beach e Statlog Heart), disponível em [Kaggle](https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction) / UCI Machine Learning Repository.
 
---
 
Projeto desenvolvido como parte de estudos em Machine Learning e Ciência de Dados aplicados à saúde.
