### Declaramos que foi utilizado inteligência artificial como apoio técnico para dúvidas pontuais e codificação

# Predição de Casos SRAG — Montes Claros / MG

> Modelo SARIMAX para previsão semanal de Síndrome Respiratória Aguda Grave (SRAG) não-COVID usando variáveis meteorológicas do INMET.

---

## Problema

Gestores da Secretaria Municipal de Saúde de Montes Claros precisam antecipar picos de SRAG para dimensionar leitos e equipes. Os dados climáticos disponíveis publicamente (INMET) carregam um sinal preditivo com **1-2 semanas de antecedência** — janela suficiente para planejamento operacional.

---

## Estrutura do repositório

```
srag_predicao/
├── srag_predicao_moc.ipynb   # Notebook principal (análise + modelo)
├── requirements.txt          # Dependências Python
├── outputs/                  # Figuras salvas automaticamente
└── README.md
```

Os dados brutos devem estar em `../bb/data/` (caminho relativo ao repositório pai):
```
bb/data/
├── srag_moc/     # influd_19.csv … influd_25.csv  (SIVEP-SRAG)
└── inmet/        # inmet_19.csv  … inmet_25.csv   (INMET horário)
```

---

## Fontes de dados

| Fonte | Dados | Acesso |
|---|---|---|
| **SIVEP-SRAG** | Internamentos por SRAG, classificação final (COVID/não-COVID) | [OpenDatasus](https://dadosabertos.saude.gov.br/dataset/srag-2019-a-2026) — gratuito |
| **INMET** | Temperatura mínima, umidade mínima, ponto de orvalho, amplitude térmica, precipitação — horários | [INMET BDMEP](https://tempo.inmet.gov.br/TabelaEstacoes/A506) — gratuito |

Município: **Montes Claros – MG** (IBGE: 314330) | Estação INMET: **A506**

---

## Metodologia

### 1. Separação do sinal epidemiológico
- Casos SRAG COVID-19 seguem dinâmica epidêmica (não climática) e são **excluídos** do target
- **Estratégia A**: semanas com >60 % de COVID excluídas
- **Estratégia C**: janela a partir de 2023 (dados limpos pós-pandemia)
- Resultado: 158 semanas de série limpa

### 2. Variáveis exógenas (fonte: INMET)

| Variável | Lag | 
|---|---|
| `temp_min` | 3 semanas
| `umid_min` | 2 semanas
| `orvalho_med` | 2 semanas
| `amp_term` | 2 semanas
| `precip` | 3 semanas
| `sin_sem`, `cos_sem` | 0 semanas

### 3. Modelo: SARIMAX(1,0,3)
- Walk-forward cross-validation (treinamento mínimo: 20 semanas)
- 138 semanas avaliadas em sequência (1-step-ahead)
- Grid search em (p,q) ∈ {0,1,2,3}² por AIC

---

## Resultados

| Modelo | MAE (casos/sem) | MAPE | R² |
|---|---|---|---|
| **SARIMAX(1,0,3)** | **6.69** | 42.3 % | 0.57 |
| XGBoost (tunado) | 8.89 | 46.2 % | 0.31 |
| Ensemble (média) | 7.18 | 41.4 % | 0.58 |

> Série com média = 19.2 casos/semana e desvio-padrão = 14.0.  
> MAE de 6.7 representa ~35 % de erro relativo — aceitável para planejamento operacional.

---

## Como executar

```bash
# 1. Criar ambiente (opcional)
python -m venv .venv && source .venv/bin/activate

# 2. Instalar dependências
pip install -r requirements.txt

# 3. Abrir o notebook
jupyter lab srag_predicao_moc.ipynb
```

> **Importante:** execute a partir do diretório `srag_predicao/` OU ajuste a variável `DATA_DIR` na célula de configuração.

---

## Diferencial da solução

1. **Antecipação operacional**: previsão 2–3 semanas à frente com dados já disponíveis
2. **Separação COVID/não-COVID**: isola o sinal climático, eliminando a contaminação epidêmica
3. **Escala municipal**: granularidade que literatura acadêmica raramente alcança
4. **Custo zero**: SIVEP-SRAG + INMET são públicos e atualizados semanalmente
5. **Interpretabilidade**: SHAP mostra _por que_ a previsão é alta (umidade baixa + frio = alerta)

---

## Dependências principais

- Python ≥ 3.10
- pandas, numpy, matplotlib, seaborn, scipy
- statsmodels ≥ 0.14
- xgboost ≥ 2.0
- shap ≥ 0.44
- scikit-learn ≥ 1.3
