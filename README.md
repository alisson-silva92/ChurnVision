🧠 ChurnVision — Predicción de Churn (Reglas + Machine Learning)

Proyecto para estimar el riesgo de churn (cancelación) por cliente, combinando:

1️⃣ Score basado en reglas — recencia, tendencia, momento de caída, ticket medio y NPS
2️⃣ Modelo supervisado (Regresión Logística) — pipeline con umbral optimizado, priorizando Recall / F2-score

🎯 Salida principal: clientes_riesgo_final.csv que contiene el riesgo final de churn por cliente.

🧩 Estructura del Proyecto

prevision_churn/
├─ ChurnVision.ipynb          # Notebook principal
├─ *.xlsx                     # Bases mensuales (año, mes, Conteo, etc.)
├─ cancelados.xlsx            # Histórico de cancelaciones
├─ NPS.xlsx                   # Notas de NPS por cliente
├─ tkt_medio/
│  └─ ticket_medio.csv        # Ticket medio por cliente
└─ clientes_riesgo_final.csv  # Generado al final de la ejecución


⚙️ Parámetros Principales (configurables al inicio del script)

# Ventanas de cálculo
N_RECENT = 3           # ventanas recientes (media reciente)
N_BASE   = 6           # ventana base para comparación
CAP_DIAS = 180         # límite máximo de recencia (días)
GAMMA    = 0.7         # suavización de la normalización (potencia)

# Pesos y ajustes
W_MOMENTO, W_TEND, W_REC = 0.50, 0.35, 0.15
ALPHA = 0.5               # mezcla entre riesgo_raw y ranking porcentual
INFLUENCIA_TICKET = 0.15
W_NPS = 0.15
NEUTRO_NPS = 0.30

# Etiquetado y modelo
K_PRAZO_MESES = 2          # ventana de etiquetado hacia adelante (en meses)
TARGET_RECALL = 0.40       # recall mínimo deseado
THR_GRID = np.linspace(0.20, 0.60, 21)


📄 Estructura Esperada de las Bases

Archivo	Campos	Observaciones
Bases mensuales (.xlsx)	CardCode, año, mes, Conteo, created, Start	Conteo = medida de uso/actividad
ticket_medio.csv	CardCode, TicketMedio	Usado para ponderar el valor del cliente
NPS.xlsx	CardCode, NPS	Renombrado internamente como nota_nps
cancelados.xlsx	CardCode, dt_cancelacion, dt_adquisicion	Se usa la fecha más antigua por cliente

Los campos de texto se limpian con strip y las columnas de fechas/números se convierten con errors='coerce'.

🧠 Lógica de Funcionamiento

1️⃣ Panel Mensual
Crea una serie continua CardCode x mes, rellenando los meses faltantes con Conteo = 0.
Calcula las siguientes señales de comportamiento:

Tendencia (slope relativo)

Momento de caída (media reciente vs base)

Recencia (días sin uso)

Engagement (accesos, días activos, tiempo de uso)

2️⃣ Score por Reglas

Normaliza las señales con MinMaxScaler y aplica la potencia GAMMA.

Combina los pesos W_MOMENTO, W_TEND, W_REC.

Mezcla con ranking porcentual (ALPHA).

Ajusta el impacto del Ticket Medio y del NPS.

Clasifica en: Bajo / Medio / Alto Riesgo.

3️⃣ Modelo Supervisado (opcional)

Genera etiqueta de churn si el cliente cancela dentro de K_PRAZO_MESES posteriores al mes de referencia.

Entrena Logistic Regression con el pipeline:
SimpleImputer → StandardScaler → LogisticRegression

Prueba el uso de SMOTE para balancear clases.

Busca el umbral óptimo en la cuadrícula THR_GRID, priorizando Recall mediante el F2-score.

📈 Interpretación de los Resultados

Columna	Significado
nivel_riesgo_regla	Riesgo categórico (Regla)
nivel_riesgo_ml	Riesgo categórico (Modelo)
comparativo_dif	Diferencia entre riesgo vía ML y vía regla
riesgo_churn_final_regra	Score final de riesgo (Regla)
riesgo_ml	Score final de riesgo (Modelo)

🧩 Usa nivel_riesgo_ml cuando el modelo supervisado esté activo.
De lo contrario, utiliza nivel_riesgo_regla.

comparativo_dif = riesgo_ml - riesgo_churn_final_regra → muestra divergencias entre el modelo y la regla.

🧪 Validación y Métricas

Validación temporal (holdout de los últimos ~3 meses) o train_test_split (alternativo).
Métricas evaluadas:

Exactitud (Accuracy)

Precisión (Precision)

Recall

F1-score

Matriz de confusión

Los resultados y métricas se muestran en consola después de generar clientes_riesgo_final.csv.

✨ Puntos Destacados

✅ Serie temporal continua por cliente (rellena meses sin uso)
✅ Cálculo robusto de señales: tendencia, momento de caída, recencia y engagement
✅ Integración de Ticket Medio y NPS
✅ Modelo supervisado con imputación, estandarización, SMOTE y optimización de umbral
✅ Exportación automática de métricas y riesgo final por cliente

🧾 Ejemplo de Salida (clientes_riesgo_final.csv)

CardCode	riesgo_churn_final_regra	nivel_riesgo_regla	riesgo_ml	nivel_riesgo_ml	comparativo_dif
C001	0.23	Bajo	0.31	Medio	0.08
C002	0.67	Alto	0.59	Alto	-0.08
C003	0.45	Medio	0.20	Bajo	-0.25

🧑‍💻 Autor

Alisson Silva
📍 Analista de Datos | Proyecto: ChurnVision
🔗 GitHub: alisson-silva92/ChurnVision
