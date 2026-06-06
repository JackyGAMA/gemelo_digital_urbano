# FloodMirror AI 🌊

**Sistema Inteligente de Predicción de Riesgo de Inundaciones para la Ciudad de México**

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104+-green.svg)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18+-61DAFB.svg)](https://reactjs.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 📋 Descripción

FloodMirror AI es un sistema de predicción de riesgo de inundaciones que combina:

- **Machine Learning Avanzado**: Modelo XGBoost con 99.96% de precisión (R²=0.9996)
- **Integración Meteorológica en Tiempo Real**: API Open-Meteo para datos actuales
- **Sistema de Alertas Inteligentes**: Combina riesgo histórico + clima actual
- **Datos Oficiales**: Atlas de Riesgo de Precipitación CDMX, sitios de encharcamiento, datos AGEB
- **IA Explicable**: Transparencia total en las predicciones
- **Privacidad por Diseño**: Cumplimiento LFPDPPP

## 🏗️ Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (React/TypeScript)               │
│  - Dashboard interactivo                                     │
│  - Mapa de riesgo (Leaflet)                                 │
│  - Visualización de alertas                                 │
│  - Gestión de privacidad (ARCO)                             │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP/REST API
┌────────────────────▼────────────────────────────────────────┐
│                    BACKEND (FastAPI)                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Endpoint /predict/v2                                 │  │
│  │  - Recibe ubicación + parámetros                     │  │
│  │  - Integra datos meteorológicos                      │  │
│  │  - Genera predicción + alertas                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  Weather    │  │  Alert       │  │  Model V2       │  │
│  │  Service    │  │  System      │  │  (XGBoost)      │  │
│  │  (Open-     │  │  (Intelligent│  │  R²=0.9996      │  │
│  │  Meteo)     │  │  Alerts)     │  │                 │  │
│  └─────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    DATOS OFICIALES                           │
│  - Atlas de Riesgo de Precipitación CDMX                    │
│  - Sitios recurrentes de encharcamiento                     │
│  - Características de viviendas por AGEB                    │
│  - Datos demográficos                                       │
└─────────────────────────────────────────────────────────────┘
```

## 🚀 Características Principales

### 1. Predicción de Riesgo con IA
- **Modelo XGBoost V2**: Entrenado con datos oficiales de CDMX
- **Precisión**: R² = 0.9996 (99.96% de precisión)
- **Características**: Intensidad de lluvia, periodo de retorno, área afectada, datos demográficos

### 2. Integración Meteorológica en Tiempo Real
- **API**: Open-Meteo (https://api.open-meteo.com)
- **Datos**: Lluvia actual, probabilidad de lluvia, precipitación esperada
- **Actualización**: Tiempo real

### 3. Sistema de Alertas Inteligentes
Combina tres factores de riesgo:
- **Riesgo Histórico** (40%): Basado en datos históricos de inundaciones
- **Riesgo Meteorológico** (35%): Condiciones climáticas actuales
- **Predicción ML** (25%): Modelo de machine learning

**Niveles de Alerta**:
- 🟢 **BAJO**: Condiciones favorables
- 🟡 **MODERADO**: Monitoreo recomendado
- 🟠 **ALTO**: Precaución necesaria
- 🔴 **CRÍTICO**: Acción inmediata requerida

### 4. Datos Oficiales
- **Atlas de Riesgo de Precipitación CDMX**: Datos históricos de precipitación
- **Sitios de Encharcamiento**: Ubicaciones con historial de inundaciones
- **Datos AGEB**: Características demográficas y de vivienda
- **Sin datos sintéticos**: 100% datos reales

### 5. Privacidad y Cumplimiento Legal
- **LFPDPPP**: Cumplimiento total con la ley mexicana
- **Derechos ARCO**: Acceso, Rectificación, Cancelación, Oposición
- **Procesamiento Local**: Datos procesados en el navegador
- **Sin almacenamiento permanente**: Privacidad por diseño

## 📦 Instalación

### Requisitos Previos
- Python 3.9+
- Node.js 18+
- Bun (opcional, recomendado para frontend)

### Backend (FastAPI)

```bash
cd floodmirror-ai-backend

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Configurar variables de entorno
cp .env.example .env

# Entrenar modelo (si no existe)
python scripts/04_model_training.py

# Iniciar servidor
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend (React)

```bash
cd floodmirror-frontend

# Instalar dependencias
bun install  # o npm install

# Configurar API URL
echo "VITE_API_URL=http://localhost:8000" > .env

# Iniciar desarrollo
bun run dev  # o npm run dev
```

## 🎯 Uso

### API Endpoints

#### 1. Predicción de Riesgo (Principal)
```http
POST /predict/v2
Content-Type: application/json

{
  "alcaldia": "Iztapalapa",
  "intensidad": 65.5,
  "period_ret": 10,
  "area_m2": 5000,
  "latitud": 19.3573,
  "longitud": -99.0554
}
```

**Respuesta**:
```json
{
  "risk_level": "high",
  "flood_probability": 0.78,
  "confidence": {
    "overall_confidence": 0.96,
    "data_completeness": 1.0,
    "model_confidence": 0.95
  },
  "explanation": {
    "main_factors": [
      "Riesgo histórico: 75.2%",
      "Riesgo meteorológico: 82.1%",
      "Lluvia actual: 12.5 mm/h"
    ],
    "interpretation": "⚠️ ALERTA ALTA en Iztapalapa: Riesgo significativo..."
  },
  "recommendations": [
    {
      "category": "alert",
      "priority": "high",
      "action": "Preparar kit de emergencia",
      "description": "Agua para 3 días, linterna, radio..."
    }
  ],
  "location_info": {
    "alcaldia": "Iztapalapa",
    "alert_level": "high",
    "weather_integrated": true,
    "weather_info": {
      "current_rain": 12.5,
      "rain_probability": 85.0,
      "weather_risk_factor": 0.821
    }
  }
}
```

#### 2. Health Check
```http
GET /health
```

#### 3. Información del Modelo
```http
GET /model/info
```

#### 4. Estadísticas de Datos
```http
GET /data/stats
```

### Frontend

1. **Abrir Dashboard**: http://localhost:5173
2. **Aceptar Aviso de Privacidad**
3. **Detectar Ubicación** o **Ingresar Dirección**
4. **Ver Predicción** con alertas y recomendaciones
5. **Explorar Mapa** de riesgo interactivo

## 📊 Datasets

### Datos Utilizados

1. **Atlas de Riesgo de Precipitación CDMX**
   - Archivo: `Atlas_de_Riesgo_de_Precipitacion_CDMX.csv`
   - Registros: 4,908
   - Características: Intensidad, periodo de retorno, área

2. **Sitios Recurrentes de Encharcamiento**
   - Archivos: `Sitios_recurrentes_de_encharcamiento.shp` (+ .dbf, .prj, .shx)
   - Formato: Shapefile
   - Ubicaciones con historial de inundaciones

3. **Características de Viviendas por AGEB**
   - Archivos: `Caracteristicas_de_viviendas_por_AGEB.shp` (+ .dbf, .prj, .shx)
   - Formato: Shapefile
   - Datos demográficos y de vivienda

### Datos Procesados

- `data/cleaned/atlas_precipitacion_clean.csv`
- `data/cleaned/encharcamiento_clean.geojson`
- `data/cleaned/viviendas_ageb_clean.geojson`
- `data/features/flood_model_ready.csv`

## 🧠 Modelo de Machine Learning

### Modelo V2 (XGBoost)

**Características**:
- Algoritmo: XGBoost Regressor
- Precisión: R² = 0.9996
- Características de entrada: 10
- Entrenamiento: 4,908 muestras

**Features**:
1. `intensidad_nivel`: Nivel de intensidad de lluvia (1-5)
2. `periodo_retorno_num`: Periodo de retorno en años
3. `intensidad_normalizada`: Intensidad normalizada (0-1)
4. `area_km2`: Área afectada en km²
5. `sitios_encharcamiento_cercanos`: Número de sitios cercanos
6. `tiene_encharcamiento_cercano`: Indicador binario
7. `densidad_poblacional`: Densidad de población
8. `vivtot`: Total de viviendas
9. `tvivpar`: Viviendas particulares
10. `vvpr_hb`: Viviendas con piso de tierra

**Entrenamiento**:
```bash
python scripts/04_model_training.py
```

## 🌐 Integración Meteorológica

### Open-Meteo API

**Endpoint**: https://api.open-meteo.com/v1/forecast

**Parámetros**:
- `latitude`, `longitude`: Coordenadas
- `current`: Temperatura, precipitación, código meteorológico
- `hourly`: Probabilidad de lluvia, precipitación
- `timezone`: America/Mexico_City

**Cálculo de Riesgo Meteorológico**:
```python
weather_risk = (
    current_rain_intensity * 0.4 +
    rain_probability * 0.3 +
    expected_precipitation * 0.3
)
```

## 🔒 Privacidad y Seguridad

### Cumplimiento LFPDPPP

1. **Consentimiento Explícito**: Banner de privacidad obligatorio
2. **Minimización de Datos**: Solo datos necesarios
3. **Procesamiento Local**: Datos en memoria del navegador
4. **Sin Almacenamiento Permanente**: No hay base de datos de usuarios
5. **Derechos ARCO**: Implementados en el frontend
   - **Acceso**: Ver datos almacenados
   - **Rectificación**: Editar ubicación
   - **Cancelación**: Eliminar datos
   - **Oposición**: No usar el servicio

### Seguridad

- **HTTPS**: Recomendado en producción
- **CORS**: Configurado para orígenes permitidos
- **Validación**: Pydantic schemas en backend
- **Sanitización**: Inputs validados

## 📁 Estructura del Proyecto

```
Desktop/
├── floodmirror-ai-backend/          # Backend FastAPI
│   ├── app/
│   │   ├── main.py                  # Aplicación principal con weather + alerts
│   │   ├── models/
│   │   │   └── schemas.py           # Modelos Pydantic
│   │   ├── ml/
│   │   │   ├── model_v2.py          # Modelo XGBoost
│   │   │   └── mirror.py            # Módulo Mirror
│   │   ├── services/
│   │   │   ├── weather.py           # ✨ Integración Open-Meteo
│   │   │   ├── alerts.py            # ✨ Sistema de alertas inteligentes
│   │   │   ├── confidence.py        # Cálculo de confianza
│   │   │   ├── explainer.py         # Explicaciones
│   │   │   └── recommendations.py   # Recomendaciones
│   │   └── utils/
│   │       └── data_loader.py       # Carga de datos
│   ├── data/                        # Datos oficiales
│   │   ├── Atlas_de_Riesgo_de_Precipitacion_CDMX.csv
│   │   ├── Sitios_recurrentes_de_encharcamiento.*
│   │   ├── Caracteristicas_de_viviendas_por_AGEB.*
│   │   └── cleaned/                 # Datos procesados
│   ├── models/                      # Modelos entrenados
│   │   ├── best_model.pkl           # Modelo XGBoost V2
│   │   └── model_metadata.json      # Metadatos
│   ├── scripts/                     # Scripts de procesamiento
│   │   ├── 02_data_cleaning.py
│   │   ├── 03_feature_engineering.py
│   │   └── 04_model_training.py
│   └── requirements.txt             # Dependencias Python
│
├── floodmirror-frontend/            # Frontend React
│   ├── src/
│   │   ├── routes/
│   │   │   └── index.tsx            # Dashboard principal
│   │   ├── components/
│   │   │   ├── FloodMap.tsx         # Mapa interactivo
│   │   │   └── RecommendationModal.tsx
│   │   └── lib/
│   │       └── api/
│   │           └── floodmirror.ts   # Cliente API
│   └── package.json
│
└── README.md                        # Este archivo
```

## 🧪 Testing

### Backend
```bash
# Test de API
python test_api.py

# Test de integración
python test_integration.py
```

### Frontend
```bash
# Ejecutar tests
bun test  # o npm test
```

## 🚢 Despliegue

### Backend (Producción)

```bash
# Instalar dependencias
pip install -r requirements.txt

# Configurar variables de entorno
export API_HOST=0.0.0.0
export API_PORT=8000

# Ejecutar con Gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

### Frontend (Producción)

```bash
# Build
bun run build  # o npm run build

# Servir archivos estáticos
# Los archivos estarán en dist/
```

### Docker (Opcional)

```dockerfile
# Backend Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 📈 Roadmap

- [x] Modelo XGBoost V2 con datos oficiales
- [x] Integración meteorológica Open-Meteo
- [x] Sistema de alertas inteligentes
- [x] Frontend con mapa interactivo
- [x] Cumplimiento LFPDPPP
- [ ] Notificaciones push
- [ ] App móvil (React Native)
- [ ] Integración con más fuentes de datos
- [ ] API pública para desarrolladores
- [ ] Dashboard para autoridades

## 🤝 Contribuciones

Este proyecto fue desarrollado para CONCIENCIA 2026.

## 📄 Licencia

MIT License - Ver archivo LICENSE para detalles

## 👥 Equipo

Desarrollado con ❤️ para la Ciudad de México

## 📞 Contacto

- **Email**: floodmirror@example.com
- **GitHub**: https://github.com/yourusername/floodmirror-ai

## 🙏 Agradecimientos

- **Gobierno de la Ciudad de México**: Por los datos oficiales
- **Open-Meteo**: Por la API meteorológica gratuita
- **Comunidad Open Source**: Por las herramientas utilizadas

---

**FloodMirror AI** - Protegiendo a la Ciudad de México con Inteligencia Artificial 🌊🤖