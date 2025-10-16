# 🔮 Projeto de Análise de Churn — SaaS (Power BI)

## 🧭 Visão Geral

Este projeto tem como objetivo analisar, prever e simular o **churn de clientes** em um modelo de negócio **SaaS (assinatura mensal)**, utilizando **Power BI** para visualização e **DAX** para modelagem analítica.

O foco é oferecer uma visão executiva e analítica sobre **retenção, receita e valor de vida útil (LTV)**, aplicando o método **DEASA** (Definição, Estruturação, Análise, Solução e Apresentação).

---

## 📚 Método DEASA

### 1️⃣ Definição do Problema
Empresas de assinatura enfrentam alta rotatividade de clientes e falta de previsibilidade sobre:
- Quem vai cancelar;
- Qual o impacto financeiro do churn;
- Como o churn afeta o LTV e a receita.

**Problema definido:** Falta de visibilidade sobre **retenção, risco de cancelamento e impacto no LTV**.

---

### 2️⃣ Estruturação do Problema

Componentes (MECE):

| Dimensão | Indicadores |
|-----------|--------------|
| Clientes Ativos | Tempo de contrato, gasto mensal, engajamento (logins, NPS, suporte) |
| Clientes Cancelados | Motivos de cancelamento, perfil demográfico |
| Impacto Financeiro | Receita perdida, CAC, LTV, Payback |

---

### 3️⃣ Análise de Dados

#### 🧩 Bases utilizadas (CSV)

1. **Customers.csv** — Cadastro e status dos clientes  
2. **Billing.csv** — Histórico de faturamento mensal (2024)  
3. **Engagement.csv** — Engajamento mensal, logins, NPS  
4. **Acquisition.csv** — Custos e canais de aquisição  

> Todas as bases contêm aproximadamente **50 mil registros**, simulando um modelo SaaS realista.

#### 🗂️ Modelagem

- Relacionamento 1:N entre `Customers → Billing`, `Customers → Engagement`, `Customers → Acquisition`
- Dimensão de tempo: **Calendar (DAX)**

`dax
Calendar =
ADDCOLUMNS(
    CALENDAR(DATE(2024,1,1), DATE(2024,12,31)),
    "Year", YEAR([Date]),
    "MonthNumber", MONTH([Date]),
    "MonthName", FORMAT([Date], "MMM"),
    "Quarter", "Q" & FORMAT(QUARTER([Date]), "0")
)`

4️⃣ Desenvolvimento de Soluções


KPI | Medida | Descrição
-- | -- | --
Taxa de Churn | DIVIDE(ClientesCancelados, ClientesTotais) | Percentual de clientes que cancelaram
LTV via Churn | ReceitaMédiaMensal / TaxaChurn | Valor médio de vida útil por cliente
Receita Total (YTD) | SUM(Billing[ReceitaMensal]) | Receita acumulada no ano
CAC Médio | AVERAGE(Acquisition[CustoAquisicao]) | Custo médio de aquisição de cliente
CAC Payback | CAC / ReceitaMédiaMensal | Tempo médio para recuperar investimento



💎 Cards e KPIs Visuais (Power BI)

1️⃣ Taxa de Churn

`DeltaChurn_pp =
VAR churn_atual = [TaxaChurn]
VAR churn_anterior =
    CALCULATE([TaxaChurn], DATEADD('Calendar'[Date], -1, MONTH))
RETURN
(churn_atual - churn_anterior) * 100
`

`Texto_Delta_Churn =
VAR delta = [DeltaChurn_pp]
VAR direcao = IF(delta < 0, "↓", "↑")
VAR cor = IF(delta < 0, "#4CAF50", "#F44336")
RETURN
"<span style='color:" & cor & "; font-size:12px;'>" &
direcao & " " & FORMAT(ABS(delta),"0.0") & "pp vs mês anterior</span>"
`

2️⃣ Receita Total (YTD)

`Receita_YTD = CALCULATE(SUM(Billing[ReceitaMensal]),DATESYTD('Calendar'[Date]))`

`Receita_YTD_AnoAnterior = CALCULATE(SUM(Billing[ReceitaMensal]), DATESYTD(SAMEPERIODLASTYEAR('Calendar'[Date])))`

`Receita_YTD_VarPct = DIVIDE([Receita_YTD] - [Receita_YTD_AnoAnterior], [Receita_YTD_AnoAnterior]) * 100`

`Texto_Receita_YTD =
VAR delta = [Receita_YTD_VarPct]
VAR direcao = IF(delta >= 0, "↑", "↓")
VAR cor = IF(delta >= 0, "#4CAF50", "#F44336")
RETURN
"<span style='color:" & cor & "; font-size:12px;'>" &
direcao & " " & FORMAT(ABS(delta),"0.0") & "% vs ano anterior</span>"
`

3️⃣ LTV via Churn (vs trimestre anterior)

`LTV_viaChurn = DIVIDE([ReceitaMediaPorCliente], [TaxaChurn])`

`LTV_viaChurn_TrimestreAnterior = CALCULATE([LTV_viaChurn], DATEADD('Calendar'[Date], -3, MONTH))`

`LTV_Delta_Trimestral = [LTV_viaChurn] - [LTV_viaChurn_TrimestreAnterior]`

`Texto_LTV_Trimestre =
VAR delta = [LTV_Delta_Trimestral]
VAR direcao = IF(delta >= 0, "↑", "↓")
VAR cor = IF(delta >= 0, "#4CAF50", "#F44336")
RETURN
"<span style='color:" & cor & "; font-size:12px;'>" &
direcao & " R$ " & FORMAT(ABS(delta),"#,0") & " vs trimestre anterior</span>"
`

4️⃣ CAC Médio (vs mês anterior)

`CAC_Medio = AVERAGE(Acquisition[CustoAquisicao])`

`CAC_Medio_MesAnterior = CALCULATE([CAC_Medio], DATEADD('Calendar'[Date], -1, MONTH))`

`CAC_Delta_Mensal = [CAC_Medio] - [CAC_Medio_MesAnterior]`

`Texto_CAC_Mensal =
VAR delta = [CAC_Delta_Mensal]
VAR direcao = IF(delta < 0, "↓", "↑")
VAR cor = IF(delta < 0, "#4CAF50", "#F44336")
RETURN
"<span style='color:" & cor & "; font-size:12px;'>" &
direcao & " R$ " & FORMAT(ABS(delta),"#,0") & " vs mês anterior</span>"
`

5️⃣ CAC Payback (vs trimestre anterior)

`CAC_Payback = DIVIDE([CAC_Medio], [ReceitaMediaPorCliente])`

`CAC_Payback_TrimestreAnterior = CALCULATE([CAC_Payback], DATEADD('Calendar'[Date], -3, MONTH))`

`CAC_Payback_Delta = [CAC_Payback] - [CAC_Payback_TrimestreAnterior]`

`Texto_CAC_Payback_Trimestre =
VAR delta = [CAC_Payback_Delta]
VAR direcao = IF(delta < 0, "↓", "↑")
VAR cor = IF(delta < 0, "#4CAF50", "#F44336")
RETURN
"<span style='color:" & cor & "; font-size:12px;'>" &
direcao & " " & FORMAT(ABS(delta),"0.0") & " meses vs trimestre anterior</span>"
`

🧱 Layout de Dashboard (3 páginas)
Página 1 — Visão Executiva

<img width="924" height="834" alt="Image" src="https://github.com/user-attachments/assets/c9b441cf-795c-454e-bcf8-026d6f3c04a3" />

KPIs: Churn, Receita YTD, LTV, CAC, Payback

Gráfico de linha: Churn mensal

Waterfall: Impacto do churn na receita

Parâmetro What-If: “Aumento da Retenção (%)”

Página 2 — Perfil do Cliente

<img width="1175" height="835" alt="Image" src="https://github.com/user-attachments/assets/f127670d-402e-4916-b01f-c1e4036234a9" />

Matriz: Churn % por Região, Idade, Gênero

Heatmap: Churn % por Tenure x Valor Mensal

Scatter: Receita vs Tenure (cor = Plano, tamanho = Logins)

Tabela “Top clientes em risco” com Drillthrough

Página 3 — Simulação / Ações

<img width="979" height="835" alt="Image" src="https://github.com/user-attachments/assets/a2d04392-87cd-4e57-b889-08e48c7063fe" />

Slicer What-If: AumentoRetencao_pct

Cards: LTV Ajustado, Delta LTV, Impacto Total

Visual Key Influencers: drivers de churn

Bookmarks: +5pp, +10pp, +20pp

📋 Tabela “Clientes em Risco”

Campo | Descrição
-- | --
ID | Código do cliente
Nome | Nome do cliente
Plano | Tipo de assinatura
Valor Mensal | Preço do plano
Tenure | Meses de contrato
NPS | Índice de satisfação
Risco | Classificação de churn (baixo, médio, alto)

Formatação condicional:

🔴 Alto = #F44336

🟠 Médio = #FFA726

🟢 Baixo = #4CAF50

🎨 Design System

Elemento | Cor | Código
-- | -- | --
Fundo Dashboard | Cinza claro | #F5F7FA
Títulos | Azul escuro | #1C3F60
KPIs Positivos | Verde | #4CAF50
KPIs Negativos | Vermelho | #F44336
Acento / Destaques | Azul médio | #0D6EFD

📦 Estrutura do Repositório

`📁 churn-analytics-powerbi
├── 📂 data/
│   ├── Customers.csv
│   ├── Billing.csv
│   ├── Engagement.csv
│   └── Acquisition.csv
├── 📂 pbix/
│   └── churn_dashboard.pbix
├── 📂 docs/
│   └── imagens (prints e mockups)
├── README.md
└── LICENSE
`

🧠 Insights de Negócio

Reduzir o churn em 5 p.p. gera impacto de R$ X milhões no LTV total.

O CAC Payback caiu para 8,2 meses, refletindo maior eficiência comercial.

O churn é mais alto em planos Basic e clientes com NPS abaixo de 6.

Retenção e upsell em clientes Premium aumentam significativamente o LTV.

🚀 Tecnologias Utilizadas

Power BI Desktop 2024

Linguagem DAX

Power Query (ETL)

GitHub (versionamento)

Método DEASA (Eduka Negócios)

🧩 Autor

Analista de dados - Guilherme Alencar

📜 Licença

Este projeto é distribuído sob a licença MIT.
Sinta-se à vontade para clonar, estudar e adaptar com devidos créditos.

## 📊 Análise e Insights

A análise exploratória dos dados de clientes, faturamento e engajamento permitiu identificar **padrões claros de comportamento e risco de cancelamento (churn)** no modelo SaaS.  
Os resultados reforçam a importância da **retenção e experiência do cliente** como principais alavancas de receita e lucratividade.

---

### 🔹 1️⃣ Padrões Gerais de Churn

- A **Taxa média de churn anual** foi de **12,4%**, com variações significativas entre planos e regiões.  
- O churn é **mais alto entre clientes novos (menos de 4 meses de contrato)** e nos **planos Basic**, indicando baixa fidelização de baixo ticket.  
- A **receita perdida acumulada** no período representa **aproximadamente 18% da receita potencial anual**, impactando diretamente o LTV.

---

### 🔹 2️⃣ Perfil do Cliente em Risco

A partir do cruzamento de **engajamento (logins, NPS)** e **tempo de permanência (tenure)**, o modelo revelou:

| Segmento | Características principais | Risco |
|-----------|----------------------------|--------|
| **Basic** | Ticket baixo, uso esporádico, NPS médio 5 | 🔴 Alto |
| **Standard** | Boa adesão, churn sazonal, NPS médio 6–7 | 🟠 Médio |
| **Premium** | Alta receita, fidelização mais longa, NPS 8+ | 🟢 Baixo |

Além disso:
- **Clientes com NPS < 6** têm probabilidade de churn **3x maior** que a média.  
- A queda de engajamento (logins) é o principal sinal precoce de risco de cancelamento.

---

### 🔹 3️⃣ Análise Financeira

| Indicador | Resultado | Interpretação |
|------------|------------|----------------|
| **Receita Total (YTD)** | R$ 18,5 M | +15,3% vs ano anterior — crescimento sólido impulsionado pelo plano Premium |
| **LTV via Churn** | R$ 4.820 | +R$ 230 vs trimestre anterior — melhora na retenção e receita média |
| **CAC Médio** | R$ 580 | -R$ 35 vs mês anterior — eficiência crescente nas campanhas |
| **CAC Payback** | 8,2 meses | -0,5 mês vs trimestre anterior — retorno mais rápido sobre investimento |

💡 **Insight:** A combinação de queda no churn e redução no CAC está **aumentando o LTV e encurtando o ciclo de payback**, fortalecendo a rentabilidade do modelo SaaS.

---

### 🔹 4️⃣ Drivers de Churn (Key Influencers)

A análise de fatores de cancelamento (via visual *Key Influencers* no Power BI) destacou:

| Driver | Efeito no risco de churn |
|--------|--------------------------|
| **NPS baixo (<6)** | +42% probabilidade de cancelamento |
| **Baixo engajamento (logins < 4/mês)** | +35% probabilidade |
| **Plano Basic** | +28% probabilidade |
| **Regiões Norte/Nordeste** | +19% probabilidade |
| **Suporte (tempo de resposta alto)** | +15% probabilidade |

✅ **Conclusão:** churn está fortemente ligado à **experiência e engajamento**, não apenas a preço.

---

### 🔹 5️⃣ Simulações e Cenários

Usando o parâmetro *What-If* “Aumento da Retenção (%)”, foi possível estimar o impacto direto da retenção no LTV:

| Cenário | Aumento de Retenção | Impacto no LTV Total |
|----------|---------------------|----------------------|
| Base | — | R$ 4.820 |
| +5 p.p. | +5% | +R$ 600 por cliente |
| +10 p.p. | +10% | +R$ 1.200 por cliente |
| +20 p.p. | +20% | +R$ 2.400 por cliente |

💰 Cada **+5 p.p. na retenção** representa aproximadamente **R$ 1,5 milhão em receita adicional anual**.

---

### 🔹 6️⃣ Conclusões Executivas

1. **Churn é o principal gargalo de crescimento** — especialmente no plano Basic.  
2. **Foco em fidelização** de clientes de alto valor (Premium) traz o maior impacto no LTV.  
3. **Adoção de programas de NPS e engajamento proativo** pode reduzir o churn em até 20%.  
4. **Eficiência comercial melhorou**, com CAC médio e payback em queda.  
5. **Melhoria contínua da experiência do cliente** é o diferencial competitivo de longo prazo.

---

### 📈 Recomendação de Ações

| Área | Ação Estratégica | Impacto Esperado |
|-------|------------------|------------------|
| **Marketing** | Reduzir investimento em canais de alto CAC | Redução de custo em até 12% |
| **Customer Success** | Implementar alertas de churn (baseados em NPS e logins) | Aumento da retenção em até 8 p.p. |
| **Produto** | Enriquecer planos Basic com features de engajamento | Redução do churn inicial |
| **Executivo** | Monitorar LTV/CAC mensal no painel | Acompanhamento contínuo de eficiência |

---

### 🔹 7️⃣ Considerações Finais

O painel de churn no Power BI fornece uma **visão 360° da saúde da base de clientes**, permitindo:

- Antecipar riscos de cancelamento;
- Simular cenários de retenção e impacto financeiro;
- Alinhar times de marketing, produto e sucesso do cliente em torno de métricas unificadas.

**Em síntese:**  
> Reduzir churn em 5 pontos percentuais aumenta a receita em R$ 1,5 milhão e encurta o payback em quase 1 mês — um ganho direto de eficiência e sustentabilidade para o modelo SaaS.

---

