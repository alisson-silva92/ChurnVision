🧠 ChurnVision — Previsão de Churn (Regras + Machine Learning)

Projeto para estimar o risco de churn por cliente, combinando:

1️⃣ Score baseado em regras — recência, tendência, momento de queda, ticket médio e NPS
2️⃣ Modelo supervisionado (Logistic Regression) — pipeline com threshold otimizado, priorizando Recall / F2-score

🎯 Saída principal: clientes_risco_final.csv contendo o risco final de churn por cliente.

🧩 Estrutura do Projeto
previsao_churn/
├─ ChurnVision.ipynb          # Notebook principal
├─ *.xlsx                     # Bases mensais (ano, mes, Contagem, etc.)
├─ cancelados.xlsx            # Histórico de cancelamentos
├─ NPS.xlsx                   # Notas de NPS por cliente
├─ tkt_medio/
│  └─ ticket_medio.csv        # Ticket médio por cliente
└─ clientes_risco_final.csv   # Gerado ao final da execução

⚙️ Parâmetros Principais (configuráveis no topo do script)
# Janelas de cálculo
N_RECENT = 3           # janelas recentes (média recente)
N_BASE   = 6           # janela base para comparação
CAP_DIAS = 180         # limite máximo para recência (dias)
GAMMA    = 0.7         # suavização da normalização (potência)

# Pesos e ajustes
W_MOMENTO, W_TEND, W_REC = 0.50, 0.35, 0.15
ALPHA = 0.5               # mistura entre risco_raw e ranking percentual
INFLUENCIA_TICKET = 0.15
W_NPS = 0.15
NEUTRO_NPS = 0.30

# Rotulagem e modelo
K_PRAZO_MESES = 2          # janela de rotulagem à frente (em meses)
TARGET_RECALL = 0.40       # recall mínimo desejado
THR_GRID = np.linspace(0.20, 0.60, 21)

📄 Estrutura Esperada das Bases
Arquivo	Campos	Observações
Bases mensais (.xlsx)	CardCode, ano, mes, Contagem, created, Start	Contagem = medida de uso/atividade
ticket_medio.csv	CardCode, TicketMedio	Usado para peso de valor do cliente
NPS.xlsx	CardCode, NPS	Renomeado internamente para nota_nps
cancelados.xlsx	CardCode, dt_cancelamento, dt_aquisicao	Data mais antiga usada por cliente

Campos textuais são stripados e as colunas de datas/números são convertidas com errors='coerce'.

🧠 Lógica de Funcionamento
1️⃣ Painel Mensal

Cria uma série contínua CardCode x mês, preenchendo meses ausentes com Contagem=0.
Calcula os seguintes sinais de comportamento:

Tendência (slope relativo)

Momento de queda (média recente vs base)

Recência (dias sem uso)

Engajamento (acessos, dias ativos, tempo de uso)

2️⃣ Score por Regras

Normaliza os sinais com MinMaxScaler e aplica potência GAMMA.

Combina pesos W_MOMENTO, W_TEND, W_REC.

Faz blend com ranking percentual (ALPHA).

Ajusta impacto de Ticket Médio e NPS.

Classifica em: Baixo / Médio / Alto Risco.

3️⃣ Modelo Supervisionado (opcional)

Gera rótulo de churn se o cliente cancelar até K_PRAZO_MESES após o mês de referência.

Treina Logistic Regression com pipeline:

SimpleImputer → StandardScaler → LogisticRegression


Testa uso de SMOTE para balanceamento.

Busca o threshold ótimo na grade THR_GRID, priorizando Recall via F2-score.

📈 Interpretação dos Resultados
Coluna	Significado
nivel_risco_regra	Risco categórico (Regra)
nivel_risco_ml	Risco categórico (Modelo)
comparativo_dif	Diferença entre risco via ML e via regra
risco_churn_final_regra	Score final de risco (Regra)
risco_ml	Score final de risco (Modelo)

🧩 Use nivel_risco_ml quando o modelo supervisionado estiver ativo.
Caso contrário, utilize nivel_risco_regra.

comparativo_dif = risco_ml - risco_churn_final_regra → mostra divergências entre o modelo e a regra.

🧪 Validação e Métricas

Validação temporal (holdout dos últimos ~3 meses) ou train_test_split (fallback).

Métricas avaliadas:

Acurácia

Precisão

Recall

F1-score

Matriz de confusão

Resultados e métricas são exibidos no console após gerar clientes_risco_final.csv.

✨ Principais Destaques

✅ Série temporal contínua por cliente (preenche meses sem uso)
✅ Cálculo de sinais robustos: tendência, momento de queda, recência e engajamento
✅ Integração de Ticket Médio e NPS
✅ Modelo supervisionado com imputação, padronização, SMOTE e otimização de limiar
✅ Exportação automática de métricas e risco final por cliente

🧾 Exemplo de Saída (clientes_risco_final.csv)
CardCode	risco_churn_final_regra	nivel_risco_regra	risco_ml	nivel_risco_ml	comparativo_dif
C001	0.23	Baixo	0.31	Médio	0.08
C002	0.67	Alto	0.59	Alto	-0.08
C003	0.45	Médio	0.20	Baixo	-0.25
🧑‍💻 Autor

Alisson Silva
📍 Analista de Dados | Projeto: ChurnVision
🔗 GitHub: alisson-silva92/ChurnVision
