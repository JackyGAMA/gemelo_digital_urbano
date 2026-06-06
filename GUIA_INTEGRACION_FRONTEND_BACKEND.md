# 🔗 Guía de Integración Frontend-Backend - FloodMirror AI

## 📋 Resumen

Esta guía explica cómo está configurada la conexión entre el frontend (React/TanStack Start) y el backend (FastAPI) de FloodMirror AI.

## 🏗️ Arquitectura de Conexión

```
┌─────────────────────────────────┐
│   Frontend (React + Vite)      │
│   Puerto: 8080                  │
│   http://localhost:8080         │
└────────────┬────────────────────┘
             │
             │ HTTP Requests
             │ (fetch API)
             │
             ▼
┌─────────────────────────────────┐
│   Backend (FastAPI + Uvicorn)  │
│   Puerto: 8000                  │
│   http://localhost:8000         │
└─────────────────────────────────┘
```

## 📁 Archivos Clave

### 1. Cliente API del Frontend
**Ubicación:** `floodmirror-frontend/src/lib/api/floodmirror.ts`

Este archivo contiene todas las funciones para comunicarse con el backend:

```typescript
// Funciones principales disponibles:
- checkHealth()              // Verificar estado del backend
- predictFloodRisk()         // Predicción con modelo V1
- predictFloodRiskV2()       // Predicción con modelo V2 (XGBoost)
- getModelInfo()             // Información del modelo V1
- getModelV2Info()           // Información del modelo V2
- getDataStats()             // Estadísticas de datos
- getAlcaldiaInfo()          // Info de alcaldía específica
```

### 2. Configuración de Variables de Entorno
**Ubicación:** `floodmirror-frontend/.env`

```env
VITE_API_URL=http://localhost:8000
NODE_ENV=development
```

### 3. Configuración CORS del Backend
**Ubicación:** `floodmirror-ai-backend/app/main.py` (líneas 117-132)

El backend ya está configurado para aceptar conexiones del frontend:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "http://localhost:5173",
        "http://localhost:5174",
        "http://localhost:8080",  # Puerto del frontend actual
        "http://127.0.0.1:8080",
        "*"  # Permite todos en desarrollo
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## 🚀 Cómo Usar la API en el Frontend

### Ejemplo 1: Verificar Conexión con el Backend

```typescript
import { checkHealth } from '@/lib/api/floodmirror';

async function verificarBackend() {
  try {
    const health = await checkHealth();
    console.log('Backend status:', health);
    // { status: "healthy", version: "1.0.0", model_loaded: true, data_loaded: true }
  } catch (error) {
    console.error('Backend no disponible:', error);
  }
}
```

### Ejemplo 2: Hacer una Predicción

```typescript
import { predictFloodRiskV2 } from '@/lib/api/floodmirror';

async function predecirRiesgo(lat: number, lng: number, alcaldia: string) {
  try {
    const resultado = await predictFloodRiskV2({
      alcaldia: alcaldia,
      latitud: lat,
      longitud: lng,
      intensidad: 50,      // mm/hr
      period_ret: 10,      // años
      area_m2: 1000        // m²
    });
    
    console.log('Nivel de riesgo:', resultado.risk_level);
    console.log('Probabilidad:', resultado.flood_probability);
    console.log('Confianza:', resultado.confidence.overall_confidence);
    console.log('Mirror activado:', resultado.mirror_info.activated);
    console.log('Recomendaciones:', resultado.recommendations);
    
    return resultado;
  } catch (error) {
    console.error('Error en predicción:', error);
    throw error;
  }
}
```

### Ejemplo 3: Integración en Componente React

```typescript
import { useState, useEffect } from 'react';
import { checkHealth, predictFloodRiskV2, type FloodPredictionResponse } from '@/lib/api/floodmirror';
import { toast } from 'sonner';

function FloodPredictionComponent() {
  const [backendStatus, setBackendStatus] = useState<'checking' | 'online' | 'offline'>('checking');
  const [prediction, setPrediction] = useState<FloodPredictionResponse | null>(null);
  const [loading, setLoading] = useState(false);

  // Verificar backend al montar
  useEffect(() => {
    checkHealth()
      .then(() => setBackendStatus('online'))
      .catch(() => setBackendStatus('offline'));
  }, []);

  const handlePredict = async (alcaldia: string, lat: number, lng: number) => {
    if (backendStatus !== 'online') {
      toast.error('Backend no disponible');
      return;
    }

    setLoading(true);
    try {
      const result = await predictFloodRiskV2({
        alcaldia,
        latitud: lat,
        longitud: lng,
        intensidad: 50,
        period_ret: 10,
        area_m2: 1000
      });
      
      setPrediction(result);
      toast.success(`Riesgo ${result.risk_level} detectado`);
    } catch (error) {
      toast.error('Error al obtener predicción');
      console.error(error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <div>Backend: {backendStatus}</div>
      {prediction && (
        <div>
          <h3>Resultado:</h3>
          <p>Riesgo: {prediction.risk_level}</p>
          <p>Probabilidad: {(prediction.flood_probability * 100).toFixed(1)}%</p>
          <p>Confianza: {(prediction.confidence.overall_confidence * 100).toFixed(1)}%</p>
        </div>
      )}
    </div>
  );
}
```

## 🔍 Endpoints Disponibles

### Backend API (http://localhost:8000)

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/` | GET | Información básica de la API |
| `/health` | GET | Estado del sistema |
| `/predict` | POST | Predicción con modelo V1 |
| `/predict/v2` | POST | Predicción con modelo V2 (XGBoost) |
| `/model/info` | GET | Info del modelo V1 |
| `/model/v2/info` | GET | Info del modelo V2 |
| `/data/stats` | GET | Estadísticas de datos |
| `/alcaldia/{nombre}` | GET | Info de alcaldía específica |
| `/docs` | GET | Documentación interactiva (Swagger) |

## 🧪 Probar la Conexión

### Opción 1: Desde el Navegador
Abre http://localhost:8000/docs para ver la documentación interactiva y probar los endpoints.

### Opción 2: Desde la Consola del Frontend
Abre la consola del navegador en http://localhost:8080 y ejecuta:

```javascript
// Verificar backend
fetch('http://localhost:8000/health')
  .then(r => r.json())
  .then(console.log);

// Hacer predicción
fetch('http://localhost:8000/predict/v2', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    alcaldia: 'Cuauhtémoc',
    intensidad: 50,
    period_ret: 10,
    area_m2: 1000
  })
})
  .then(r => r.json())
  .then(console.log);
```

### Opción 3: Desde PowerShell

```powershell
# Verificar backend
curl http://localhost:8000/health

# Hacer predicción
curl -X POST http://localhost:8000/predict/v2 `
  -H "Content-Type: application/json" `
  -d '{\"alcaldia\":\"Cuauhtémoc\",\"intensidad\":50,\"period_ret\":10,\"area_m2\":1000}'
```

## 📊 Flujo de Datos Completo

```
1. Usuario interactúa con el mapa
   ↓
2. Frontend detecta ubicación (lat, lng)
   ↓
3. Frontend identifica alcaldía
   ↓
4. Frontend llama a predictFloodRiskV2()
   ↓
5. Petición HTTP POST a /predict/v2
   ↓
6. Backend procesa con XGBoost
   ↓
7. Backend activa Mirror si faltan datos
   ↓
8. Backend calcula confianza
   ↓
9. Backend genera recomendaciones
   ↓
10. Respuesta JSON al frontend
    ↓
11. Frontend actualiza UI con resultados
```

## 🛠️ Próximos Pasos para Integración Completa

Para integrar completamente el backend con el frontend actual:

1. **Modificar `routes/index.tsx`:**
   - Importar funciones de `@/lib/api/floodmirror`
   - Reemplazar datos simulados con llamadas reales a la API
   - Agregar manejo de estados de carga y errores

2. **Actualizar el botón "Estimar riesgo":**
   ```typescript
   import { predictFloodRiskV2 } from '@/lib/api/floodmirror';
   
   const handleEstimate = async () => {
     setLoading(true);
     try {
       const result = await predictFloodRiskV2({
         alcaldia: selectedZone?.name,
         latitud: userPosition?.[0],
         longitud: userPosition?.[1],
         intensidad: 50,
         period_ret: 10,
         area_m2: 1000
       });
       
       // Actualizar UI con resultado real
       setRealRisk(result.risk_level);
       setConfidence(result.confidence.overall_confidence * 100);
       setRecommendations(result.recommendations);
     } catch (error) {
       toast.error('Error al conectar con el backend');
     } finally {
       setLoading(false);
     }
   };
   ```

3. **Agregar indicador de estado del backend:**
   ```typescript
   const [backendOnline, setBackendOnline] = useState(false);
   
   useEffect(() => {
     checkHealth()
       .then(() => setBackendOnline(true))
       .catch(() => setBackendOnline(false));
   }, []);
   ```

## ⚠️ Solución de Problemas

### Error: "Failed to fetch"
- **Causa:** Backend no está corriendo
- **Solución:** Ejecuta `cd floodmirror-ai-backend && uvicorn app.main:app --reload`

### Error: "CORS policy"
- **Causa:** Puerto del frontend no está en la lista de CORS
- **Solución:** Agrega el puerto en `app/main.py` línea 120-127

### Error: "Model not loaded"
- **Causa:** Modelo no está entrenado o archivos de datos faltantes
- **Solución:** Coloca los archivos CSV en `floodmirror-ai-backend/data/`

### Backend responde pero con errores 500
- **Causa:** Datos de entrada inválidos o modelo no cargado
- **Solución:** Verifica los logs del backend en la terminal

## 📚 Recursos Adicionales

- **Documentación API:** http://localhost:8000/docs
- **Backend README:** `floodmirror-ai-backend/README.md`
- **Frontend README:** `floodmirror-frontend/README.md`
- **Quickstart Backend:** `floodmirror-ai-backend/QUICKSTART.md`

## ✅ Checklist de Integración

- [x] Cliente API creado (`floodmirror.ts`)
- [x] Variables de entorno configuradas (`.env`)
- [x] CORS configurado en backend
- [x] Backend corriendo en puerto 8000
- [x] Frontend corriendo en puerto 8080
- [ ] Componentes actualizados para usar API real
- [ ] Manejo de errores implementado
- [ ] Estados de carga agregados
- [ ] Pruebas de integración realizadas

---

**Última actualización:** 2026-06-06
**Versión:** 1.0.0