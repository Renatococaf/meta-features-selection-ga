# Meta-Feature Selection with Genetic Algorithms

**Projeto Final — Disciplina de Meta-aprendizagem**  
Universidade Federal de Alagoas (UFAL) — Instituto de Computação  
Professor: Lucas Amorim  
Aluno: Renato Coca Terrazas Freire

---

## Descrição

Este projeto implementa um sistema de **seleção de meta-features via Algoritmo Genético (GA)** aplicado a meta-aprendizagem. Partindo do Exemplo 3 da disciplina (22 datasets, 56 meta-features), o trabalho expande o meta-dataset e aplica GA para encontrar o subconjunto ótimo de meta-features que melhora a predição de qual algoritmo de classificação tem melhor desempenho em um dado dataset.

---

## Resultados

| | Exemplo 3 (referência) | Este projeto (baseline) | Após GA |
|---|---|---|---|
| Datasets | 22 | 35 | 35 |
| Meta-features | 56 | 224 | **114** |
| RMSE (LR, RF) | 0.09078 | 0.08768 | **0.07902** |
| Spearman (LR, RF) | 0.30 | 0.76 | **0.78** |

**Redução de features:** 224 → 114 (**−49.1%**)  
**Melhora de RMSE:** 0.08768 → 0.07902 (**−9.9%**)

---

## Estrutura do Repositório

```
meta-features-selection-ga/
│
├── meta_learning_ga.ipynb          # Notebook principal com toda a implementação
│
├── data/
│   └── meta_dataset.csv            # Meta-dataset expandido (35 datasets × 224 meta-features)
│
├── results/
│   ├── baseline_results.csv        # RMSE e Spearman do meta-modelo baseline (224 features)
│   ├── optimized_results.csv       # RMSE e Spearman do meta-modelo otimizado (114 features)
│   ├── comparison_results.csv      # Tabela comparativa baseline vs GA
│   └── ga_results.json             # Cromossomo final, histórico de fitness e convergência do GA
│
└── figures/
    ├── correlation_heatmap.png         # Correlação entre as 224 meta-features
    ├── performance_distributions.png   # Distribuição de acurácia dos 6 algoritmos base
    ├── feature_importance_baseline.png # Top 25 meta-features por importância (Random Forest)
    ├── ga_convergence.png              # Curva de convergência do GA (RMSE e nº de features)
    ├── comparison_charts.png           # RMSE e Spearman: baseline vs após GA
    ├── wilcoxon_boxplot.png            # Boxplot do teste de Wilcoxon (10-fold CV)
    └── selected_vs_importance.png      # Sobreposição: GA vs importância RF
```

---

## Metodologia

### 1. Construção do Meta-Dataset
- **Parte A:** 20 datasets do Exemplo 3 do professor (via OpenML)
- **Parte B:** 15 datasets extras da benchmark suite OpenML-CC18
- **Meta-features:** 5 grupos pymfe (`general`, `statistical`, `info-theory`, `model-based`, `complexity`) × 4 medidas de resumo (`mean`, `sd`, `min`, `max`) → **224 meta-features**
- **Algoritmos base avaliados:** DecisionTree, SVM, KNN, LogisticRegression, Perceptron, MLP

### 2. Algoritmo Genético
| Parâmetro | Valor |
|---|---|
| Tamanho da população | 30 |
| Número de gerações | 50 |
| Taxa de crossover | 0.8 |
| Taxa de mutação | 0.05 |
| Elitismo | Top-2 preservados |
| Fitness | RMSE (5-fold CV, RandomForest) |
| Seleção | Torneio (k=3) |
| Crossover | Single-point |
| Cache de fitness | Sim |

### 3. Avaliação
- Meta-modelo: Random Forest Regressor e KNN Regressor
- Métricas: RMSE e Spearman ρ (10-fold CV)
- Teste estatístico: Wilcoxon signed-rank

---

## Como Executar

### Requisitos
```bash
pip install openml pymfe scikit-learn numpy pandas matplotlib seaborn scipy
```

### Execução
1. Abra `meta_learning_ga.ipynb` no Jupyter ou VS Code
2. Execute todas as células em sequência (**Run All**)
3. A coleta de dados leva ~10–20 min; o GA leva ~5–10 min (dependendo do hardware)
4. Os resultados são salvos automaticamente em CSV e JSON

> **Nota:** Na primeira execução, o cache `meta_dataset_cache.pkl` é criado automaticamente. Nas execuções seguintes, os dados são carregados do cache sem re-processar.

---

## Principais Conclusões

1. **A expansão de meta-features já melhora o meta-modelo:** Spearman ρ subiu de 0.30 (Exemplo 3) para 0.76 com as 224 features expandidas, antes mesmo do GA.

2. **O GA reduz eficientemente a dimensionalidade:** Selecionou 114 de 224 features (−49.1%) mantendo ou melhorando os resultados para o algoritmo alvo.

3. **Especialização da seleção:** O subconjunto selecionado melhora a predição de LogisticRegression, SVM e MLP, mas piora para DecisionTree e Perceptron — mostrando que a seleção ótima de meta-features depende do algoritmo alvo.

4. **GA vs Importância RF:** O GA descartou a 3ª, 4ª e 5ª features mais importantes pelo Random Forest (l3.max, f1v.max, l1.max) por serem redundantes com as duas primeiras — demonstrando seleção mais sofisticada que um simples ranking.

5. **Significância estatística:** A melhora não atingiu significância estatística (Wilcoxon p=0.232), esperado dado o tamanho do meta-dataset (35 datasets). Com mais datasets, a diferença provavelmente seria confirmada.

---

## Referências

- Rivolli, A., et al. (2022). *Meta-features for meta-learning*. Knowledge-Based Systems.
- Vanschoren, J. (2018). *Meta-learning: A survey*. arXiv:1810.03548.
- OpenML-CC18 Benchmark Suite: https://www.openml.org/s/99
- pymfe: https://github.com/ealcobaca/pymfe
