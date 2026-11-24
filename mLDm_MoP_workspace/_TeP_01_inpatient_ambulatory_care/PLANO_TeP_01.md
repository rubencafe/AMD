# TeP_01 — Inpatient and Ambulatory Care Prediction | Plano de trabalho

> Ficheiro de contexto para continuar o trabalho noutra sessão (com o Claude ou não).
> Criado em 2026-11-24. Este ficheiro está na raiz da pasta da atividade e **não entra no ZIP de submissão**
> (só entram os itens `[TO_ZIP]` de `01_input/_mLDm_xx_submission_requirements_rev01.txt`).

---

## 0. Como retomar (instruções para uma nova sessão)

1. Ler este ficheiro do início ao fim.
2. Ler o enunciado: `_handout.pdf` (10 páginas, secções 1–6, requisitos R01–R10).
3. Ver o estado das tarefas na **secção 6** (checkboxes) e as decisões na **secção 7**.
4. Antes de criar código, rever o estilo dos notebooks de referência (secção 2.3), que é o que se pretende replicar.
5. Atualizar este ficheiro sempre que uma tarefa fica concluída ou uma decisão é tomada.

**Estado atual (2026-11-24):** só foi feita a análise e o plano. **Ainda não foi criado nem alterado nenhum
ficheiro de código.** O plano foi reorganizado pelos R01–R10 (secção 6, decisões em 7.2). As decisões D1–D7
(secção 7.1) ainda não foram confirmadas pelo utilizador.

---

## 1. Objetivo (resumo do enunciado)

Dados reais do SNS ("Morbilidade e Mortalidade Hospitalar"), mensais, de 2021 até 2026.

- **Unidade de observação (data-grain mensal):** `(period, institution, diagnosis, age_group, sex)`.
- Atividade descrita por: internamentos (I), dias de internamento, episódios de ambulatório (A), óbitos.
- **Três problemas de previsão:**

| Problema | Tipo | Alvo | Definição |
|---|---|---|---|
| ocorrência de óbito | classificação binária | `has_death` | 0 = sem óbitos, 1 = um ou mais |
| nível de internamento | classificação multiclasse | `inpatient_level` | discretizar `I / (I + A)` em low/medium/high com os intervalos da tabela `inpatient_level` |
| dias de internamento | regressão | `num_inpatient_day` | total de dias de internamento no mês |

- **Regra de prediction-time:** a atividade do mês a prever (I, A, dias, óbitos) só serve para construir os alvos,
  **nunca** como feature. As features vêm dos 5 atributos da unidade de observação (e derivados).
- **Dataset C** é exclusivo para deployment: nunca usar em treino, avaliação ou seleção.

### 1.1 Requisitos do relatório (por esta ordem no relatório)

| Req | Secção | Conteúdo |
|---|---|---|
| R01 | 1 | um risco concreto de *information leakage* em prediction-time e como foi evitado |
| R02 | 2 | diagrama ER (entidades, atributos, identificadores, relações, cardinalidade, opcionalidade) |
| R03 | 2 | quantificar registos originais, unidades distintas, repetições e conflitos em A e B; distribuição dos alvos |
| R04 | 2 | impacto do critério *worst-case* num alvo vs uma regra determinística alternativa |
| R05 | 3 | comparar e justificar as representações de features nos 3 problemas |
| R06 | 3 | análise do 1R nos 2 problemas de classificação (matrizes de contingência, feature escolhida, erros, vs baseline maioritária) |
| R07 | 4 | caracterizar e justificar as partições temporais treino/teste |
| R08 | 4 | comparar modelos com as baselines; diferenças entre métricas |
| R09 | 4 | variação do desempenho entre períodos de teste; justificar o modelo escolhido por problema |
| R10 | 5 | execução standalone com modelos persistidos: consistência input/output, sem retreino, valores não vistos |

### 1.2 Regras obrigatórias (checklist rápida)

- [ ] Ficheiros fornecidos (`_mLDm_*`) ficam **inalterados**; trabalha-se em cópias sem o prefixo `_mLDm_`.
- [ ] Consolidação: exatamente 1 registo original por unidade, regra *worst-case*: ordenar por
      `num_death DESC, num_inpatient_day DESC, num_inpatient_admission DESC, num_ambulatory_episode DESC, id ASC`.
      A mesma regra para A e B.
- [ ] Divergências face às tabelas de referência corrigidas **antes** da integração, de forma reproduzível,
      e documentadas em `03_output/sns_integration_check.csv` (`dataset, attribute, original_value, reference_value`).
      Nenhum registo descartado silenciosamente.
- [ ] Analytical-dataset: 5 atributos + `inpatient_level`, `num_inpatient_day`, `has_death`; validar unicidade,
      derivação dos alvos e ausência de perdas.
- [ ] `OneRClassifier` em Python, **obrigatoriamente com DuckDB `PIVOT`** (linhas = valores da feature,
      colunas = classes, células = frequências); desempate determinístico; valor não visto → classe maioritária global.
- [ ] Modelos obrigatórios nos 3 problemas: Decision Tree, kNN, Random Forest (+ 1R nos 2 de classificação).
- [ ] Baselines: classe maioritária (classificação), média do treino (regressão).
- [ ] Avaliação temporal com **janela expansiva**; todas as transformações dependentes dos dados ajustadas só no treino de cada fold.
- [ ] Métricas: accuracy, balanced accuracy, macro-F1 (classificação); MAE, RMSE (regressão).
- [ ] Modelos finais retreinados com **todos** os dados A+B, guardados num único `03_output/sns_deployed_models.joblib`.
- [ ] `02_deployed_predictors.py` standalone; output `03_output/_mLDm_00_sns_data_C_prediction.csv` com
      `period, institution, diagnosis, age_group, sex, inpatient_level, num_inpatient_day, has_death`
      (por esta ordem), mesmo nº e ordem de linhas do input.
- [ ] Relatório `.pdf` ≤ 10 páginas + capa (AMD \ MLDM, nº do grupo, nome e nº de cada elemento);
      identificar os ficheiros de código relevantes; ênfase em resultados e justificações.
- [ ] Tudo em `03_output` reproduzível a partir de `01_input` + `02_code`, sem passos manuais não documentados.

---

## 2. Mapa de ficheiros e pastas

Raiz do repositório git: `C:\me\AMD` (branch `main`).
Workspace: `C:\me\AMD\mLDm_MoP_workspace`.
Atividade: `C:\me\AMD\mLDm_MoP_workspace\_TeP_01_inpatient_ambulatory_care` (abaixo designada `TeP/`).

### 2.1 Materiais fornecidos (NÃO alterar)

| Caminho | Conteúdo |
|---|---|
| `TeP/_handout.pdf` | enunciado |
| `TeP/01_input/_mLDm_00_sns_data_A.csv` | 326 692 registos, 2021-01 → 2024-12; separador `;`, cabeçalhos em PT |
| `TeP/01_input/_mLDm_00_sns_data_B.csv` | 154 014 registos, 2025-01 → 2026-06; mesmo formato de A |
| `TeP/01_input/_mLDm_00_sns_data_C.csv` | 11 580 obs., 2026-07 → 2026-09; separador `,`; colunas `period,institution,diagnosis,age_group,sex` (institution = nome, diagnosis = código) |
| `TeP/01_input/_mLDm_01_sns_schema.sql` | esquema PostgreSQL: `region`, `institution`, `diagnosis`, `source_record`, `inpatient_level`, função `get_canonical_name()` e índices únicos canónicos |
| `TeP/01_input/_mLDm_02_sns_data_db.sql` | referência: 5 regiões, 43 instituições, 21 capítulos de diagnóstico (códigos 1–19, 21, 22), níveis low [0,0.2[ / medium [0.2,0.8[ / high [0.8,1] |
| `TeP/01_input/_mLDm_xx_submission_requirements_rev01.txt` | `[TO_ADAPT]`, `[TO_EXIST]`, `[TO_ZIP]` |
| `TeP/02_code/_mLDm_00_sns_db_boot.py` | cria a BD `mldm_tep` no container `mldm_postgres` (usa `RUNTIME = "podman"`) |
| `TeP/02_code/_mLDm_00_sns_selected_solutions.ipynb` | notebook com soluções parciais e TODOs (só dataset A) |
| `TeP/02_code/_mLDm_xx_build_submission_zip_rev01.py` | gera o ZIP: `python _mLDm_xx_build_submission_zip_rev01.py NN` (a partir de `02_code`) |

Colunas dos CSV A/B: `Período;Região;Instituição;Código Capítulo Diagnóstico Principal;Descrição Capítulo Diagnóstico Principal;Faixa Etária;Sexo;Internamentos;Dias de Internamento;Ambulatório;Óbitos`.

### 2.2 Entregáveis a criar

| Caminho | No ZIP? | Notas |
|---|---|---|
| `TeP/02_code/00_sns_selected_solutions.ipynb` | sim | cópia do notebook fornecido com todos os TODO resolvidos |
| `TeP/02_code/01_sns_build_modelling_datasets.ipynb` | sim | integração A+B, consolidação, analytical-dataset, modelling-datasets |
| `TeP/02_code/02_deployed_predictors.py` | sim | aplicação standalone de previsão |
| `TeP/02_code/00_sns_db_boot.py` | sim | (proposto) cópia do boot com `RUNTIME = "docker"` |
| `TeP/02_code/sns_lib.py` | sim | (proposto) módulo partilhado: `OneRClassifier`, derivação de features, folds temporais, caminhos |
| `TeP/02_code/03_sns_temporal_evaluation.ipynb` | sim | (proposto) avaliação temporal e seleção |
| `TeP/02_code/04_sns_final_models.ipynb` | sim | (proposto) retreino final e persistência |
| `TeP/03_output/sns_analytical_dataset.csv` | não (tem de existir) | `[TO_EXIST]` |
| `TeP/03_output/sns_integration_check.csv` | sim | correções às fontes |
| `TeP/03_output/sns_deployed_models.joblib` | sim | 3 modelos finais + pré-processamento |
| `TeP/03_output/_mLDm_00_sns_data_C_prediction.csv` | sim | previsões para C |
| `TeP/03_output/<modelling datasets e resultados>` | não | nomes livres (ex.: `sns_modelling_has_death.csv`, `sns_eval_results_*.csv`) |
| `TeP/04_report/report.pdf` | sim | relatório final |
| `TeP/04_report/README.txt` | sim | instruções de reprodução |
| `TeP/04_report/submission_check.txt` | sim | gerado automaticamente pelo script de submissão |

**Atenção:** a pasta `02_code/` inteira entra no ZIP (exceto ficheiros `_mLDm_*`, `__pycache__`, `.ipynb_checkpoints`).
Não deixar ficheiros temporários/rascunhos em `02_code/`. Eventuais `.ows` do Orange vão para `02_code/` e têm de ser referidos no relatório.

### 2.3 Trabalhos de referência (estilo e métodos a replicar)

| Caminho (relativo ao workspace) | Para quê |
|---|---|
| `MoP_03_from_data_source_to_dataset_2of2/04_report/03_penguins_analysis.ipynb` | análise com DuckDB→PostgreSQL, CUBE, PIVOT, janelas `OVER`; perguntas-guia GQ → evidência E/A → conclusão |
| `MoP_03_from_data_source_to_dataset_2of2/04_report/04_penguins_dataset.ipynb` | construção e validação do analytical-dataset (CQ1–CQ3, unidade de observação, data-grain, `COPY ... TO csv`) |
| `MoP_04_from_dataset_to_model/02_code/01_penguins_feature_representation.ipynb` | `ROOT/INPUT/OUTPUT`, `ColumnTransformer` + `OneHotEncoder(categories=...)`, `ID` técnico, export `.csv`/`.tab` (Orange), `joblib` |
| `MoP_04_from_dataset_to_model/02_code/02_penguins_modelling.ipynb` | partições comuns por `ID`, `train_evaluate(...)`, comparação de classificadores e scalers (versão com TODOs por completar) |
| `MoP_04_from_dataset_to_model/02_code/03_penguins_selection_deployment.ipynb` | padrão de deployment: escolher configuração, treinar, guardar, recarregar, validar input, transformar, prever, checkpoint |
| `MoP_02_from_data_source_to_dataset_1of2/04_report/00_penguins_db_boot.py` | exemplo de cópia do boot script com `RUNTIME = "docker"` |
| `MoP_02_from_data_source_to_dataset_1of2/04_report/penguins_ER_model.pdf` | exemplo de diagrama ER entregue antes |
| `_mLDm_xx_build_submission_zip.py` (raiz do workspace) e `submitted_G12_MoP_01_install_and_test.zip` | o grupo é **G12** |

Teoria (raiz do repo `C:\me\AMD`): `MLDM_04_data_source_and_analytical_dataset_v01.pdf`,
`MLDM_05_from_data_type_to_feature_representation_v01.pdf`, `MLDM_06_learning_single_and_ensemble_models_v01.pdf` (1R / single-feature classifiers),
`MLDM_07_evaluationOfLearning_v03.pdf`, `MLDM_08_algorithmsInductionDecisionTree_simple_version_v01.pdf`, `MLDM_12_instanceBasedLearning.pdf` (kNN).

### 2.4 Ambiente

- Python: venv em `C:\me\AMD\mLDm_MoP_workspace\__venv` (Python 3.13). Executável: `__venv\Scripts\python.exe`.
  Pacotes já instalados: duckdb 1.5.4, pandas 3.0.3, numpy 2.5.1, scikit-learn 1.9.0, joblib 1.6.0, matplotlib 3.11.0,
  psycopg 3.3.4, jupyterlab 4.6.1, ipykernel.
- PostgreSQL: `C:\me\AMD\mLDm_MoP_workspace\__container\postgres\compose.yaml` (container `mldm_postgres`, postgres:16,
  porta 5432, pgAdmin em 5050). Docker via **Rancher Desktop** (`docker` 29.9.3-rd); estava **parado** em 2026-11-24.
  `podman` não está instalado.
- BD da TeP: `mldm_tep` (criada pelo boot script). Ligação DuckDB (credenciais locais de desenvolvimento, já presentes no notebook fornecido):
  `ATTACH 'host=localhost port=5432 dbname=mldm_tep user=mldm password=mldm' AS pg (TYPE postgres); USE pg.public;`
- Escrita no PostgreSQL a partir do DuckDB: `CALL postgres_execute('pg', $$ ... $$);` (ver notebook fornecido, células 15–19).

---

## 3. Factos sobre os dados (verificados em 2026-11-24, só leitura)

| | A | B | C |
|---|---|---|---|
| períodos | 2021-01 → 2024-12 (48 meses) | 2025-01 → 2026-06 (18 meses) | 2026-07 → 2026-09 |
| registos | 326 692 | 154 014 | 11 580 |
| unidades de observação distintas | 317 239 | 111 033 | – |
| unidades repetidas | 7 684 (2,4%) | 27 723 (25%) | – |
| unidades com conflito de valores | 7 684 (todas as repetidas) | 27 723 (todas as repetidas) | – |

- Não há sobreposição de períodos entre A e B.
- **Divergência com a referência em B:** instituição `Hosp. de Cascais Dr. José de Almeida` (19 registos, Região de Saúde LVT)
  → valor de referência `Hospital de Cascais Dr. José de Almeida`. É uma abreviatura, `get_canonical_name()` **não** a resolve:
  tem de ser corrigida explicitamente e registada em `sns_integration_check.csv` (dataset B).
- Em A não há divergências (regiões, instituição×região e diagnóstico código×descrição batem certo). Diagnósticos de B também batem certo.
- C: todas as instituições e diagnósticos existem na referência; inclui os 7 grupos etários e os 3 sexos.
- `sex = 'I'` é muito raro (27 registos em A, 6 em B) → atenção a valores raros/não vistos nos folds.
- Valores esperados do notebook fornecido (dataset A): 317 239 unidades, 309 555 únicas, 7 684 repetidas, 9 453 registos em excesso;
  `inpatient_level` em A: high 173 630, low 60 900, medium 82 709 (restantes NULL por I+A = 0);
  `num_inpatient_day` em A: min 0, max 11 078, média 83,45, mediana 15.
- `inpatient_level` fica NULL quando I + A = 0 (denominador zero). Manter no analytical-dataset; excluir só do modelling-dataset multiclasse.
- Intervalos: `min_ratio <= r < max_ratio`, exceto r = 1 que pertence a `high` (ver JOIN no notebook fornecido, célula 19).

---

## 4. Convenções de trabalho (herdadas de MoP_03/MoP_04)

- DuckDB em memória ligado ao PostgreSQL com `ATTACH`; lógica de dados em SQL com views (TEMP no DuckDB ou views no PostgreSQL).
- Cada célula começa com um bloco de comentário:
  ```
  # ------------------------------------------------------------
  # título curto da célula
  # ------------------------------------------------------------
  ```
- Perguntas-guia (GQ/CQ) antes da recolha de evidência; células de interpretação com `E1/A1` e "conclusion for GQx".
- `# expected output:` + `assert` com mensagens `"PTS | ..."` em cada validação.
- Caminhos: `ROOT = Path.cwd(); if ROOT.name == "02_code": ROOT = ROOT.parent`, depois `INPUT`/`OUTPUT`.
- `OneHotEncoder` com categorias pré-definidas (as da referência), `ID` técnico que não é feature, mesmas partições para todos os modelos.
- **Diferença face ao MoP_04:** aqui OHE, scalers e qualquer transformação dependente dos dados têm de ser ajustados **só no treino
  de cada fold** → usar `sklearn.pipeline.Pipeline` dentro de `cross_validate(cv=folds)`. Os modelling-datasets guardam features
  por codificar.
- O `OneRClassifier` (e qualquer classe própria) tem de estar num módulo `.py` importável (ex.: `sns_lib.py`), senão o
  `joblib.load` falha no `02_deployed_predictors.py`.

---

## 5. Notebook fornecido — TODOs a resolver (`00_sns_selected_solutions.ipynb`)

| Célula | TODO |
|---|---|
| 4 | nome da tabela de staging (`source_stage`) e mapeamento das colunas `diagnosis_description`, `age_group` ("Faixa Etária"), `num_ambulatory_episode` ("Ambulatório") |
| 5 | `DESCRIBE source_stage`; colunas `region, institution, diagnosis_code, diagnosis_description` |
| 8 | condição `WHERE id_institution IS NULL` |
| 9 | `diagnosis_description`, `COUNT(*) AS n_records`, condição `WHERE id_diagnosis IS NULL` |
| 10 | `COUNT(*) - COUNT(id_institution) AS n_unmatched_institution`, idem `id_diagnosis` |
| 12 | as três contagens de não correspondências |
| 14 | `SELECT 'matched', COUNT(*)` |
| 15 | `JOIN diagnosis d ON s.diagnosis_code = d.id AND s.diagnosis_description = d.description;` |
| 18 | ordem *worst-case*: `num_death DESC, num_inpatient_day DESC, num_inpatient_admission DESC, num_ambulatory_episode DESC,` |
| 19 | `CASE WHEN num_death > 0 THEN 1 ELSE 0 END AS has_death` |
| 21 | distribuição de `inpatient_level` (`COUNT(*) ... GROUP BY ... ORDER BY`) |
| 22–24 | (markdown TODO) investigar e discutir valores extremos de `num_inpatient_day`, sem alterar o analytical-dataset |

O notebook chama `_mLDm_00_sns_db_boot.py`; na cópia, apontar para `00_sns_db_boot.py` (versão docker).

---

## 6. Plano de tarefas (organizado pelos R, por ordem de execução; marcar `[x]` quando concluído)

> Organização decidida em 2026-11-24 (ver §7.2): o plano segue os requisitos R01–R10, executados pela ordem das
> dependências. O **R01 passa para o início** como decisão de desenho. O **relatório só é escrito no fim** (Fase 11):
> durante o trabalho guarda-se apenas a evidência (tabelas, figuras, números) indicada em cada fase.
>
> Ordem: Ambiente → R01 → R02 → R03 → R04 → R07 (folds) → R05 (construir) → R06 → R08 (+ comparação do R05) → R09 → R10 → relatório e entrega.
> O R07 é feito antes do R05/R06 porque o 1R e a comparação de representações têm de usar os mesmos folds temporais.

### Fase 0 — Ambiente (suporte)
- [ ] T01. Arrancar o Rancher Desktop e o container `mldm_postgres` (`docker compose up -d` em `__container/postgres`).
- [ ] T02. Criar `02_code/00_sns_db_boot.py` (cópia com `RUNTIME = "docker"`); correr e confirmar a BD `mldm_tep`.

### Fase 1 — R01: contrato de prediction-time (decisão de desenho)
- [ ] T03. Definir e documentar, antes de qualquer modelação, o que pode e não pode ser feature:
      - permitidas: os 5 atributos da unidade de observação e derivados deles (ex.: `year`, `month`,
        `region` via tabela de referência, `age_group` ordinal);
      - proibidas: internamentos, ambulatório, dias e óbitos do mês a prever (são eles que definem os alvos);
        agregados que incluam o mês a prever ou meses futuros; qualquer dado do dataset C;
      - regra: tudo o que aprende com os dados (OHE, scaling, taxas históricas, regras do 1R) é ajustado só no
        treino de cada fold (`Pipeline` + `cross_validate`).
- [ ] T04. Implementar a derivação de features numa função única em `02_code/sns_lib.py` (usada no treino e no deploy)
      e escolher o risco concreto a relatar no R01. Opcional: experiência de controlo (ajuste fora vs dentro do fold)
      para mostrar o efeito do leakage.
- Evidência a guardar: lista de features permitidas/proibidas; (opcional) resultados da experiência de controlo.

### Fase 2 — R02: análise do modelo relacional
- [ ] T05. Analisar `_mLDm_01_sns_schema.sql` e desenhar o diagrama ER. Relações: region 1 — 0..N institution
      (institution obriga a region); institution 1 — 0..N source_record; diagnosis 1 — 0..N source_record;
      `inpatient_level` sem FK (relação lógica por intervalo de `inpatient_ratio`). Identificadores: `region.id`,
      `institution.id`, `diagnosis.id` (código oficial), `source_record.id`, `inpatient_level.name`; unicidade
      canónica de nomes via índices sobre `get_canonical_name()`.
- Evidência a guardar: diagrama ER (figura/PDF em `04_report/`, fora do ZIP).
- Estado (2026-11-24): rascunho do ER feito (Anexo A); falta desenhá-lo no draw.io e exportar a figura.

### Fase 3 — R03: integração, consolidação e analytical-dataset
- [ ] T06. Copiar `_mLDm_00_sns_selected_solutions.ipynb` para `02_code/00_sns_selected_solutions.ipynb`, resolver
      todos os TODO (secção 5) e apontar o boot para `00_sns_db_boot.py`.
- [ ] T07. Confirmar os *expected outputs* (326 692 / 317 239 / 7 684 / níveis / estatísticas de dias).
- [ ] T08. TODO dos valores extremos: P99/P99.9, padrões por instituição/diagnóstico/idade, impacto em MAE vs RMSE,
      decisão (proposta: manter os dados; testar `log1p` do alvo via `TransformedTargetRegressor`).
- [ ] T09. Em `02_code/01_sns_build_modelling_datasets.ipynb`: integração genérica (loop A, B), repetível sem duplicar
      (DELETE + INSERT por intervalo de `period`: A = [2021-01-01, 2025-01-01[, B = [2025-01-01, 2026-07-01[),
      lista explícita de correções (Cascais em B), `assert` de que registos integrados = registos de origem.
- [ ] T10. Gravar `03_output/sns_integration_check.csv`.
- [ ] T11. Consolidação *worst-case* (mesma view para A e B) e quantificação por dataset: registos, unidades,
      repetidas, em excesso, conflitos; distribuição dos 3 alvos (por dataset e total).
- [ ] T12. Analytical-dataset (5 atributos + 3 alvos). Validar unicidade, alvos (domínios, coerência com as contagens),
      unidades A + B = linhas. Exportar `03_output/sns_analytical_dataset.csv`.
- Evidência a guardar: tabela A/B/total (registos, unidades, repetições, conflitos); distribuição dos alvos;
  estatísticas dos valores extremos de `num_inpatient_day`.

### Fase 4 — R04: impacto do critério worst-case
- [ ] T13. Regra alternativa (proposta: *best-case*, ordem ascendente, desempate `id ASC`); comparar `has_death`
      (taxa de positivos, nº de unidades que mudam de classe, por dataset).
- Evidência a guardar: tabela worst-case vs alternativa.

### Fase 5 — R07: partições temporais (definição)
- [ ] T14. Função de folds em `sns_lib.py`, janela expansiva anual: treino ≤2021→teste 2022, ≤2022→2023, ≤2023→2024,
      ≤2024→2025, ≤2025→2026 (jan–jun). Mesmos folds para todos os modelos de cada problema.
- Evidência a guardar: tabela de folds (períodos, nº de observações, distribuição dos alvos por fold).

### Fase 6 — R05: representações de features (construção)
- [ ] T15. Modelling-datasets por problema e representação (gravados em `03_output`, features por codificar), por exemplo:
      (a) só categóricas (institution, diagnosis, age_group, sex);
      (b) + temporais (`year`, `month` ou sin/cos do mês);
      (c) + derivadas (`region` via referência, `age_group` ordinal);
      (d) opcional: taxas históricas (ex.: óbito por instituição×diagnóstico) calculadas só no treino de cada fold.
      Excluir linhas com `inpatient_level` NULL só no dataset multiclasse. A comparação é feita na Fase 8.
- Evidência a guardar: descrição de cada representação (features, nº de colunas depois da codificação).

### Fase 7 — R06: OneRClassifier
- [ ] T16. Implementar `OneRClassifier` em `02_code/sns_lib.py` (compatível com sklearn: `BaseEstimator`, `ClassifierMixin`):
      `fit` com DuckDB `PIVOT` por feature, maioria por linha com desempate determinístico (classes ordenadas), erro total
      = Σ(total da linha − máximo), escolhe a feature de menor erro (desempate por ordem fixa); `predict` com classe
      maioritária global para valores não vistos; guardar matrizes/regras para análise.
- [ ] T17. Validar contra o exemplo do notebook (`id_diagnosis` × `inpatient_level`); avaliar com os folds do R07 nos
      dois problemas de classificação.
- Evidência a guardar: matrizes de contingência, feature escolhida, erros, comparação com a baseline maioritária, por fold.

### Fase 8 — R08: modelos vs baselines (e comparação do R05)
- [ ] T18. `02_code/03_sns_temporal_evaluation.ipynb`: `Pipeline` (pré-processamento + modelo) com
      `cross_validate(cv=folds)` para cada representação: Dummy (baseline), 1R, Decision Tree, kNN (com scaling),
      Random Forest; versões de classificação/regressão; grelhas pequenas de hiperparâmetros (ex.: `max_depth`,
      `min_samples_leaf`, `n_neighbors`, `n_estimators`, `class_weight`). Opcional: outro algoritmo.
- [ ] T19. Métricas por fold e média ± desvio; guardar resultados em `03_output/`.
- Evidência a guardar: tabela modelos × representações × métricas vs baselines; accuracy vs balanced accuracy/macro-F1;
  MAE vs RMSE.

### Fase 9 — R09: variação temporal e seleção
- [ ] T20. Analisar a variação do desempenho entre folds e escolher o modelo (e representação) de cada problema.
- Evidência a guardar: gráfico métrica × fold; justificação da escolha.

### Fase 10 — R10: deployment
- [ ] T21. `02_code/04_sns_final_models.ipynb`: retreinar os 3 pipelines escolhidos com todo A+B; guardar dicionário
      `{inpatient_level, num_inpatient_day, has_death, metadata}` em `03_output/sns_deployed_models.joblib`.
- [ ] T22. `02_code/02_deployed_predictors.py`: carrega o joblib, valida `_mLDm_00_sns_data_C.csv` (colunas, formato `YYYY-MM`,
      domínios), deriva features com a mesma função do treino (`sns_lib.py`), prevê e escreve
      `03_output/_mLDm_00_sns_data_C_prediction.csv` (5 colunas de entrada + 3 alvos, mesma ordem e nº de linhas).
      Decidir o formato de `num_inpatient_day` (proposta: arredondar e limitar a ≥ 0) e `has_death` como inteiro 0/1.
- [ ] T23. Demonstrar execução standalone, verificar input/output, ausência de `fit`, e valores não vistos com um
      ficheiro sintético à parte (sem alterar C).
- Evidência a guardar: output do script, verificações, resultado do teste com valores não vistos.

### Fase 11 — Relatório e entrega (só no fim)
- [ ] T24. Escrever `04_report/report.pdf` a partir da evidência guardada: capa + ≤ 10 páginas, secções 1–5,
      R01–R10 por ordem, tabelas/figuras, ficheiros de código relevantes. Orçamento indicativo de páginas:
      R01 0,5 · R02 1 · R03 1 · R04 0,5 · R05 1 · R06 1,25 · R07 0,5 · R08 1,5 · R09 1,25 · R10 0,75 · ligações 0,75.
- [ ] T25. `04_report/README.txt`: ambiente e dependências, inicialização da BD, ordem de execução, execução do
      `02_deployed_predictors.py`, ficheiros gerados.
- [ ] T26. `python _mLDm_xx_build_submission_zip_rev01.py 12` (em `02_code`); verificar `04_report/submission_check.txt`;
      testar a reprodução de ponta a ponta numa BD limpa.

---

## 7. Decisões

### 7.1 Em aberto (com recomendação)

| # | Decisão | Recomendação |
|---|---|---|
| D1 | Runtime da BD | criar `00_sns_db_boot.py` com `docker` (Rancher Desktop) |
| D2 | Representação de institution/diagnosis no analytical-dataset | nome da instituição + código do diagnóstico (formato de C), em vez dos ids internos |
| D3 | Granularidade dos folds temporais | anual (5 folds); eventualmente um fold extra só para 2026 S1 |
| D4 | Regra alternativa para R04 e alvo | *best-case* (ascendente) aplicada a `has_death` |
| D5 | Features históricas | começar só com as 5 dimensões + derivadas; taxas históricas (só no treino de cada fold) como representação alternativa. **Sem lags do mês anterior**: C vai até 2026-09 e só há dados até 2026-06 |
| D6 | Custo do kNN (~400k linhas) | subamostrar o treino só para o kNN, com seed fixa e documentado |
| D7 | Ferramenta do relatório e do ER | a definir pelo grupo (Word, LaTeX, draw.io…) |

### 7.2 Tomadas

| Data | Decisão | Justificação |
|---|---|---|
| 2026-11-24 | Plano organizado pelos R01–R10, executado pela ordem das dependências (R07 antes de R05/R06; comparação do R05 depois do R08) | o relatório segue os R; executar à letra pela ordem dos R obrigaria a refazer trabalho |
| 2026-11-24 | R01 passa para o início, como contrato de prediction-time (features permitidas/proibidas, ajuste só no treino de cada fold) | orienta todas as decisões de features, avaliação e deploy desde o arranque |
| 2026-11-24 | O relatório é escrito só no fim (Fase 11); durante o trabalho guarda-se apenas a evidência de cada R | preferência do utilizador |

_(registar aqui novas decisões: data, decisão, justificação)_

---

## 8. Riscos e cuidados

- Fazer OHE/scaling fora do fold = leakage (e foge ao que se fez no MoP_04: aqui é obrigatório usar Pipeline por fold).
- `OneRClassifier` fora de um módulo `.py` → o `joblib.load` falha no script de deploy.
- Valores raros (`sex = 'I'`) ou instituições ausentes num fold → `OneHotEncoder(handle_unknown="ignore")` ou categorias fixas da referência.
- `inpatient_level` NULL (I + A = 0): não perder estas unidades no analytical-dataset; excluir só no treino multiclasse.
- Desequilíbrio de classes: comparar accuracy com balanced accuracy e macro-F1; considerar `class_weight="balanced"`.
- Valores extremos de `num_inpatient_day` dominam o RMSE: comparar com MAE; considerar `log1p` do alvo.
- O último fold (2026) só tem 6 meses: referir na R07/R09.
- Não usar o dataset C em nenhuma fase antes do deploy.
- Reprodutibilidade: `random_state` fixo em todos os modelos/amostragens; nada manual fora do README.

---

## Anexo A — Diagrama ER (rascunho para o draw.io, R02)

Notação *crow's foot*, como no ER entregue no MoP_02 (`MoP_02_.../04_report/penguins_ER_model.pdf`), agora com atributos e chaves.

| Relação | Ponta do lado "1" | Ponta do lado "N" | Leitura | Origem no esquema |
|---|---|---|---|---|
| region — institution | `ERmandOne` (‖) em region | `ERzeroToMany` (o<) em institution | cada instituição pertence a exatamente 1 região; uma região tem 0..N instituições | `institution.id_region NOT NULL` + FK |
| institution — source_record | `ERmandOne` em institution | `ERzeroToMany` em source_record | cada registo é de exatamente 1 instituição; uma instituição tem 0..N registos | `source_record.id_institution NOT NULL` + FK |
| diagnosis — source_record | `ERmandOne` em diagnosis | `ERzeroToMany` em source_record | cada registo tem exatamente 1 capítulo de diagnóstico; um capítulo tem 0..N registos | `source_record.id_diagnosis NOT NULL` + FK |
| inpatient_level — source_record (linha tracejada) | `ERzeroToOne` (o\|) em inpatient_level | `ERzeroToMany` em source_record | relação lógica, **sem FK**: cada registo cai em 0..1 nível (0 quando I + A = 0); um nível abrange 0..N registos | intervalo `[min_ratio, max_ratio[` sobre `I / (I + A)` |

Notas para o texto da R02:
- `source_record` tem chave substituta (`id`); a chave natural da unidade de observação
  `(period, id_institution, id_diagnosis, age_group, sex)` **não** é única por desenho (repetições preservadas; consolidação é um passo posterior).
- `age_group` e `sex` são atributos com domínio fixo (`CHECK`), não entidades.
- `diagnosis.id` é o código oficial do capítulo (não gerado); `region.id` e `institution.id` são `IDENTITY`.
- Unicidade dos nomes também ao nível canónico: índices únicos sobre `get_canonical_name(name/description)`.
- `inpatient_level` é uma tabela de configuração isolada no modelo relacional.

Fonte mermaid (pode ser colada em draw.io → Organizar → Inserir → Avançado → Mermaid, ou em mermaid.live):

```
erDiagram
  REGION ||--o{ INSTITUTION : "agrupa"
  INSTITUTION ||--o{ SOURCE_RECORD : "reporta"
  DIAGNOSIS ||--o{ SOURCE_RECORD : "classifica"
  INPATIENT_LEVEL |o..o{ SOURCE_RECORD : "logica, sem FK"
  REGION {
    integer id PK "identity"
    text name UK "NOT NULL"
  }
  INSTITUTION {
    integer id PK "identity"
    text name UK "NOT NULL"
    integer id_region FK "NOT NULL"
  }
  DIAGNOSIS {
    integer id PK "codigo oficial SNS"
    text description UK "NOT NULL"
  }
  SOURCE_RECORD {
    bigint id PK "identity"
    date period "NOT NULL, YYYY-MM-01"
    integer id_institution FK "NOT NULL"
    integer id_diagnosis FK "NOT NULL"
    text age_group "CHECK 7 grupos"
    varchar1 sex "CHECK F, M, I"
    integer num_inpatient_admission "CHECK >= 0"
    integer num_inpatient_day "CHECK >= 0"
    integer num_ambulatory_episode "CHECK >= 0"
    integer num_death "CHECK >= 0"
  }
  INPATIENT_LEVEL {
    text name PK "low, medium, high"
    numeric min_ratio "NOT NULL, >= 0"
    numeric max_ratio "NOT NULL, <= 1"
  }
```
