# BRIEFING — Análise de Portabilidade Numérica | Claro Brasil

**De:** Diretoria de Retenção e Crescimento de Base — Claro Brasil
**Para:** Equipe de Analytics
**Data:** 05/08/2026
**Prioridade:** Alta
**Deadline:** 12/08/2026

---

## 1. Contexto da Empresa

A Claro Brasil é a segunda maior operadora de telecomunicações do país, atuando em telefonia móvel (pré, controle, pós-pago), banda larga fixa (fibra), TV por assinatura, combos convergentes e soluções empresariais/M2M. Opera em todos os 26 estados + DF e compete diretamente com Vivo, TIM, Oi, além de operadoras regionais (Algar Telecom, Surf Telecom) e MVNOs (Correios Celular).

O mercado de telecomunicações brasileiro opera sob regime de **portabilidade numérica** regulado pela Anatel desde 2008: qualquer cliente pode migrar de operadora mantendo seu número. Isso transformou portabilidade no principal indicador de competitividade do setor — cada migração representa um voto do consumidor sobre qual operadora entrega mais valor.

## 2. Momento de Mercado

- Crescimento orgânico (novos assinantes) está estagnado — o Brasil tem mais linhas ativas que habitantes
- A disputa agora é **roubar base da concorrência**, não expandir mercado
- A Claro investiu pesado em expansão 5G e fibra óptica nos últimos 2 anos
- A diretoria precisa saber se esse investimento está **convertendo em ganho de base via portabilidade** ou se a empresa está **perdendo mais clientes do que ganha**

## 3. Problema de Negócio

A VP de Retenção suspeita que a Claro está com **saldo negativo de portabilidade em categorias de alto valor** (pós-pago, combos, empresarial), enquanto o ganho ocorre majoritariamente em pré-pago — clientes de baixa receita. Se confirmado, o faturamento recorrente está sendo corroído mesmo que o volume absoluto de migrações pareça saudável.

Além disso, existem sinais de que:
- Determinadas regiões concentram perdas desproporcionais
- O canal digital (App/Site) está subutilizado para aquisição via portabilidade
- Os motivos de saída mudaram ao longo dos 3 anos analisados (2022–2024)

A diretoria quer uma **análise exploratória completa** antes de decidir onde alocar o orçamento de retenção do próximo trimestre.

## 4. Perguntas de Negócio

A análise deve responder:

1. **Saldo de portabilidade:** A Claro está ganhando ou perdendo base? Qual o net migration mês a mês e a tendência ao longo dos 3 anos?

2. **Impacto financeiro:** Qual o saldo de receita líquida mensal das migrações? O dinheiro que entra (novos clientes) compensa o que sai (clientes perdidos)? Quebrar por categoria de serviço.

3. **Perfil de perda:** Quem está saindo da Claro? Para quais concorrentes? Com qual motivo? Existe concentração por região, faixa etária ou tipo de serviço?

4. **Canais de aquisição:** Quais canais trazem mais clientes via portabilidade? Qual o mix canal × categoria de serviço? O digital está crescendo ou estagnado?

5. **Satisfação e permanência:** Existe relação entre tempo de permanência e direção da migração? Clientes que saem tinham notas de satisfação menores? Há padrão temporal (saem mais rápido em determinado serviço)?

## 5. KPIs Esperados

| KPI | Definição | Meta de referência |
|---|---|---|
| **Net Migration** | (migrações IN) − (migrações OUT) por período | Positivo e crescente |
| **Net Revenue Impact** | Σ receita_liquida_mensal por período | Positivo |
| **Churn Rate por Portabilidade** | OUT / base ativa estimada | < 2% mensal |
| **Win-back Ratio** | IN / OUT por concorrente | > 1.0 para cada concorrente |
| **ARPU Migração IN vs OUT** | Média do valor_mensal_servico por direção | ARPU IN ≥ ARPU OUT |
| **Taxa de Conclusão** | Concluída / total de solicitações | > 85% |
| **NPS Proxy** | Média nota_satisfacao por segmento | ≥ 7 |

## 6. Stakeholders e Expectativas

| Stakeholder | Cargo | O que espera ver |
|---|---|---|
| **Renata Almeida** | VP Retenção e Crescimento de Base | Visão executiva: estamos ganhando ou perdendo? Onde está o problema? Onde investir? |
| **Carlos Henrique** | Gerente de Canais Digitais | Performance do App e Site vs. canais físicos. Argumentos para aumentar budget digital |
| **Patrícia Souza** | Coordenadora Regional (NE/N) | Diagnóstico regional: por que Norte e Nordeste podem estar descolados do nacional? |
| **André Nakamura** | Analista Sênior de Pricing | Relação entre valor do serviço e propensão a migrar. Categorias onde preço é o motivo dominante |

## 7. Dataset

Arquivo SQL (PostgreSQL) com 5 tabelas em modelo estrela:

- **FatoMigracao** — 50.000 registros de portabilidade (2022-01 a 2024-12)
- **DimCliente** — 8.000 clientes com CPF, UF, faixa etária, região
- **DimServico** — 21 serviços em 6 categorias
- **DimOperadora** — 7 operadoras (Claro + 6 concorrentes)
- **dCalendario** — 1.096 dias (calendário completo 2022–2024)

Anomalias e concept drift foram injetados nos dados — espera-se que o analista identifique e documente achados inesperados.

**Arquivo:** `COMPLETO_Migração_Claro_Brasil__Portabilidade__postgresql.sql`

## 8. Entrega Esperada

- **Formato:** Jupyter Notebook comentado + resumo executivo em Markdown (1 página)
- **Seções obrigatórias:** Setup → Ingestão → Limpeza/Validação → EDA por dimensão de negócio → Visualizações → Conclusões e Recomendações
- **Prazo:** 7 dias corridos
- **Apresentação:** 15 minutos para a VP + stakeholders (slides opcionais, notebook projetado é aceito)

## 9. Restrições e Premissas

- Dados sensíveis (CPF) devem ser tratados — não expor em visualizações
- A coluna `receita_liquida_mensal` já traz sinal (+ para IN, − para OUT) — não duplicar a lógica
- Status "Em Processamento" e "Cancelada pelo Cliente" devem ser analisados separadamente das migrações efetivadas ("Concluída")
- A fato tem colunas desnormalizadas (uf, regiao, categoria_servico) — validar consistência com as dimensões

---

*"Não quero um relatório bonito. Quero saber onde estou sangrando cliente e quanto isso custa."*
— Renata Almeida, VP Retenção

---

**Próximo passo:** Analista apresenta o plano de análise (estrutura do notebook, hipóteses, abordagem por pergunta) antes de abrir o Jupyter.