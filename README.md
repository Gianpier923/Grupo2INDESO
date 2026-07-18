# 📈 Ernesto Investing AI
**Sistema Operacional Integrado: Frontend + Backend (FastAPI) + MongoDB + Autenticación (Supabase) + App Streamlit**

![UNMSM](https://img.shields.io/badge/UNMSM-FISI-blue)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-00a393)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248)
![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-FF6F00)
![scikit-learn](https://img.shields.io/badge/scikit--learn-SVC-F7931E)
[![Streamlit](https://img.shields.io/badge/Streamlit-Cloud-FF4B4B)](https://grupo2indeso-yvhpkj5p7lzsrvlynlkult.streamlit.app/)
![Supabase](https://img.shields.io/badge/Supabase-Auth-3ECF8E)

**🚀 App en vivo:** [grupo2indeso-yvhpkj5p7lzsrvlynlkult.streamlit.app](https://grupo2indeso-yvhpkj5p7lzsrvlynlkult.streamlit.app/)

Repositorio de la implementación final del proyecto **Ernesto Investing AI**, desarrollado para el curso **Introducción al Desarrollo de Software (iDeSo)** — Facultad de Ingeniería de Sistemas e Informática (FISI), Universidad Nacional Mayor de San Marcos (UNMSM).

**Profesor:** Mg. Ing. Ernesto D. Cancho-Rodríguez, MBA (The George Washington University)

## 📋 Descripción del Proyecto

Plataforma completa de soporte a la decisión de inversión (DSS) enfocada en 5 tickers del sector minero (**FSM, VOLCABC1.LM, ABX.TO, BVN, BHP**). El sistema descarga datos reales de Yahoo Finance, calcula indicadores técnicos, entrena y compara múltiples modelos de Machine Learning y Deep Learning (SVC, 4 arquitecturas recurrentes, LSTM Regressor), ejecuta backtesting de estrategias, analiza el sentimiento de noticias con NLP (VADER), y expone todo mediante una API REST y dos frontends independientes: uno en HTML/JS desplegado en GitHub Pages y otro en Streamlit (Python puro) desplegado en Streamlit Cloud.

## 🏗 Arquitectura y Fases del Sistema

### Backend — Notebooks (Google Colab → MongoDB Atlas)

| Notebook | Fase | Entrada | Salida (colecciones MongoDB) |
|---|---|---|---|
| `Notebook1_Ingesta_MongoDB.ipynb` | Ingesta de datos | `yfinance` (5 tickers, 1 año) | `precios_ohlcv` (OHLCV + SMA-20/50, EMA-12/26, RSI-14) |
| `Notebook2_SVC_MongoDB.ipynb` | Clasificador SVC | `precios_ohlcv` | `predicciones` / `metricas_modelos` (modelo=`SVC`) |
| `Notebook3_RNN_MongoDB.ipynb` | Clasificadores recurrentes | `precios_ohlcv` | `predicciones` / `metricas_modelos` (modelo=`LSTM`,`BiLSTM`,`GRU`,`SimpleRNN`) |
| `Notebook4_Backtest_MongoDB.ipynb` | Backtesting (cruce SMA 20/50) | `precios_ohlcv` | `resultados_backtest` (retorno, Sharpe, drawdown, operaciones, curva de equity) |
| `Notebook5_LSTM_Regressor_MongoDB.ipynb` | Pronóstico de precios | `precios_ohlcv` | `predicciones_lstm` / `metricas_lstm` (horizontes 7/14/30/60 días, RMSE vs ARIMA) |
| `Notebook6_Fase3_NLP_VADER_MongoDB.ipynb` | Sentimiento de noticias | Noticias financieras | `sentimiento_nlp` (score VADER, clasificación Bullish/Bearish/Neutral) |
| `Notebook7_FastAPI.ipynb` | API REST | Todas las colecciones anteriores | Servidor FastAPI expuesto vía `ngrok` |

Todos los notebooks comparten el mismo patrón de conexión segura (contraseña de MongoDB solicitada con `getpass`, nunca en texto plano) y usan `delete_many` + `insert_one`/`insert_many` para evitar duplicados al re-ejecutarse.

### API REST (`Notebook7_FastAPI.ipynb`)

Servidor **FastAPI** que **lee** los datos ya procesados en MongoDB (no recalcula nada) y los expone mediante **ngrok**. CORS habilitado (`allow_origins=["*"]`) para que el frontend en GitHub Pages consuma la API con `fetch()`.

| Endpoint | Descripción |
|---|---|
| `GET /api/salud` | Estado del servidor y conteo de documentos de todas las colecciones |
| `GET /api/mercado/{ticker}` | Histórico OHLCV con indicadores técnicos |
| `GET /api/svc/{ticker}` | Señal, confianza y métricas del clasificador SVC |
| `GET /api/rnn/{ticker}` | Predicción y métricas de las 4 arquitecturas recurrentes |
| `GET /api/rnn/{ticker}/{modelo}` | Predicción y métricas de una arquitectura RNN específica |
| `GET /api/backtest/{ticker}` | Métricas, operaciones y curva de equity del backtest SMA 20/50 |
| `GET /api/lstm-regressor/{ticker}` | Pronóstico de precio a 7/14/30/60 días con bandas de confianza |
| `GET /api/sentimiento/{ticker}` | Análisis de sentimiento VADER de noticias recientes |
| `GET /api/resumen/{ticker}` | Endpoint de conveniencia: combina todos los módulos anteriores para un ticker |

### Frontend Web — HTML5 + Tailwind CSS + JavaScript

Interfaz multipágina desplegada en **GitHub Pages**, con autenticación de usuarios (Supabase Auth) y módulos analíticos que consumen la API vía `fetch()`.

| Archivo | Módulo | Endpoint(s) consumido(s) |
|---|---|---|
| `index.html` | Autenticación | Login, registro y recuperación de contraseña (Supabase Auth) |
| `home.html` | Portal / Configuración | `GET /api/salud` — configura y valida la URL de ngrok |
| `modulo_mercado.html` | Dashboard de Mercado | `/api/mercado/{ticker}` |
| `modulo_svc.html` | Clasificador SVC | `/api/svc/{ticker}` |
| `investai_modelos_ia.html` | Modelos RNN | `/api/rnn/{ticker}` |
| `investai_lstm.html` | Pronóstico LSTM Regressor | `/api/lstm-regressor/{ticker}` |
| `investai_nlp.html` | Sentimiento de noticias | `/api/sentimiento/{ticker}` |
| `reportes_backtesting.html` | Reportes de Backtesting | `/api/backtest/{ticker}` |
| `estrategias.html` | Estrategias de Inversión | `/api/resumen/{ticker}` |
| `portafolio.html` | Gestión de Portafolio | Seguimiento de posiciones del usuario |
| `broker.html` | Ejecución de Órdenes (Broker) | Simulación de compra/venta |
| `dashboard_completo.html` | Dashboard Completo | `/api/resumen/{ticker}` — vista consolidada |

> Todos los módulos (excepto `index.html`) obtienen la URL de la API mediante `sessionStorage.getItem('API_URL')`, configurada una única vez desde `home.html`.

### Bono Streamlit (`app.py`) — Frontend alternativo 100 % Python

Aplicación independiente en **Streamlit**, desplegada en **Streamlit Cloud** con URL permanente (no depende de que Colab/ngrok estén activos). Lee directamente de MongoDB Atlas usando `st.secrets["MONGO_URI"]`, sin pasar por la API FastAPI.

| Tab | Colecciones que lee | Equivalente en el frontend HTML |
|---|---|---|
| 📊 Market Dashboard | `precios_ohlcv` | `modulo_mercado.html` |
| 🤖 SVC Classifier | `predicciones` / `metricas_modelos` (SVC) | `modulo_svc.html` |
| 🔁 RNN (4 arquitecturas) | `predicciones` / `metricas_modelos` (LSTM, BiLSTM, GRU, SimpleRNN) | `investai_modelos_ia.html` |
| 📈 LSTM Regressor | `predicciones_lstm` / `metricas_lstm` | `investai_lstm.html` |
| 💰 Backtesting | `resultados_backtest` | `reportes_backtesting.html` |
| 📰 NLP / VADER | `sentimiento_nlp` | `investai_nlp.html` |

**URL de la app:** [https://grupo2indeso-yvhpkj5p7lzsrvlynlkult.streamlit.app/](https://grupo2indeso-yvhpkj5p7lzsrvlynlkult.streamlit.app/)

## 🚀 Instrucciones de Despliegue

Sigue estos pasos estrictamente en orden:

### 1. Backend — Notebooks en Google Colab

Ejecutar en este orden (cada uno depende de las colecciones pobladas por el anterior):

1. `Notebook1_Ingesta_MongoDB.ipynb` — ingresar la contraseña de MongoDB Atlas (`getpass`) y ejecutar todo. Puebla `precios_ohlcv`.
2. `Notebook2_SVC_MongoDB.ipynb` — entrena el SVC. Puebla `predicciones` / `metricas_modelos` (SVC).
3. `Notebook3_RNN_MongoDB.ipynb` — entrena las 4 arquitecturas recurrentes (puede tardar varios minutos). Puebla `predicciones` / `metricas_modelos` (RNN).
4. `Notebook4_Backtest_MongoDB.ipynb` — corre el backtest SMA 20/50. Puebla `resultados_backtest`.
5. `Notebook5_LSTM_Regressor_MongoDB.ipynb` — entrena el LSTM Regressor y genera pronósticos. Puebla `predicciones_lstm` / `metricas_lstm`.
6. `Notebook6_Fase3_NLP_VADER_MongoDB.ipynb` — analiza noticias con VADER. Puebla `sentimiento_nlp`.
7. `Notebook7_FastAPI.ipynb` — configurar el *Secret* `NGROK_AUTHTOKEN` en Colab, ingresar la contraseña de MongoDB y ejecutar todo. Copiar la **URL pública de ngrok** que se imprime al final.

### 2. Frontend HTML — GitHub Pages

1. Subir los archivos `.html` al repositorio y activar **GitHub Pages** (Settings → Pages).
2. Ingresar por `index.html` (iniciar sesión o registrarse con Supabase).
3. En `home.html`, pegar la **URL de ngrok** obtenida del backend y verificar la conexión (`GET /api/salud`).
4. Navegar hacia los módulos para visualizar los tableros con datos reales.

*(Nota: al usar el nivel gratuito de ngrok, la primera visita puede mostrar una advertencia en el navegador. Las llamadas `fetch()` incluyen el header `ngrok-skip-browser-warning` para evitarlo.)*

### 3. Frontend alternativo — Streamlit Cloud

1. Crear la app en [share.streamlit.io](https://share.streamlit.io) apuntando a este repositorio, rama `main`, archivo principal `app.py`.
2. En **Advanced settings → Secrets**, configurar:
   ```toml
   MONGO_URI = "mongodb+srv://<usuario>:<password>@cluster0.sxjck8h.mongodb.net/?appName=Cluster0"
   ```
3. Desplegar. La URL resultante es permanente y no depende de Colab/ngrok.

### Ejecución local (opcional, para desarrollo)

```bash
git clone https://github.com/eq2ideso26i-prog/Grupo2INDESO.git
cd Grupo2INDESO
pip install -r requirements.txt
streamlit run app.py
```

Se requiere crear un archivo `.streamlit/secrets.toml` local con el mismo `MONGO_URI` indicado arriba (este archivo **no** debe subirse al repositorio).

## 🛠 Tecnologías Utilizadas
* **Backend / API:** Python, FastAPI, uvicorn, pyngrok
* **Machine Learning:** scikit-learn (SVC + GridSearchCV)
* **Deep Learning:** TensorFlow / Keras (LSTM, BiLSTM, GRU, SimpleRNN), LSTM Regressor
* **NLP:** VADER Sentiment Analysis
* **Datos / Backtesting:** yfinance, pandas, numpy
* **Base de Datos:** MongoDB Atlas, pymongo
* **Autenticación:** Supabase Auth
* **Infraestructura:** Google Colab, ngrok, Streamlit Cloud, GitHub Pages
* **Frontend Web:** HTML5, Tailwind CSS, Vanilla JavaScript (Fetch API), Lucide Icons, Plotly.js, Chart.js
* **Frontend alternativo:** Streamlit, Plotly (Python)

## 📂 Estructura del Repositorio
```
├── notebooks/
│   ├── Notebook1_Ingesta_MongoDB.ipynb        # Fase 1 — yfinance → indicadores → MongoDB
│   ├── Notebook2_SVC_MongoDB.ipynb            # Fase 1 — MongoDB → SVC + GridSearchCV → MongoDB
│   ├── Notebook3_RNN_MongoDB.ipynb            # Fase 1 — MongoDB → LSTM/BiLSTM/GRU/SimpleRNN → MongoDB
│   ├── Notebook4_Backtest_MongoDB.ipynb       # Fase 1 — MongoDB → Backtest SMA 20/50 → MongoDB
│   ├── Notebook5_LSTM_Regressor_MongoDB.ipynb # Fase 1 — MongoDB → Pronostico LSTM → MongoDB
│   ├── Notebook6_Fase3_NLP_VADER_MongoDB.ipynb# Fase 3 — Noticias → VADER → MongoDB
│   └── Notebook7_FastAPI.ipynb                # Fase 2 — MongoDB → FastAPI → ngrok
├── frontend/
│   ├── index.html                             # Autenticacion (Supabase)
│   ├── home.html                              # Portal / configuracion de API_URL
│   ├── modulo_mercado.html                    # Dashboard de mercado
│   ├── modulo_svc.html                        # Clasificador SVC
│   ├── investai_modelos_ia.html                # Modelos RNN
│   ├── investai_lstm.html                     # Pronostico LSTM Regressor
│   ├── investai_nlp.html                      # Sentimiento de noticias
│   ├── reportes_backtesting.html              # Reportes de backtesting
│   ├── estrategias.html                       # Estrategias de inversion
│   ├── portafolio.html                        # Gestion de portafolio
│   ├── broker.html                            # Ejecucion de ordenes (broker)
│   └── dashboard_completo.html                # Dashboard consolidado
├── app.py                                     # Bono Streamlit (6 tabs, lee MongoDB directo)
├── requirements.txt                           # Dependencias de la app Streamlit
└── README.md
```

## 📄 Licencia
Proyecto académico elaborado para el curso iDeSo - UNMSM FISI. Sin fines comerciales.
