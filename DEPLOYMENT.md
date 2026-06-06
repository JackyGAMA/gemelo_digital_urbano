# 🚀 FloodMirror AI - Guía de Despliegue

## Checklist Pre-Despliegue

### ✅ Backend
- [x] Modelo V2 entrenado y guardado en `models/best_model.pkl`
- [x] Datos oficiales cargados en `data/`
- [x] Integración meteorológica Open-Meteo implementada
- [x] Sistema de alertas inteligentes funcionando
- [x] Dependencias actualizadas en `requirements.txt`
- [x] Variables de entorno configuradas
- [x] Eliminados archivos de mock/sample data

### ✅ Frontend
- [ ] API URL configurada correctamente
- [ ] Build de producción probado
- [ ] Mapa funcional con datos reales
- [ ] Alertas meteorológicas visibles
- [ ] Privacidad LFPDPPP implementada

### ✅ Documentación
- [x] README.md completo
- [x] Arquitectura documentada
- [x] API endpoints documentados
- [x] Guía de instalación

### ✅ Seguridad
- [x] .gitignore actualizado
- [x] Secretos no incluidos en repositorio
- [x] CORS configurado
- [ ] HTTPS en producción

## 📋 Pasos de Despliegue

### 1. Preparación del Entorno

#### Backend
```bash
cd floodmirror-ai-backend

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Verificar modelo
python -c "from app.ml.model_v2 import FloodPredictionModelV2; m = FloodPredictionModelV2(); m.load_model(); print('✓ Modelo cargado')"
```

#### Frontend
```bash
cd floodmirror-frontend

# Instalar dependencias
bun install  # o npm install

# Configurar API
echo "VITE_API_URL=https://api.floodmirror.com" > .env

# Build
bun run build
```

### 2. Despliegue Backend

#### Opción A: Servidor Linux (Ubuntu/Debian)

```bash
# Instalar dependencias del sistema
sudo apt update
sudo apt install python3.9 python3-pip python3-venv nginx

# Clonar repositorio
git clone https://github.com/yourusername/floodmirror-ai.git
cd floodmirror-ai/floodmirror-ai-backend

# Configurar entorno
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Instalar Gunicorn
pip install gunicorn

# Crear servicio systemd
sudo nano /etc/systemd/system/floodmirror.service
```

**Contenido de floodmirror.service**:
```ini
[Unit]
Description=FloodMirror AI Backend
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/floodmirror-ai-backend
Environment="PATH=/path/to/floodmirror-ai-backend/venv/bin"
ExecStart=/path/to/floodmirror-ai-backend/venv/bin/gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

[Install]
WantedBy=multi-user.target
```

```bash
# Iniciar servicio
sudo systemctl start floodmirror
sudo systemctl enable floodmirror
sudo systemctl status floodmirror
```

**Configurar Nginx**:
```bash
sudo nano /etc/nginx/sites-available/floodmirror
```

```nginx
server {
    listen 80;
    server_name api.floodmirror.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Activar sitio
sudo ln -s /etc/nginx/sites-available/floodmirror /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Instalar SSL con Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d api.floodmirror.com
```

#### Opción B: Docker

**Dockerfile**:
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar aplicación
COPY . .

# Exponer puerto
EXPOSE 8000

# Comando de inicio
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  backend:
    build: ./floodmirror-ai-backend
    ports:
      - "8000:8000"
    environment:
      - API_HOST=0.0.0.0
      - API_PORT=8000
    volumes:
      - ./floodmirror-ai-backend/data:/app/data
      - ./floodmirror-ai-backend/models:/app/models
    restart: unless-stopped

  frontend:
    build: ./floodmirror-frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: unless-stopped
```

```bash
# Desplegar
docker-compose up -d

# Ver logs
docker-compose logs -f

# Detener
docker-compose down
```

#### Opción C: Plataformas Cloud

**Heroku**:
```bash
# Instalar Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Login
heroku login

# Crear app
heroku create floodmirror-api

# Configurar buildpack
heroku buildpacks:set heroku/python

# Crear Procfile
echo "web: uvicorn app.main:app --host 0.0.0.0 --port \$PORT" > Procfile

# Deploy
git push heroku main

# Ver logs
heroku logs --tail
```

**Railway**:
```bash
# Instalar Railway CLI
npm install -g @railway/cli

# Login
railway login

# Inicializar proyecto
railway init

# Deploy
railway up
```

### 3. Despliegue Frontend

#### Opción A: Vercel
```bash
# Instalar Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy
cd floodmirror-frontend
vercel --prod
```

#### Opción B: Netlify
```bash
# Instalar Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
cd floodmirror-frontend
bun run build
netlify deploy --prod --dir=dist
```

#### Opción C: Nginx (Servidor propio)
```bash
# Build
cd floodmirror-frontend
bun run build

# Copiar archivos
sudo cp -r dist/* /var/www/floodmirror/

# Configurar Nginx
sudo nano /etc/nginx/sites-available/floodmirror-frontend
```

```nginx
server {
    listen 80;
    server_name floodmirror.com www.floodmirror.com;
    root /var/www/floodmirror;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
# Activar y recargar
sudo ln -s /etc/nginx/sites-available/floodmirror-frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# SSL
sudo certbot --nginx -d floodmirror.com -d www.floodmirror.com
```

### 4. Configuración de Producción

#### Variables de Entorno Backend
```bash
# .env (NO INCLUIR EN GIT)
API_HOST=0.0.0.0
API_PORT=8000
ENVIRONMENT=production
LOG_LEVEL=INFO
CORS_ORIGINS=https://floodmirror.com,https://www.floodmirror.com
```

#### Variables de Entorno Frontend
```bash
# .env.production
VITE_API_URL=https://api.floodmirror.com
VITE_ENVIRONMENT=production
```

### 5. Monitoreo y Logs

#### Backend Logs
```bash
# Systemd
sudo journalctl -u floodmirror -f

# Docker
docker-compose logs -f backend

# Archivo
tail -f /var/log/floodmirror/app.log
```

#### Métricas
```bash
# Instalar Prometheus + Grafana (opcional)
# https://prometheus.io/docs/introduction/first_steps/
```

### 6. Backup

#### Datos
```bash
# Backup diario
0 2 * * * tar -czf /backups/floodmirror-data-$(date +\%Y\%m\%d).tar.gz /path/to/data/
```

#### Modelos
```bash
# Backup semanal
0 3 * * 0 tar -czf /backups/floodmirror-models-$(date +\%Y\%m\%d).tar.gz /path/to/models/
```

### 7. Actualizaciones

```bash
# Backend
cd floodmirror-ai-backend
git pull
source venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart floodmirror

# Frontend
cd floodmirror-frontend
git pull
bun install
bun run build
sudo cp -r dist/* /var/www/floodmirror/
```

## 🔒 Seguridad en Producción

### Checklist
- [ ] HTTPS habilitado (SSL/TLS)
- [ ] CORS configurado correctamente
- [ ] Rate limiting implementado
- [ ] Firewall configurado
- [ ] Logs de seguridad activos
- [ ] Backups automáticos
- [ ] Monitoreo de uptime
- [ ] Variables de entorno seguras

### Rate Limiting (Nginx)
```nginx
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    location /predict {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://127.0.0.1:8000;
    }
}
```

## 📊 Monitoreo

### Health Checks
```bash
# Backend
curl https://api.floodmirror.com/health

# Frontend
curl https://floodmirror.com
```

### Uptime Monitoring
- UptimeRobot: https://uptimerobot.com/
- Pingdom: https://www.pingdom.com/
- StatusCake: https://www.statuscake.com/

## 🐛 Troubleshooting

### Backend no inicia
```bash
# Verificar logs
sudo journalctl -u floodmirror -n 50

# Verificar puerto
sudo netstat -tulpn | grep 8000

# Verificar permisos
ls -la /path/to/floodmirror-ai-backend
```

### Frontend no carga
```bash
# Verificar Nginx
sudo nginx -t
sudo systemctl status nginx

# Verificar archivos
ls -la /var/www/floodmirror/
```

### Modelo no carga
```bash
# Verificar archivo
ls -lh models/best_model.pkl

# Verificar permisos
chmod 644 models/best_model.pkl

# Probar carga
python -c "from app.ml.model_v2 import FloodPredictionModelV2; m = FloodPredictionModelV2(); m.load_model()"
```

## 📞 Soporte

Para problemas de despliegue:
- Email: support@floodmirror.com
- GitHub Issues: https://github.com/yourusername/floodmirror-ai/issues

---

**FloodMirror AI** - Deployment Guide v2.0