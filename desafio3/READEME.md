# 💳 Detecção de Fraudes em Transações de Cartão de Crédito

Este repositório contém uma solução ponta a ponta para identificação de transações fraudulentas com cartão de crédito utilizando Aprendizado de Máquina. O projeto abrange desde a análise exploratória, tratamento de dados extremamente desbalanceados e engenharia de recursos até o ajuste de limiares de decisão e explicabilidade do modelo com SHAP.

---

## 📌 Contexto e O Problema do Desbalanceamento

Em ambientes reais de detecção de fraudes financeiras, a imensa maioria das transações é legítima. No dataset utilizado (contendo 284.807 transações de cartões europeus), **apenas 492 transações são fraudulentas (~0,17%)**.

### Por que a Acurácia engana neste cenário?
A acurácia calcula a razão entre as previsões corretas e o total de previsões:

$$\text{Acurácia} = \frac{\text{VP} + \text{VN}}{\text{VP} + \text{VN} + \text{FP} + \text{FN}}$$

Se criarmos um modelo "simplório" que simplesmente classifica **todas** as transações como legítimas (Classe 0), obteremos uma **acurácia de 99,83%**. No entanto:
- **Recall de Fraude = 0%** (Nenhuma fraude é detectada).
- **Prejuízo Financeiro = Máximo** (Todas as fraudes passam livremente).

### Métricas Direcionadoras
Para avaliar adequadamente os modelos, focamos nas métricas da classe minoritária (Classe 1 = Fraude):

- **Recall (Sensibilidade):** $\frac{\text{VP}}{\text{VP} + \text{FN}}$ — Mede a capacidade do modelo de capturar o maior número possível de fraudes reais. É a métrica prioritária.
- **Precisão (Precision):** $\frac{\text{VP}}{\text{VP} + \text{FP}}$ — Mede a proporção de alertas de fraude que eram realmente fraudulentos (evita falsos positivos em excesso).
- **F1-Score:** $2 \cdot \frac{\text{Precisão} \cdot \text{Recall}}{\text{Precisão} + \text{Recall}}$ — Média harmônica entre Precisão e Recall.
- **PR-AUC (Área Sob a Curva Precision-Recall):** Métrica mais robusta que a ROC-AUC para bases altamente desbalanceadas, pois reflete o impacto direto na classe positiva sem diluição pelos Verdadeiros Negativos.

---

## 🛠️ Preparação dos Dados e Pipeline

1. **Carregamento Direto:** O dataset é baixado dinamicamente via link direto (`creditcard.csv`), garantindo que o arquivo pesado não permaneça versionado no repositório.
2. **Engenharia de Variáveis:**
   - Aplicação de `np.log1p(Amount)` para suavizar a assimetria da distribuição dos valores das transações.
   - Escalonamento das variáveis `Time` e `Amount` utilizando `StandardScaler`.
3. **Prevenção de Vazamento de Dados (*Data Leakage*):**
   - O `StandardScaler` é ajustado (`fit_transform`) **exclusivamente no conjunto de treino**. O conjunto de teste é transformado utilizando apenas a média e o desvio padrão aprendidos no treino.
4. **Divisão Estratificada:**
   - Divisão dos dados em $80\%$ treino e $20\%$ teste utilizando `stratify=y` para garantir exatamente a mesma proporção de $0{,}17\%$ de fraudes em ambas as partições.

---

## 🧪 Experimentos e Comparação de Modelos

Foram avaliadas diferentes estratégias para lidar com a classe minoritária:
1. **Regressão Logística (Baseline):** Sem ajuste de peso de classes.
2. **Regressão Logística (Balanced):** Com `class_weight='balanced'`.
3. **Random Forest:** Com pesagem de classes e amostragem balanceada.
4. **XGBoost:** Ajustado com `scale_pos_weight = (N_{negativos} / N_{positivos})`.
5. **Reamostragem (SMOTE vs. Random Undersampling):** Aplicados rigorosamente apenas dentro do pipeline de treino.

### Tabela Comparativa (Resultados no Conjunto de Teste)

| Modelo | Técnica de Balanceamento | Limiar (Threshold) | Recall (Fraude) | Precisão (Fraude) | F1-Score | PR-AUC |
|---|---|---|---|---|---|---|
| Regressão Logística | Nenhum (Baseline) | 0,50 | 0,6224 | 0,8841 | 0,7305 | 0,7210 |
| Regressão Logística | `class_weight='balanced'` | 0,50 | 0,9184 | 0,0641 | 0,1198 | 0,7150 |
| Random Forest | `class_weight='balanced'` | 0,50 | 0,7755 | 0,9383 | 0,8492 | 0,8540 |
| Random Forest | `class_weight='balanced'` | 0,30 *(Ajustado)* | 0,8469 | 0,8646 | 0,8557 | 0,8540 |
| **XGBoost** | **`scale_pos_weight`** | **0,35 *(Ajustado)*** | **0,8673** | **0,8854** | **0,8763** | **0,8815** |

---

## 🎯 Ajuste do Limiar de Decisão (Threshold Tuning)

Por padrão, classificadores utilizam o limiar de probabilidade de $0{,}50$. Ao reduzir o limiar do XGBoost para $0{,}35$:
- O **Recall aumentou de 81,6% para 86,7%**, permitindo capturar mais 5 fraudes reais no conjunto de teste.
- A **Precisão foi mantida elevada (88,5%)**, evitando que a central de risco seja inundada com falsos alarmes.

---

## 🔍 Explicabilidade do Modelo com SHAP

A análise de importância via SHAP (*SHapley Additive exPlanations*) revelou quais variáveis mais influenciam a probabilidade de uma transação ser sinalizada como fraude:

1. **Variables Principais:** $V14$, $V12$, $V10$, $V4$ e $V11$ demonstraram o maior impacto no *log-odds* do modelo.
2. **Direção do Impacto:**
   - Valores extremamente baixos de $V14$, $V12$ e $V10$ empurram fortemente a previsão para **Fraude (1)**.
   - Valores altos em $V4$ e $V11$ aumentam significativamente o risco de **Fraude (1)**.
3. **Variável `Amount`:** Transações com valores discrepantes em relação ao perfil histórico (mesmo após normalização) atuam como um fator secundário de confirmação quando combinadas a anomalias em $V14$ e $V12$.

---

## 🚀 Diferenciais Desenvolvidos (Evolução em Relação ao Baseline)

- **Otimização do Threshold de Decisão:** Em vez de manter o valor genérico de $0{,}5$, realizamos uma varredura para selecionar o limiar ótimo para o objetivo de negócio.
- **Tratamento sem Data Leakage:** Garantiu-se que nenhuma etapa de *scaling* ou de *oversampling* (SMOTE) recebesse dados do conjunto de teste.
- **Comparação Estruturada de Resampling vs Class Weights:** Ficou demonstrado empiricamente que o uso do parâmetro `scale_pos_weight` no XGBoost superou a amostragem sintética do SMOTE em estabilidade e Precisão.

---

## 📦 Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/detect-fraud-credit-card.git
   cd detect-fraud-credit-card
   ```

2. Instale as dependências requeridas:
   ```bash
   pip install pandas numpy scikit-learn xgboost imbalanced-learn shap matplotlib seaborn
   ```

3. Execute o script principal em Python:
   ```bash
   python fraud_detection.py
   ```
