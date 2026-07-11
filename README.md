[README.md](https://github.com/user-attachments/files/29917159/README.md)
# 📈 Ernesto Investing AI
**Sistema Operacional Integrado: Frontend + Backend (FastAPI) + MongoDB + Autenticación (Supabase)**

![UNMSM](https://img.shields.io/badge/UNMSM-FISI-blue)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-00a393)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248)
![scikit-learn](https://img.shields.io/badge/scikit--learn-SVC-F7931E)
![Supabase](https://img.shields.io/badge/Supabase-Auth-3ECF8E)

Repositorio de la implementación final del proyecto **Ernesto Investing AI**, desarrollado para la Semana 13 del curso **Introducción al Desarrollo de Software (iDeSo)** de la Facultad de Ingeniería de Sistemas e Informática (FISI) de la Universidad Nacional Mayor de San Marcos (UNMSM).

**Profesor:** Mg. Ing. Ernesto D. Cancho-Rodríguez, MBA

## 📋 Descripción del Proyecto

Plataforma completa de análisis de mercado financiero enfocada en 5 tickers del sector minero (**FSM, VOLCABC1.LM, ABX.TO, BVN, BHP**). El sistema descarga datos reales, calcula indicadores técnicos (SMA, EMA, RSI), entrena un modelo predictivo de Machine Learning (SVC) para generar señales de compra/venta (BUY/SELL), y expone todo a través de una API RESTful consumida por un frontend interactivo con múltiples módulos analíticos y autenticación de usuarios.

## 🏗 Arquitectura y Fases del Sistema

### 1. Ingesta de Datos y Machine Learning (`Notebook1_Ingesta_MongoDB.ipynb`, `Notebook2_SVC_MongoDB.ipynb`)
* **Notebook 1 — Ingesta de Datos:** descarga OHLCV con `yfinance` para los 5 tickers mineros, calcula indicadores técnicos (SMA-20, SMA-50, EMA-12, EMA-26, RSI-14) y los guarda en la colección `precios_ohlcv` de **MongoDB Atlas**.
* **Notebook 2 — Clasificador SVC:** lee `precios_ohlcv`, construye features y un target BUY/SELL sin *data leakage* (partición temporal 80/20), entrena un `SVC` optimizado con `GridSearchCV` y guarda señal, confianza y métricas (accuracy, precision, recall, F1, matriz de confusión) en las colecciones `predicciones` y `metricas_modelos`.

### 2. Backend y API REST (`Notebook3_API_FastAPI.ipynb`)
Levanta un servidor **FastAPI** en Google Colab que **lee** los datos ya procesados en MongoDB (no recalcula nada) y los expone a Internet mediante **ngrok**.

* **Endpoints:**
  * `GET /api/salud` — estado del servidor y de la base de datos.
  * `GET /api/mercado/{ticker}` — histórico OHLCV con indicadores técnicos.
  * `GET /api/svc/{ticker}` — señal, confianza y métricas del clasificador SVC.
* CORS habilitado (`allow_origins=["*"]`) para que el frontend en GitHub Pages pueda consumir la API con `fetch()`.

### 3. Frontend Web (`HTML5 + Tailwind CSS + JavaScript`)
Interfaz multipágina lista para desplegar en **GitHub Pages**, con autenticación de usuarios y varios módulos analíticos que consumen la API vía `fetch()`.

| Archivo | Módulo | Descripción |
|---|---|---|
| `index.html` | Autenticación | Login, registro y recuperación de contraseña vía **Supabase Auth**. Punto de entrada del sistema. |
| `home.html` | Portal / Configuración | Hub de navegación hacia los módulos. Permite ingresar y verificar la URL pública de ngrok (`GET /api/salud`), guardada en `sessionStorage` para toda la sesión. |
| `modulo_mercado.html` | Dashboard de Mercado | Precios OHLCV e indicadores técnicos por ticker (`/api/mercado/{ticker}`). |
| `modulo_svc.html` | Clasificador SVC | Señales BUY/SELL, confianza y métricas del modelo (`/api/svc/{ticker}`). |
| `estrategias.html` | Estrategias de Inversión | Análisis y comparación de estrategias basadas en las señales del modelo. |
| `portafolio.html` | Gestión de Portafolio | Seguimiento de posiciones y composición del portafolio del usuario. |
| `reportes_backtesting.html` | Reportes de Backtesting | Evaluación histórica del desempeño de las señales generadas. |
| `broker.html` | Ejecución de Órdenes (Broker) | Simulación de envío de órdenes de compra/venta. |
| `dashboard_completo.html` | Dashboard Completo | Vista consolidada de mercado, señales y portafolio en una sola pantalla. |

> Todos los módulos (excepto `index.html`) obtienen la URL de la API mediante `sessionStorage.getItem('API_URL')`, configurada una única vez desde `home.html`.

## 🚀 Instrucciones de Despliegue

Sigue estos pasos estrictamente en orden:

### Backend (Google Colab + MongoDB)
1. Abrir **`Notebook1_Ingesta_MongoDB.ipynb`** en Google Colab, ingresar la contraseña de MongoDB Atlas cuando se solicite (`getpass`) y ejecutar todo para poblar `precios_ohlcv`.
2. Abrir **`Notebook2_SVC_MongoDB.ipynb`** y ejecutar todo para entrenar el modelo SVC y guardar `predicciones` y `metricas_modelos`.
3. Abrir **`Notebook3_API_FastAPI.ipynb`**, configurar el *Secret* `NGROK_AUTHTOKEN` en Colab, ingresar la contraseña de MongoDB y ejecutar todo. Al final, **copia la URL pública de ngrok** que se imprime en el Paso 6.

### Frontend
1. Sube los archivos `.html` a **GitHub Pages** (o ábrelos localmente).
2. Ingresa por `index.html` (inicia sesión o regístrate con Supabase).
3. En `home.html`, pega la **URL de ngrok** obtenida del backend y verifica la conexión (`Sistema en línea`).
4. Navega hacia los módulos (`modulo_mercado.html`, `modulo_svc.html`, `dashboard_completo.html`, etc.) para visualizar los tableros con datos reales.

*(Nota: al usar el nivel gratuito de ngrok, la primera visita puede mostrar una advertencia en el navegador. Todas las llamadas `fetch()` del frontend incluyen el header `ngrok-skip-browser-warning` para evitar bloqueos.)*

## 🛠 Tecnologías Utilizadas
* **Backend:** Python, FastAPI, uvicorn, pyngrok
* **Machine Learning / Data:** scikit-learn (SVC + GridSearchCV), yfinance, pandas, numpy
* **Base de Datos:** MongoDB Atlas, pymongo
* **Autenticación:** Supabase Auth
* **Infraestructura:** Google Colab, ngrok
* **Frontend:** HTML5, Tailwind CSS, Vanilla JavaScript (Fetch API), Lucide Icons

## 📂 Estructura del Repositorio
```
├── Notebook1_Ingesta_MongoDB.ipynb      # Fase 1 — yfinance → indicadores → MongoDB
├── Notebook2_SVC_MongoDB.ipynb          # Fase 1 — MongoDB → SVC + GridSearchCV → MongoDB
├── Notebook3_API_FastAPI.ipynb          # Fase 2 — MongoDB → FastAPI → ngrok
├── index.html                           # Autenticación (Supabase)
├── home.html                            # Portal / configuración de API_URL
├── modulo_mercado.html                  # Dashboard de mercado
├── modulo_svc.html                      # Clasificador SVC
├── estrategias.html                     # Estrategias de inversión
├── portafolio.html                      # Gestión de portafolio
├── reportes_backtesting.html            # Reportes de backtesting
├── broker.html                          # Ejecución de órdenes (broker)
├── dashboard_completo.html              # Dashboard consolidado
└── README.md
```

## 📄 Licencia
Proyecto académico elaborado para el curso iDeSo - UNMSM FISI. Sin fines comerciales.
