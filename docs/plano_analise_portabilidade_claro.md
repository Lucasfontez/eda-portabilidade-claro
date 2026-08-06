# PLANO DE ANÁLISE — Portabilidade Numérica Claro Brasil

**Projeto:** Análise de Portabilidade e Churn — Claro Brasil
**Analista:** Lucas
**Referência:** BRIEF_Claro_Portabilidade_EDA.md
**Estrutura:** Notebook único com seções (`01_ingestao_e_limpeza.ipynb` → análise completa)
**Dataset:** `COMPLETO_Migração_Claro_Brasil__Portabilidade__postgresql.sql` (intocado em `data/raw/`)

---

## Estrutura do Notebook

### 01. Setup e Configuração do Ambiente
- Importação de pacotes (pandas, numpy, sqlite3, scipy, statsmodels, seaborn, matplotlib)
- Configurações globais de exibição e temas visuais de gráficos

### 02. Ingestão, Saneamento e Validação Relacional
- Parsing do arquivo SQL via banco SQLite em memória (`open()` → `read()` → `executescript()` → `pd.read_sql()`)
- Pré-processamento da string SQL para compatibilidade PostgreSQL → SQLite (tratamento via Python, nunca editando o arquivo original)
- Coerção de tipos de dados (`dtypes`: datetime, float, category)
- Sanity check de nulos, duplicatas e integridade referencial entre Fato e Dimensões
- Tratamento e padronização de strings (strip, upper/lower)
- Renomeação explícita de colunas ambíguas com dicionário documentado:

| Nome original (DDL) | Nome renomeado | Justificativa |
|---|---|---|
| `direcao` | `direcao_migracao` | Semântica clara |
| `motivo` | `motivo_migracao` | Distingue de outros possíveis motivos |
| `canal` | `canal_aquisicao` | Descreve função do canal |
| `tempo_permanencia_dias` | mantém | Já é descritivo |
| `nota_satisfacao` | mantém | Já é descritivo |

- **Auditoria de colunas desnormalizadas:** cruzamento entre `FatoMigracao` e dimensões (`DimCliente`, `DimServico`) para mapear divergências — achados registrados como output da seção

### 03. Análise Exploratória de Dados (EDA) e Detecção de Anomalias
- Checagem da janela temporal dos 36 meses e completude mensal
- Análise de distribuição e detecção de outliers em variáveis numéricas (`valor_mensal_servico`, `tempo_permanencia_dias`, `nota_satisfacao`)
- Inspeção de valores categoricamente inconsistentes ou omissões atípicas
- Sumarização inicial da qualidade das variáveis
- **Objetivo de negócio:** identificar anomalias injetadas nos dados antes de qualquer análise — achados aqui podem mudar premissas das seções seguintes

### 04. Q1 — Saldo de Portabilidade (Volume e Tendência Temporal)
- **Filtro:** `status_portabilidade == 'Concluída'` (migrações efetivadas)
- Agregação mensal de entradas (IN), saídas (OUT) e cálculo do Net Migration
- Análise de tendência temporal ao longo dos 3 anos (série temporal + média móvel)
- **Pergunta de negócio:** A Claro está ganhando ou perdendo base? Qual a tendência?

### 05. Q2 — Impacto Financeiro (Receita Líquida e Ticket Médio)
- Receita entrante (IN) vs. receita perdida (OUT) mês a mês
- Cálculo do ARPU Delta (`ARPU_entrada − ARPU_saida`) por período
- Balanço de receita líquida discriminado por categoria de serviço
- **Pergunta de negócio:** O dinheiro que entra compensa o que sai? Onde está o gap?

### 06. Q3 — Perfil de Perda e Matriz de Concorrência
- Matriz de migração por operadora concorrente (quem rouba mais base da Claro)
- Ranqueamento dos principais motivos de saída
- Segmentação demográfica: concentração por região/UF, faixa etária e serviço
- **Pergunta de negócio:** Quem está saindo, para onde e por quê?

### 07. Q4 — Desempenho dos Canais de Aquisição
- Volume absoluto de captação e mix por canal × categoria de serviço
- Evolução temporal da participação do canal digital (App/Site) — crescimento vs. estagnação
- Análise de taxa de eficiência de portabilidade (Concluídas vs. Canceladas) por canal
- **Pergunta de negócio:** Quais canais trazem mais clientes? O digital está crescendo?

### 08. Q5 — Satisfação e Tempo de Permanência (Tenure)
- Comparação da `nota_satisfacao` entre clientes entrantes (IN) e saídos (OUT)
- Análise da distribuição de `tempo_permanencia_dias` por categoria de serviço
- Mapeamento de padrões temporais de churn precoce
- **Pergunta de negócio:** Clientes que saem estavam mais insatisfeitos? Há padrão de permanência?

### 09. Testes Estatísticos de Hipóteses

#### H1 — Satisfação: Clientes IN vs. OUT
- **H₀:** μ_out = μ_in (nota média de satisfação dos clientes que saem é igual à dos que entram)
- **H₁:** μ_out < μ_in (nota média dos que saem é significativamente menor)
- **Variáveis:** `nota_satisfacao` × `direcao_migracao` (IN vs OUT)
- **Tipo:** Numérica discreta/ordinal × Categórica binária
- **Teste:** Mann-Whitney U (não-paramétrico) — avaliar normalidade antes; se normal, Teste t

#### H2 — ARPU Delta: Ticket Médio das Perdas vs. Ganhos
- **H₀:** μ_arpu_out = μ_arpu_in (ticket médio dos clientes perdidos é igual ao dos entrantes)
- **H₁:** μ_arpu_out > μ_arpu_in (ticket médio dos perdidos é significativamente maior)
- **Variáveis:** `valor_mensal_servico` × `direcao_migracao` (IN vs OUT)
- **Tipo:** Numérica contínua × Categórica binária
- **Teste:** Teste t / Mann-Whitney U, complementado por ANOVA de duas vias por `categoria_servico`

#### H3 — Permanência: Canal Digital vs. Canal Físico
- **H₀:** μ_tenure_digital = μ_tenure_fisico (tempo de permanência de clientes do canal digital é igual ao dos canais físicos)
- **H₁:** μ_tenure_digital < μ_tenure_fisico (clientes do canal digital têm permanência menor)
- **Variáveis:** `tempo_permanencia_dias` × `canal_aquisicao` (agrupado: Digital vs Físico)
- **Tipo:** Numérica contínua × Categórica nominal
- **Teste:** Mann-Whitney U / Kruskal-Wallis + Curva de Sobrevivência (Log-Rank Test)
- **Agrupamento:** Digital = {App Minha Claro, Site Claro} | Físico = {Loja Própria, Loja Autorizada, Parceiro/Correspondente, Porta a Porta, Central de Atendimento}

#### H4 — Associação: Motivo de Saída × Categoria de Serviço
- **H₀:** `motivo_migracao` é independente de `categoria_servico`
- **H₁:** Existe associação estatisticamente significativa entre motivo e categoria
- **Variáveis:** `motivo_migracao` × `categoria_servico`
- **Tipo:** Categórica nominal × Categórica nominal
- **Teste:** Chi-Quadrado de Independência (χ²)

### 10. Conclusões de Negócio e Recomendações Estratégicas
- Resumo dos principais achados por pergunta de negócio
- Plano de ação recomendado para retenção e otimização de portabilidade
- Entregável: resumo executivo em Markdown (1 página) para a VP Renata Almeida

---

## Decisões Técnicas Documentadas

### Colunas Desnormalizadas na FatoMigracao

**Regra adotada:**
- A `FatoMigracao` armazena a "foto do evento" — os dados no momento exato da portabilidade (UF, região, categoria do serviço naquela data)
- As dimensões (`DimCliente`, `DimServico`) representam o cadastro mestre/atualizado

**Uso nas análises:**
- Seções 04 a 08 (perguntas de negócio): colunas da `FatoMigracao` como fonte primária, pois refletem o contexto histórico da transação
- Seção 02: auditoria cruzada entre Fato e Dimensões para identificar divergências (possíveis anomalias injetadas)
- Merge com dimensões apenas quando a análise exigir atributos exclusivos da dimensão (ex: `nome_servico`, `nome` do cliente, `tipo` da operadora)

### Filtro de Status

| Status | Uso |
|---|---|
| `Concluída` | Base para todos os KPIs de volume e receita (seções 04 a 08) |
| `Em Processamento` | Analisado separadamente — indica pipeline ativo, não conversão |
| `Cancelada pelo Cliente` | Analisado separadamente — indica desistência, útil para taxa de eficiência (seção 07) |

### Compatibilidade PostgreSQL → SQLite

O arquivo SQL foi gerado em dialeto PostgreSQL. Tratamento feito via Python (nunca editando o arquivo original):
- Comentários (`--`): SQLite ignora nativamente — sem ação necessária
- `NUMERIC(18,2)`: SQLite aceita mas ignora precisão — verificar arredondamentos após carga
- `VARCHAR(n)`: SQLite trata como TEXT — sem impacto funcional
- `CONSTRAINT ... PRIMARY KEY`: sintaxe aceita pelo SQLite
- `CREATE INDEX`: sintaxe simples, aceita pelo SQLite
- Estratégia: `open()` → `read()` → `conn.executescript()` — se erro, tratar a linha específica no Python

---

## Stack Técnica

| Componente | Ferramenta |
|---|---|
| IDE | VS Code + extensão Jupyter |
| Linguagem | Python 3.x (venv) |
| Ingestão | sqlite3 (stdlib) |
| Manipulação | pandas, numpy |
| Visualização | matplotlib, seaborn |
| Estatística | scipy, statsmodels |
| Dados | `data/raw/` (imutável) → `data/processed/` (outputs) |