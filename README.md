# Avaliação Quantitativa de Resumos: **OpenAI GPT‑4(o)-mini** vs. **Llama‑3 70B**

> **Fonte única da verdade:** todo o procedimento descrito aqui existe, linha a linha, no notebook **`summaries_evaluation.ipynb`**.  Este README é apenas um guia textual. Siga o notebook para obter exatamente os mesmos passos, fórmulas e resultados.

---

## 🎯 Objetivo

Comparar, de forma transparente e reproduzível, a qualidade de resumos gerados por dois LLMs distintos.  A avaliação combina **julgamento humano** e **métrica automática (RAGAS)**, controlando a confiabilidade dos avaliadores por meio do **ICC(3,k)** e corrigindo o peso dado à métrica automática em função da **correlação de Spearman (ρ)**.

---

## 📂 Dados de Entrada

`data/summaries_evaluation.csv` — cada linha contém:

| Coluna                           | Descrição                                      |
| -------------------------------- | ---------------------------------------------- |
| `text`                           | Texto‑fonte (input a ser resumido)             |
| `summary_openai` / `summary_llama` | Resumo gerado por cada modelo                |
| `h1_*_rating` … `h3_*_rating`    | Notas de três avaliadores humanos (0-1)        |
| `ragas_*_rating`                 | Score do RAGAS para o respectivo resumo       |
| `class_label`                    | <small>(removido na análise)</small>           |

O notebook elimina linhas/colunas com valores nulos ou irrelevantes e cria dois subconjuntos bem definidos:

* **`df_openai`** — texto + colunas `OPENAI_COLS`
* **`df_llama`**  — texto + colunas `LLAMA_COLS`

---

## 🔬 Pipeline Analítico (passo a passo)

1. **Carregamento & limpeza** (`pandas`).
2. **Confiabilidade dos avaliadores**
   * Calcula‐se o **ICC(3,k)** para cada modelo.
3. **Estimativa da nota humana agregada**
   * **OpenAI:** mediana → `H_rob` (robusto a outliers).
   * **LLaMA:** média ponderada → `H_bar`, onde o peso de cada avaliador é `wᵢ = max(ρᵢ, 0)` (ρᵢ = correlação de Spearman do avaliador com o grupo).
4. **Correlação com métrica automática**
   * Spearman ρ entre `H_*` e `ragas_*_rating`.
5. **Peso da métrica automática**
   ```python
   w_A = min(max(ρ, 0), 0.30)  # teto recomendado para dados ruidosos
   ```
6. **Score composto por sumário**
   ```python
   Score = (H_* + w_A * ragas_*_rating) / (1 + w_A)
   ```
7. **Exportação de resultados**
   * `openai_results.csv` & `llama_results.csv` (um Score por linha).
8. **Comparação direta**
   * Concatenação:
     ```python
     df_scores = pd.concat([
         df_llama['Score'].rename('score_llama'),
         df_openai['Score'].rename('score_openai')
     ], axis=1)
     df_scores['veredito'] = np.where(
         df_scores['score_llama'] >= df_scores['score_openai'], 'llama', 'openai'
     )
     ```
9. **Visualizações** — o notebook cria tabelas e gráficos simples (CPU‑only).

---

## 🗃️ Estrutura do Repositório

```text
.
├── data/
│   └── summaries_global_evaluation.csv
├── outputs/                # Gerado após executar o notebook
│   ├── openai_results.csv
│   └── llama_results.csv
├── summaries_evaluation.ipynb
├── requirements.txt
└── README.md
```

---

## ⚙️ Requisitos

* **Python ≥ 3.10**
* Dependências principais (ver `requirements.txt`): `pandas`, `numpy`, `pingouin`, `scipy`, `jupyter`.

### Passo a passo para reproduzir
```bash
# 1) Clone o repositório
git clone https://github.com/vtrnduda/summarization-metrics-analysis.git
cd summarization-metrics-analysis

# 2) Ambiente virtual (opcional)
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate  # Windows

# 3) Instale as dependências
pip install -r requirements.txt

# 4) Execute o notebook
jupyter lab  # (ou jupyter notebook)
```
Execute todas as células. Os CSVs de saída vão aparecer em `outputs/`.

---

## 📑 Saídas Esperadas

| Arquivo                        | Conteúdo                                        |
| ------------------------------ | ---------------------------------------------- |
| `outputs/openai_results.csv`   | Colunas `text`, `Score`, métricas intermediárias |
| `outputs/llama_results.csv`    | Idem para o LLaMA                               |
| `outputs/df_scores.csv`        | `score_openai`, `score_llama`, `veredito`       |

A última célula do notebook exibe `df_scores` diretamente em tela.

---

## 🤝 Contribuindo

1. *Fork* ➡️  `git checkout -b minha‑feature` ➡️  commit ➡️  push ➡️  Pull Request.
2. Mantenha a mesma nomenclatura de colunas para que o notebook funcione sem ajustes.

