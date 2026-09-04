# Desafio Inferência Causal

<img src = "img\magalu.jpg">

## Table of Contents

- [Overview](#overview)
- [The Causal Inference Scenario](#the-causal-inference-scenario)
- [Foundational Concepts](#foundational-concepts)
- [Choosing the Appropriate Model](#choosing-the-appropriate-model)
    - [Multivariate Regression Model](#multivariate-regression-model)
    - [Integrating Propensity Score Matching (PSM) with Difference-in-Differences (DiD)](#integrating-psm-with-did)
- [Sensitivity Analysis](#sensitivity-analysis)
- [Spillover Effects](#spillover-effects)
- [Respostas](#respostas)
- [References](#references)

## Overview
This document provides an overview of key causal inference concepts and their relevance to analyzing user behavior at Magalu.

## The Causal Inference Scenario

*The underlying question when want to work with Program Evaluation is to be able to find causal inference (as with Econometrics in general): What is the outcome (Y) of the treated group after the treatment was perfomed, had they not been treated?*

Magalu's scenario mirrors a typical observational study setup. Due to operational constraints, a randomized controlled trial (RCT) was not feasible to assess the causal impact of a new recommendation algorithm on user engagement and sales. 

Instead, the algorithm was implemented for a subset of users, selected based on their historical purchase patterns and navigation behavior. 

**This setup introduces potential biases** because the treatment group (users exposed to the algorithm) is not randomly assigned, and **differences between treated and control groups may not be due to the algorithm alone**.

For example:
* Users who are already high spenders or have a history of frequent visits might be more likely to receive the treatment, which could skew the results. **If the analysis simply compares sales or engagement outcomes between treated and control groups, it might incorrectly attribute differences in these outcomes to the algorithm, when in fact they could be driven by the users' previous behavior or characteristics, such as historical spending or access frequency**.

To address this, we can use **Propensity Score Matching (PSM)** to balance the treatment and control groups based on observable characteristics, mitigating the selection bias. 

However, PSM alone does not fully account for time-varying effects or pre-existing trends. Therefore, applying a **Difference-in-Differences (DiD)** approach alongside PSM allows us to control for unobserved, time-invariant confounders and estimate the causal impact of the algorithm by comparing changes in outcomes (e.g., sales or engagement) before and after the treatment for both groups. 

**This combined approach enhances the validity of our causal estimates, providing a clearer picture of the algorithm's impact on user behavior.** 

(*Considerations For the future*) Traditional logistic regression for calculating propensity scores often assumes linear relationships between covariates, which may miss complex interactions or non-linear effects. This limitation can result in inaccurate propensity scores and biased estimates. This could be addressed by using more advanced machine learning techniques, such as **Generalized Boosted Regression Models (GBM)**, a flexible method that can model non-linear interactions and higher-order relationships between covariates, providing more accurate propensity scores and improving causal inference.

Now, let's check some important **foundational concepts**, which will show why we can proceed with the implementation of such method.

## Foundational Concepts

Let's start with some definitions and evaluate how they are related to (and how they affect) our project:

- [Conceptual Framework](#conceptual-framework)
    - [Research Design](#research-design)
    - [Data Structure](#data-structure)
    - [Unit of Observation](#unit-of-observation)
    - [The Magalu Project](#the-magalu-project)
- [Random Variables](#random-variables)
- [Probability Distribution of Random Variables](#probability-distribution-of-random-variables)
- [Random Sample](#random-sample)
- [Independence](#independence)
- [Identical Distribution](#identical-distribution)
- [Common Issues when violating the Identical Distribution assumption](#common-Issues-when-violating-the-Identical-distribution-assumption)
- [Violating the IID Assumptions](#violating-the-iid-assumptions)
- [What could we do](#what-could-we-do)
- [Confounders](#confounders)
- [Imbalance](#imbalance)

## Conceptual Framework

### Research Design
Let's start with Research Design. 

**Definition**: **Research Design** is the framework that guides how a study is conducted, including data collection, analysis, and interpretation

**Types of Research Design**:

Quasi-experimental and natural experiments are specific types of research designs that aim to estimate causal effects when full experimental control (like randomization) is not possible.

**Moving to the Conceptual Framework within Research Design**:
* Research Design
    * **Observational Studies**:
        A subset of research design that specifically refers to studies where the researcher does not manipulate or control the assignment of treatments or exposures.
        * Non-causal (e.g., descriptive studies)
        * **Causal**: Focused on inferring cause-and-effect relationships using statistical methods (e.g., natural experiments, quasi-experiments).
            * **Quasi-Experimental** Designs (e.g., Difference-in-Differences)
                * **Program Evaluation**: A process to assess the effectiveness of interventions, often using causal methods such as quasi-experimental designs (e.g., Difference-in-Differences, Propensity Score Matching) when randomization is not possible.
            * Natural Experiments (e.g., policy changes, geographical assignments)
* Experimental Studies
    * Randomized Controlled Trials (RCTs)

Now, let's move to Data Structure and Unit of Observation.

### **Data Structure**
* Definition: In the context of *Econometrics and Statistical Modeling*, a `Data Structure` refers to the way data is organized in relation to time, units, and observations.
* There are many data structures, such as:
    * Cross-Sectional Data
    * Time Series Data
    * Pooled Cross Sections
    * Panel or Longitudinal Data

* **In this project, we have both Cross-Sectional and Time Series dimensions**, which is the Basis for **Panel Data**.

    **Panel Data** involves repeated observations of the same units over time. In this case, treatment assignment is tracked for the same users (identified by `user_id`) across two `time` periods (time = 0 and time = 1).
    * **Multiple Time Periods**: The same individuals are observed at multiple time points, allowing us to analyze changes over time.
    * **Repeated Observations**: Variables like `Frequência de Acesso`, `Engajamento`, and `Vendas` are measured repeatedly for each user, providing a clear basis for analyzing how treatment affects these metrics.

This panel data structure is critical for methods like **Difference-in-Differences (DiD)**, as it allows us to estimate causal effects by comparing changes within individuals over time.

### **Unit of Observation**
* **Definition**: *Unit of Observation* refers to the entity for which data is collected at each time point.
* For this project, we are observing Magalu's **users**. Therefore, we have **user-level** data over two periods of time.

### The Magalu Project

* We face Observational data of the Causal type, due to the nature of the research question:
    * We are evaluating the causal effect of an intervention (e.g., exposure to a new algorithm) using treated and control groups. 
    * This is a quasi-experiment because it does not involve random assignment of participants into treatment and control groups (RCT was not possible), but rather relies on a naturally occurring or deliberately assigned treatment in a way that mimics experimental conditions.


## Random Variables
**Definition**: In Statistical or Machine Learning terms, Random Variables represent **features** or attributes of the data.

**Example in Magalu Context**

* $X_1$: user_id
* $X_2$: time
* $X_3$: treatment
* $X_4$: segmento
* $X_5$: historico_compras
* $X_6$: frequencia_acesso
* $X_7$: engajamento
* $X_8$: vendas

## Probability Distribution of Random Variables
1. **Marginal Probability Distribution** 
   The probability distribution of a **single random variable** $X$ describes the likelihood of its possible values.
   
$$
P(X = x) \quad \text{(discrete)} \quad \text{or} \quad f_X(x) \quad \text{(continuous)}.
$$

2. **Joint Probability Distribution**:  
   The probability distribution of **multiple random variables** $X$ and $Y$ describes the likelihood of their simultaneous outcomes:
   
$$
P(X = x, Y = y) \quad \text{(discrete)} \quad \text{or} \quad f_{X,Y}(x, y) \quad \text{(continuous)}.
$$

### Random Sample
**Definition**: A **Random Sample** is a set of **Independent** and **Identically Distributed** (IID) Random Variables.

In other words, a **random sample** of size $n$ from a population consists of a set of $n$ **random variables**, which can be defined by $\textbf{X} = (X_1, X_2, \dots, X_n)$ that satisfy the following conditions:

1. Independence
2. Identical Distribution


A **Random Sample** ensures that each observation in the dataset (each data on the **user**) is **drawn from the same probability distribution** and **is independent of the others**.

Below, let's learn about them and see how the violation of such assumptions might lead to issues in Causal Inference. 

### Independence
$$
X_i \perp X_j \quad \text{for all } i \neq j
$$

Independence ensures that the value of one random variable does not influence the value of another. 

There are two key aspects to consider when discussing independence: **Marginal Independence** and **Joint Independence**.

* **Marginal Independence**
    
    Focuses on the distributions of individual random variables.
    * *Magalu's example*: Variables like `Histórico Compras` and `Engajamento` might not be marginally independent. For example, **higher spending users may also engage more with the platform**.

* **Joint Independence**

    Extends Marginal Independence to all possible subsets of random variables and ensures that their joint distribution is the product of their marginal distributions.
    * *Magalu's example*: Variables such as `Frequência de Acesso`, `Engajamento`, and `Vendas` are unlikely to be jointly independent because **access frequency likely influences engagement, which in turn impacts sales**.

### Identical Distribution
$$
X_1, X_2, \dots, X_n \sim F, \quad \text{where } F \text{ is the population distribution}.
$$
$$or$$
$$
X_i \sim X_j \quad \text{for all} \quad i, j
$$

It ensures that all random variables in the sample are drawn from the same probability distribution. This assumption is crucial because it guarantees that the behavior of the random variables is consistent and comparable.

* *Magalu's example*:
    * In this project, **identical distribution** means that every observation (every **user** row in the dataset) comes from the same underlying "data-generating process" (or the same *population*). 
    * In simple terms, this assumes that all users (observations on each `user_id`) behave similarly in terms of how their features (like `Frequência de Acesso`, `Engajamento`, or `Vendas`) are distributed.

There are a few issues that arise from violating the Identical Distribution assumption, see below.

### Common Issues when violating the Identical Distribution assumption
1. **Hetereogeneity** (*see definition below*)

    "**Segments Represent Different Types of Users**"
    
    The dataset includes a `segmento` column (e.g., `novo`, `frequente`, `alto_valor`), which means users belong to different groups. These groups naturally have different behaviors:
    * `Alto_valor` users probably make more purchases (`Vendas`) and have higher `Histórico Compras`.
    * `Novo` users might have lower `Engajamento` and `Frequência de Acesso` because they are new to the platform.
    
    So:
    * **Users** in the `alto_valor` segment likely have a different distribution of `Histórico Compras` compared to users in the `novo` segment.
    * Similarly, `Frequência de Acesso` might be higher for **frequent** users than for **novo** users.
    
    These differences (due to heterogeneity across segments) mean that the features' distributions may vary between segments, violating the **Identical Distribution** assumption.
     

2. **Treatment Assignment vs. Change in Bevavior**
    
    "**Exposure to Treatment Assignment may influence behavior**"
    
    Treatment Assignment refers to whether a user is exposed to the new recommendation algorithm (treatment = 1) or not (treatment = 0). 
    
    This assignment can directly influence user behavior in several ways:

    * **More Accurate Recommendations**: Users exposed to the new algorithm may receive more relevant product suggestions, leading to:
        * Increased Engagement: Spending more time on the platform (engajamento).
        * Higher Sales: Making more or larger purchases (vendas).
    * **Psychological Impact**: Users may become more active simply because they feel part of a test or are receiving a "premium" experience.
    * **Behavioral Changes**: Better recommendations can alter how users interact with the platform, influencing their behavior beyond the immediate effects of the algorithm.
    
    Users in the treatment group (exposed to the new algorithm) may exhibit different behavior compared to those in the control group, even before the intervention, due to inherent differences rather than the treatment itself.

    Example:
    * **User A (treated)**: After receiving improved recommendations, spends more time browsing (`engajamento`) and purchases more items (`vendas`).
    * **User B (control)**: Without the new algorithm, spends less time and makes fewer purchases.

    So: 
    * Behavior differences between the groups complicate causal inference, making it harder to attribute changes solely to the treatment.
    * The Identical Distribution assumption ensures that users, apart from the treatment, should be comparable in terms of characteristics and behaviors.

### Violating the IID Assumptions

*How Violating **Independence** and **Identical Distribution** Might Affect Causal Inference*

1. **Violating Independence**

* **Dependence Between Observations**: If user behaviors are influenced by other users or external factors (e.g., social interactions, platform-wide changes), the assumption of independence is violated.
* **Spillover Effects**: A user’s behavior might be influenced by others’ experiences with the treatment, leading to a situation where the treated group’s behavior affects the control group or vice versa.
* **Distorted Results**: If observations are not independent, it becomes difficult to isolate the true treatment effect, and the analysis may overestimate or underestimate the impact of the treatment.

Violating independence complicates causal inference because we assume that each observation is independent, meaning the outcome for one user should not be influenced by the outcome of another. When this assumption is violated, attributing changes in behavior to the treatment becomes more challenging.

2. **Violating Identical Distribution**
* **Averages can be misleading**: Comparing treatment and control groups without considering differences could distort the true treatment effect.
* **The analysis can be biased**: Differences may be wrongly attributed to the treatment, when they are actually due to underlying variations in user groups or time effects.

We should ask ourselves: *Are the observations (users' data) IID?*
* **Independence**: Users’ behaviors (e.g., spending, engagement) could be influenced by platform-wide effects, such as promotions, seasonal trends, or the introduction of the new algorithm. **Thus, complete independence across users may not hold**.
* **Identical Distribution**: The dataset includes treatment and control groups, which introduces `Heterogeneity`. **Users in the treatment group might have different patterns due to exposure to the algorithm.**

### What could we do?

If any of the IID assumptions are violated, it’s crucial to adjust our analysis to account for potential biases and ensure that the causal inferences we draw are valid. Here are some methods that can help:

1. **Stratify by Segment**

    Divide users into segments (e.g., `novo` vs. `frequente`) to make sure we are comparing similar groups. This approach helps in controlling for differences in behavior or characteristics between groups, leading to more accurate comparisons.

2. **Control for Covariates**

    Include relevant variables, such as `segmento` or `Histórico Compras`, to adjust for differences that might affect the outcome. This helps to isolate the true treatment effect by accounting for confounding factors.

3. **Use Matching**

    Matching methods like **Propensity Score Matching (PSM)** allow us to pair similar users from the treatment and control groups. This helps in balancing the groups based on observed characteristics, reducing bias and improving causal inference.

4. **Robust Statistical Methods**

    If the observations are not strictly IID, consider using:

    * **Robust Standard Errors** or **Clustering** to adjust for correlations or heteroscedasticity within groups.
    * **Difference-in-Differences (DiD)**, which is particularly useful for controlling for time-related trends or group-level dependencies (e.g., changes in behavior over time due to exposure to the algorithm).

## Confounders

* **Definition**: When a covariate is related to both the treatment assignment and the outcome, it becomes a **confounder**. This means it can bias the estimate of the treatment effect unless properly accounted for.

## Imbalance

* **Definition**: **Imbalance** refers to the unequal distribution of a covariate (e.g., `segmento`) between the treatment and control groups. In the context of causal inference, it indicates that the treatment group differs systematically from the control group with respect to this covariate, which can affect the validity of causal estimates.

* **Treatment-Group Differences**: If a covariate like `segmento` is disproportionately represented in the treatment group compared to the control group, it introduces selection bias, where differences in outcomes could be attributed to these pre-existing differences rather than the treatment itself.

* **Diagnostic of Imbalance**: In this analysis, the joint probability distribution revealed that certain levels of segmento (e.g., "`alto_valor`") were overrepresented in the treatment group compared to the control group. This indicates a lack of comparability between the groups with respect to this variable.

Addressing such imbalances (via matching, weighting, or adjusting in regression models) is essential to ensure valid causal inference.

## Choosing the Appropriate Model

### Multivariate Regression Model

To estimate the effect of the new algorithm on `engagement` and `sales`, the most simple method is a **Multivariate Regression Model** with **Adjustment for Confounders**. 

This approach allows:

* **Adjustment for Identified Confounders**:
    * The data indicate that `segmento`, `histórico de compras`, and `frequencia_acesso` are **potential confounders**. 
    * These factors are associated with both the treatment and the outcomes, making it essential to adjust for them to isolate the causal effect of the treatment.

* **Direct Estimation of the Treatment Effect**:
    * The coefficient associated with the treatment variable in the regression model $\beta_{treatment}$ will provide a direct estimate of the treatment effect on `engagement` and `sales`, adjusting for the influence of confounders.

### Integrating PSM with DiD

While **Difference-in-Differences (DiD)** is effective for estimating causal effects in panel data, it relies on the assumption that treatment and control groups are comparable.

If there are **imbalances**, i.e., *systematic pre-existing differences in characteristics between these groups*, **Propensity Score Matching (PSM)** can be used as a pre-treatment adjustment to improve the balance between the groups.

* **Imbalance and Its Impact**: Imbalance occurs when covariates, such as *user characteristics*, are distributed differently between the treatment and control groups. This can bias the estimated treatment effect, as differences in outcomes may stem from these covariates rather than the treatment itself.

* **PSM**: This technique addresses imbalance by creating a matched control group. It pairs users in the treatment and control groups with similar characteristics based on a propensity score, **ensuring that confounding variables are balanced between the groups**. This step enhances the validity of causal comparisons.

* **Combining DiD and PSM**: By first applying **PSM** to reduce significant pre-treatment differences, we ensure the **DiD** method is applied to more balanced groups. DiD then estimates causal effects over time by leveraging the remaining differences in outcomes between the matched groups.

**In summary**
1. We start with PSM to correct for imbalances and address potential biases.
2. Then, we apply DiD to measure causal effects across time. Together, these methods provide a robust framework for causal inference, enhancing the accuracy and credibility of our results.

Steps:
1. Fit a logistic regression to estimate propensity scores.
2. Calculate *Inverse Probability of Treatment Weights (IPTW)* for treated and control groups, i.e., *weighting*.
3. Attach the propensity scores and weights to the dataset.
4. Inspect Treated Observations
    * For this, we'll first look at the weights for treated (treatment = 1) units and confirm if they are being calculated correctly.
5. Check Weight Distribution
    * We will plot a histogram to visualize the distribution of the IPTW weights, and optionally stabilize them.
6. Validate Balance
    * After calculating the IPTW weights, we will test if the covariates are balanced between treated and control units by comparing the mean of covariates before and after applying the weights.
7. Run the DiD Regression Model

### Sensitivity Analysis

Sensitivity analysis with placebo tests evaluates the robustness of your causal estimates by testing whether the observed effect is detectable in settings where no effect should theoretically exist. For example:

* Placebo Treatment: Assign the treatment randomly to individuals or groups and re-estimate the model. No significant effect should appear if the original estimates are valid.

* Placebo Outcome: Use an unrelated outcome variable instead of the actual one. Significant results here could indicate spurious correlations.

Placebo tests help detect hidden biases or violations of assumptions, strengthening causal claims.

### Spillover Effects

Spillover Effects occur when the treatment impacts not only treated individuals but also non-treated ones through interactions or shared environments. This violates the assumption of no interference between units (SUTVA), potentially biasing estimates.

**Addressing Spillovers**:
* Inclusion in the Model: Add terms capturing spillover exposure, like $
Y_{it} = \beta_2 (\text{spillover}_i \times \text{time}_t)$

**Define Spillover Groups**: 
* Identify units indirectly affected (e.g., geographic proximity or network connections).

**Simulations/Placebo Tests**: 
* Test robustness by simulating scenarios with varying spillover assumptions. 
* Accounting for spillovers ensures more accurate causal estimates.

## Respostas

**Pergunta 1**

**Parte 1**

**Analise a distribuição da atribuição do tratamento. Identifique potenciais variáveis de confusão. Verifique se há desbalanceamento nas covariáveis entre os grupos de tratamento e controle?**

1. **Distribuição da Atribuição do Tratamento**
   - 90.96% no grupo de controle, 9.04% no grupo de tratamento.
   - Implicação: Desbalanceamento que pode afetar o poder estatístico e criar problemas de confusão.

2. **Distribuição Marginal de `segmento`**
   - `frequente`: 40.28%, `novo`: 29.99%, `alto_valor`: 29.73%.
   - Implicação: Desequilíbrio na distribuição de `segmento` pode ser um fator de confusão.

3. **Distribuições Marginais para `historico_compras` e `frequencia_acesso`**
   - `historico_compras` é uma variável contínua com alta variabilidade, o que pode ser um fator de confusão.
   - `frequencia_acesso` também é um fator potencial de confusão devido ao seu impacto no tratamento e nos resultados.

4. **Distribuição Conjunta de `treatment` vs. `segmento`**
   - Grupo de tratamento concentrado em `alto_valor` (63.65%).
   - Implicação: Desbalanceamento de `segmento` entre os grupos, sugerindo que `segmento` é um fator de confusão.

5. **Resumo por Tratamento**
   - `historico_compras` e `frequencia_acesso` são significativamente mais altos no grupo de tratamento.
   - Implicação: Isso reforça preocupações sobre confusão devido a essas variáveis.

**Conclusão**: O desbalanceamento nos grupos de tratamento e controle em relação a `segmento`, `historico_compras`, e `frequencia_acesso` sugere a necessidade de técnicas robustas de inferência causal (como matching ou weighting) para estimar o efeito do novo algoritmo.

**Perguntas sobre Confounders**

1. **Por que a assimetria na distribuição marginal de `segmento` é uma potencial confounder?**
   - A assimetria por si só não é um problema, mas se o `segmento` estiver relacionado tanto à atribuição do tratamento quanto aos resultados (`engajamento` ou `vendas`), pode ser um fator de confusão.

2. **Por que `segmento` é uma confounder baseado na distribuição conjunta entre tratamento e controle?**
   - O desbalanceamento entre os grupos, com o grupo de tratamento sendo dominado por `alto_valor`, sugere que a atribuição do tratamento pode não ser aleatória e pode estar relacionada com os resultados.

3. **Por que `historico_compras` e `frequencia_acesso` são confounders?**
   - O grupo de tratamento tem valores mais altos para essas variáveis, e elas estão provavelmente relacionadas aos resultados (`engajamento` e `vendas`), além de influenciar a atribuição ao tratamento.

**Parte 2**

**Proponha um método adequado de inferência causal para estimar o efeito do novo algoritmo sobre o engajamento e as vendas. Justifique sua escolha**

**Escolha do Modelo**

**1. Separação de `Engajamento` e `Vendas`**
- `Engajamento` e `Vendas` têm mecanismos distintos e, por isso, devem ser tratados separadamente. 
- A regressão simples foi tentada inicialmente para ajustar variáveis de confusão e estimar o efeito do tratamento diretamente.

**2. Diferença em Diferenças (DiD)**

**Justificativa**: DiD controla o viés de seleção e as diferenças iniciais entre grupos, assumindo tendências paralelas.

**Modelo**:

<img src = "img\eq_1.jpg">

**3. Propensity Score Matching (PSM)**
- **Justificativa**: PSM ajuda a equilibrar as covariáveis entre os grupos de tratamento e controle, reduzindo viés de seleção.
- **Processo**:
  1. Estima-se o escore de propensão usando um modelo logit/probit.
  2. Emparelha-se as unidades tratadas com controles com escore similar.

**Conclusão**
- **Método Proposto**: Iniciamos com regressão simples, mas a abordagem principal será DiD, com PSM como complemento para balancear as covariáveis.
- **Justificativa**: Essas técnicas combinadas proporcionam estimativas mais precisas e robustas do efeito causal do novo algoritmo.


### Pergunta 2

**Implemente o método e estime o Average Treatment Effect (ATE) e o Average Treatment Effect on the Treated (ATT). Discuta as suposições necessárias para a validade do método escolhido e avalie se elas s são plausíveis neste contexto.**

**Comparação de Métodos**

| Modelo de Regressão         | **PSM + DiD** | **DiD** | **Regressão Multivariada Simples** |
|-----------------------------|----------------|---------|-----------------------------------|
| **Engajamento**             |                |         |                                   |
| ATE                         | 2.067          | 2.0345  | 0.9657                            |
| ATT                         | 2.067          | 2.0345  | 7.2872                            |
| **Vendas**                  |                |         |                                   |
| ATE                         | 9.425          | 9.5037  | 4.9698                            |
| ATT                         | 9.425          | 9.5037  | 83.3684                           |

**Métodos**

1. **PSM + DiD (Novo Método)**:
   - **Prós**: Combina Propensity Score Matching (PSM) para correção de viés de seleção e Difference-in-Differences (DiD) para estimativa dos efeitos do tratamento ao longo do tempo, proporcionando inferência causal mais robusta.
   - **Contras**: Requer processamento adicional para emparelhamento e pode enfrentar desafios com variáveis não observadas.

2. **DiD (Direto)**:
   - **Prós**: Simples e eficaz para estimar efeitos causais ao longo do tempo, ideal para dados em painel com observações pré e pós-tratamento.
   - **Contras**: Assume tendências paralelas entre os grupos de tratamento e controle, o que pode não ser válido em todos os casos.

3. **Regressão Multivariada Simples**:
   - **Prós**: Fácil de implementar e interpretar.
   - **Contras**: Potencial viés de variável omitida se fatores relevantes não forem incluídos, tornando difícil inferir efeitos causais com precisão.

**Validade dos Métodos**

**PSM + DiD**:
- **Suposições**: 
  1. Tendências paralelas para DiD (os grupos de tratamento e controle evoluem de maneira semelhante ao longo do tempo sem o tratamento).
  2. Ajuste adequado dos escores de propensão para equilibrar características observadas entre os grupos.
- **Validade**: A suposição de tendências paralelas pode ser plausível se as características iniciais entre os grupos de tratamento e controle forem similares. No entanto, pode haver desafios caso existam variáveis não observadas que influenciem a escolha de tratamento.

**DiD**:
- **Suposições**:
  1. Tendências paralelas.
- **Validade**: Esta suposição é crítica, e se não for válida, os resultados podem ser enviesados. A plausibilidade depende da homogeneidade das tendências temporais nos grupos de tratamento e controle.

**Regressão Multivariada Simples**:
- **Suposições**:
  1. Nenhum viés de variável omitida.
  2. Linearidade na relação entre variáveis.
- **Validade**: A suposição de ausência de viés de variável omitida pode não ser plausível, especialmente em um contexto com múltiplos fatores influenciando os resultados. A regressão simples pode ser afetada por esses vieses.

### Resultados
- **Engajamento** e **Vendas** apresentaram efeitos significativos do tratamento com **PSM + DiD**, indicando impacto robusto pós-intervenção.
- O método **DiD** isolado apresentou resultados similares, mas o **PSM + DiD** forneceu estimativas mais precisas ao corrigir o viés de seleção.
- A **regressão multivariada** superestimou os efeitos do tratamento, especialmente em vendas, sugerindo viés de variáveis omitidas.

**Conclusão**
- O **PSM + DiD** se mostrou o método mais robusto para estimar os efeitos do tratamento, superando a regressão multivariada e o DiD isolado em termos de precisão e correção de viés.

### Pergunta 3

**Realize testes de sensibilidade para avaliar a robustez de suas estimativas. Considere a possibilidade de variáveis não observadas estarem influenciando seus resultados. Como isso afetaria sua análise?**

**Análise do Teste de Sensibilidade**

**Para `vendas`**:
  - Coeficiente para `random_treatment`: `-0.1178` (valor-p: `0.403`).
  - Coeficiente para `Random_Treated_Post`: `-0.1430` (valor-p: `0.483`).

Não foram encontrados efeitos placebo estatisticamente significativos, indicando que não houve efeitos espúrios de tratamento devido à atribuição aleatória.

**Para `engajamento`**:
  - Coeficiente para `random_treatment`: `-0.0084` (valor-p: `0.569`).
  - Coeficiente para `Random_Treated_Post`: `-0.0022` (valor-p: `0.919`).

De maneira similar, não foram detectados efeitos placebo significativos, o que reforça a validade dos efeitos do tratamento real em sua análise principal.

**Conclusão**

- Os resultados dos testes de sensibilidade afirmam que os achados do método **PSM + DiD** são robustos.
- Isso sugere que variáveis não observadas têm pouca influência sobre os resultados, aumentando a confiabilidade das conclusões sobre os efeitos do tratamento.

## Pergunta 4

**Suponha que haja efeitos de spillover entre os usuários. Discuta como isso afeta sua análise e proponha métodos para abordá-lo. Ajuste o modelo:**

<img src = "img\eq_2.jpg">

**Impacto dos Efeitos de Spillover na Análise**

Se houver efeitos de spillover, o tratamento de um usuário pode influenciar outros, violando a **SUTVA**. Isso gera viés nas estimativas de ATE e ATT, já que usuários não tratados podem experimentar efeitos parciais do tratamento.

**Método Proposto para Abordar Spillovers**

**Modelo ajustado para incluir spillovers**:

<img src = "img\eq_3.jpg">

1. **Definir a Variável de Spillover**: Quantificar a exposição aos spillovers, como por conexões de rede ou proximidade geográfica.
2. **Integrar Spillovers no PSM + DiD**:
   - **PSM**: Combinar unidades não tratadas com exposições semelhantes a spillover.
   - **DiD**: Adicionar o termo $\beta_2 (\text{spillover}_i \times \text{time}_t)$ para isolar os efeitos indiretos.

**Interpretação dos Resultados**

1. **Efeito Positivo de Spillover ($\beta_2 > 0$)**: Spillovers aumentam os resultados dos usuários não tratados, subestimando o efeito direto do tratamento.
2. **Efeito Negativo de Spillover ($\beta_2 < 0$)**: Spillovers reduzem os resultados do grupo controle, superestimando os efeitos do tratamento.

**Conclusão**
Este ajuste fortalece a análise, oferecendo uma visão mais clara dos efeitos diretos e indiretos do tratamento.

## References
* Introductory Econometrics (Jeffrey Wooldridge)
* Mostly Harmless Econometrics (Joshua Angrist & Pischke)
* Probability Theory and Statistical Inference: Econometric Modeling with Observational Data (Aris Spanos)
* *"Using propensity scores in difference-in-differences models to estimate the effects of a policy change"* (https://pmc.ncbi.nlm.nih.gov/articles/PMC4267761/#FD2)