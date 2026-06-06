# 🔗 Guía de Integración Frontend-Backend FloodMirror AI

## 📍 Ubicación de los Proyectos

```
C:\Users\jacqu\Desktop\
├── floodmirror-ai-backend\     ← Backend FastAPI
└── CONCIENCIA-2026\            ← Frontend React
```

## 🚀 Paso 1: Iniciar el Backend

### 1.1 Abrir Terminal para Backend

```bash
cd C:\Users\jacqu\Desktop\floodmirror-ai-backend
```

### 1.2 Activar Entorno Virtual

```bash
venv\Scripts\activate
```

### 1.3 Generar Datos de Prueba (si no existen)

```bash
python generate_sample_data.py
```

### 1.4 Iniciar Servidor Backend

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

✅ **Backend corriendo en:** `http://localhost:8000`
📚 **Documentación API:** `http://localhost:8000/docs`

---

## 🎨 Paso 2: Iniciar el Frontend

### 2.1 Abrir NUEVA Terminal para Frontend

```bash
cd C:\Users\jacqu\Desktop\CONCIENCIA-2026
```

### 2.2 Instalar Dependencias (primera vez)

```bash
npm install
```

O si usas Bun:
```bash
bun install
```

### 2.3 Iniciar Servidor Frontend

```bash
npm run dev
```

O con Bun:
```bash
bun run dev
```

✅ **Frontend corriendo en:** `http://localhost:5173` (o el puerto que indique)

---

## 🔌 Paso 3: Verificar la Conexión

### 3.1 Probar Backend Directamente

Abre tu navegador y ve a:
```
http://localhost:8000/health
```

Deberías ver:
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "model_loaded": true,
  "data_loaded": true
}
```

### 3.2 Probar desde el Frontend

1. Abre el frontend: `http://localhost:5173`
2. Acepta el aviso de privacidad
3. Marca el checkbox de consentimiento
4. Haz clic en "Detectar mi ubicación automática" o ingresa una dirección
5. Haz clic en "Estimar riesgo"

---

## 📝 Archivos Creados para la Integración

### Backend (ya configurado):
- ✅ CORS habilitado para `localhost:5173`, `localhost:3000`, etc.
- ✅ Endpoint `/predict` listo para recibir datos del frontend

### Frontend (archivos nuevos):
- ✅ `src/lib/api/floodmirror.ts` - Cliente API para conectar con backend
- ✅ `.env.local` - Configuración de URL del backend

---

## 🔧 Cómo Usar la API desde el Frontend

### Ejemplo de Uso en el Código del Frontend

```typescript
import { predictFloodRisk, mapRiskLevel } from "@/lib/api/floodmirror";

// Hacer predicción
const response = await predictFloodRisk({
  alcaldia: "Iztapalapa",
  intensidad: 45.5,
  area_m2: 5000.0,
  latitud: 19.3573,
  longitud: -99.0699
});

// Usar la respuesta
console.log("Riesgo:", response.risk_level);
console.log("Probabilidad:", response.flood_probability);
console.log("Confianza:", response.confidence.overall_confidence);
console.log("Mirror activado:", response.mirror_info.activated);
console.log("Explicación:", response.explanation.interpretation);
console.log("Recomendaciones:", response.recommendations);

// Mapear al formato del frontend
const frontendRisk = mapRiskLevel(response.risk_level);
```

---

## 🎯 Integrar en el Dashboard Existente

### Modificar `src/routes/index.tsx`

Reemplaza la lógica simulada con llamadas reales al backend:

```typescript
// Importar al inicio del archivo
import { predictFloodRisk, mapRiskLevel } from "@/lib/api/floodmirror";

// En la función handleDetectLocation (línea ~137):
navigator.geolocation.getCurrentPosition(
  async (pos) => {
    const lat = pos.coords.latitude;
    const lng = pos.coords.longitude;
    setUserPosition([lat, lng]);
    
    // NUEVO: Llamar al backend real
    try {
      const prediction = await predictFloodRisk({
        latitud: lat,
        longitud: lng,
        alcaldia: zone?.name // si tienes la alcaldía
      });
      
      const frontendRisk = mapRiskLevel(prediction.risk_level);
      const zone = await resolveAlcaldia(lat, lng);
      
      if (zone) {
        // Actualizar con el riesgo real del backend
        setSelectedZone({ ...zone, risk: frontendRisk });
        toast.success(`Riesgo ${prediction.risk_level}: ${prediction.flood_probability * 100}%`);
      }
    } catch (error) {
      console.error("Error al predecir:", error);
      toast.error("Error al conectar con el backend");
    }
    
    setLocating(false);
  },
  // ... resto del código
);
```

### En el botón "Estimar riesgo" (línea ~268):

```typescript
onClick={async () => {
  setLastAddress(address);
  
  // NUEVO: Llamar al backend
  try {
    const prediction = await predictFloodRisk({
      alcaldia: address // o extraer alcaldía del address
    });
    
    const frontendRisk = mapRiskLevel(prediction.risk_level);
    
    setSelectedZone({ 
      id: address, 
      name: address, 
      risk: frontendRisk 
    });
    
    toast.success(
      `Riesgo ${prediction.risk_level} (${(prediction.flood_probability * 100).toFixed(1)}%) - Confianza: ${(prediction.confidence.overall_confidence * 100).toFixed(0)}%`
    );
  } catch (error) {
    console.error("Error:", error);
    toast.error("Error al estimar riesgo");
  }
}}
```

---

## 🐛 Solución de Problemas

### Error: "Failed to fetch" o "Network Error"

**Causa:** Backend no está corriendo o CORS mal configurado

**Solución:**
1. Verifica que el backend esté corriendo: `http://localhost:8000/health`
2. Revisa la consola del backend para errores
3. Verifica que el puerto 8000 no esté ocupado

### Error: "Model not loaded"

**Causa:** Datos CSV no existen o modelo no entrenado

**Solución:**
```bash
cd C:\Users\jacqu\Desktop\floodmirror-ai-backend
python generate_sample_data.py
```

Luego reinicia el backend.

### Frontend no se conecta al backend

**Causa:** URL incorrecta en `.env.local`

**Solución:**
Verifica que `CONCIENCIA-2026\.env.local` contenga:
```
VITE_API_URL=http://localhost:8000
```

Reinicia el servidor frontend después de cambiar `.env.local`.

### Puerto 8000 ocupado

**Solución:**
Usa otro puerto para el backend:
```bash
uvicorn app.main:app --reload --port 8001
```

Y actualiza `.env.local`:
```
VITE_API_URL=http://localhost:8001
```

---

## 📊 Endpoints Disponibles

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/health` | GET | Estado del backend |
| `/predict` | POST | Predicción de riesgo |
| `/model/info` | GET | Info del modelo ML |
| `/data/stats` | GET | Estadísticas de datos |
| `/alcaldia/{name}` | GET | Info de alcaldía |

---

## ✅ Checklist de Integración

- [ ] Backend corriendo en puerto 8000
- [ ] Datos CSV generados en `floodmirror-ai-backend/data/`
- [ ] Frontend corriendo en puerto 5173
- [ ] `.env.local` configurado en frontend
- [ ] Health check funciona: `http://localhost:8000/health`
- [ ] CORS configurado correctamente
- [ ] Predicción de prueba funciona desde `/docs`
- [ ] Frontend puede hacer llamadas al backend

---

## 🎉 ¡Listo!

Ahora tienes:
- ✅ Backend FastAPI con IA funcional
- ✅ Frontend React conectado
- ✅ Módulo Mirror para datos limitados
- ✅ Explicaciones y recomendaciones en español
- ✅ Sistema completo para demo de hackathon

---

## 📞 Comandos Rápidos

### Terminal 1 - Backend:
```bash
cd C:\Users\jacqu\Desktop\floodmirror-ai-backend
venv\Scripts\activate
uvicorn app.main:app --reload
```

### Terminal 2 - Frontend:
```bash
cd C:\Users\jacqu\Desktop\CONCIENCIA-2026
npm run dev
```

### Probar API:
```bash
curl -X POST "http://localhost:8000/predict" -H "Content-Type: application/json" -d "{\"intensidad\":50.0,\"alcaldia\":\"Iztapalapa\"}"
```

---

**¡Tu sistema FloodMirror AI está listo para el hackathon!** 🌊🤖