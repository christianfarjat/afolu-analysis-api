# 🌿 AFOLU Analysis API

Backend API REST para análisis AFOLU (Agriculture, Forestry and Other Land Use) y estratificación geoespacial de campos ganaderos con potencial de secuestro de carbono en Patagonia.

## 📋 Descripción

Este backend API proporciona servicios para:
- 📤 Carga y procesamiento de archivos KMZ/KML de campos
- 🗺️ Análisis geoespacial y cálculo de áreas
- 🌡️ Obtención de datos climáticos en tiempo real
- 🌱 Simulación de NDVI y análisis de vegetación  
- 🎯 Clustering K-means para estratificación
- 💰 Cálculo de potencial de secuestro de carbono
- 📊 Recomendaciones AFOLU personalizadas

## 🏗️ Arquitectura

```
afolu-analysis-api/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app y endpoints
│   ├── config.py               # Configuración y variables de entorno
│   ├── models/                 # Pydantic models
│   │   ├── __init__.py
│   │   ├── campo.py
│   │   ├── afolu.py
│   │   └── response.py
│   ├── services/               # Lógica de negocio
│   │   ├── __init__.py
│   │   ├── kmz_processor.py    # Procesamiento de archivos geoespaciales
│   │   ├── clima_service.py    # Datos climáticos Open-Meteo
│   │   ├── ndvi_calculator.py  # Cálculo de NDVI
│   │   ├── clustering.py       # K-means clustering
│   │   └── afolu_calculator.py # Cálculos AFOLU
│   ├── utils/                  # Utilidades
│   │   ├── __init__.py
│   │   ├── geo_utils.py
│   │   └── validators.py
│   └── db/                     # Database (opcional)
│       ├── __init__.py
│       └── models.py
├── tests/
│   ├── __init__.py
│   ├── test_api.py
│   ├── test_services.py
│   └── fixtures/
│       └── test_campo.kmz
├── docs/
│   └── api_documentation.md
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── .env.example
├── .gitignore
├── requirements.txt
├── setup.py
└── README.md
```

## 🚀 Instalación

### Requisitos
- Python 3.9+
- PostgreSQL 13+ (opcional, para persistencia)
- PostGIS (opcional, para geometrías)

### Instalación Local

```bash
# Clonar repositorio
git clone https://github.com/christianfarjat/afolu-analysis-api.git
cd afolu-analysis-api

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate  # Windows

# Instalar dependencias
pip install -r requirements.txt

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus configuraciones

# Iniciar servidor de desarrollo
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Docker

```bash
# Build y run
docker-compose up --build

# API disponible en http://localhost:8000
```

## 📡 Endpoints API

### 1. Upload Campo
```http
POST /api/v1/campos/upload
Content-Type: multipart/form-data

Parameters:
  file: archivo KMZ/KML del campo

Response:
{
  "campo_id": "uuid-123",
  "nombre": "Campo Don José",
  "area_ha": 1834.5,
  "centroide": {
    "lat": -40.5,
    "lon": -70.5
  },
  "created_at": "2025-12-28T20:00:00Z"
}
```

### 2. Análisis Completo
```http
POST /api/v1/analisis/completo
Content-Type: application/json

Body:
{
  "campo_id": "uuid-123"
}

Response:
{
  "campo": {...},
  "clima": {
    "temperatura_actual": 21.5,
    "precipitacion_7dias": 0.0,
    "temperatura_promedio": 12.0
  },
  "ndvi": {
    "promedio": 0.769,
    "min": 0.688,
    "max": 0.880
  },
  "estratificacion": {
    "num_estratos": 5,
    "estratos": [
      {
        "id": 1,
        "ndvi_medio": 0.832,
        "area_ha": 413.4,
        "productividad": "MUY ALTA"
      }
    ]
  },
  "afolu": {...}
}
```

### 3. Análisis AFOLU
```http
POST /api/v1/afolu/calcular
Content-Type: application/json

Body:
{
  "campo_id": "uuid-123",
  "periodo_años": 20
}

Response:
{
  "escenarios": [
    {
      "nombre": "Forestación (Pino/Ciprés)",
      "secuestro_anual_tCO2": 13389.6,
      "secuestro_periodo_tCO2": 267792.0,
      "inversion_total_USD": 4586250.0,
      "beneficio_neto_conservador_USD": -2567630.0,
      "beneficio_neto_optimista_USD": 5126050.0,
      "roi_conservador_%": -55.98,
      "roi_optimista_%": 111.77
    }
  ],
  "recomendacion_principal": {...}
}
```

### 4. Datos Climáticos
```http
GET /api/v1/clima?lat=-40.5&lon=-70.5

Response:
{
  "temperatura_actual": 21.5,
  "precipitacion_7dias": 0.0,
  "temperatura_promedio_anual": 12.0,
  "precipitacion_anual": 180.0
}
```

## 🔧 Configuración

### Variables de Entorno (.env)

```env
# API Configuration
API_VERSION=v1
API_TITLE="AFOLU Analysis API"
DEBUG=True
ALLOWED_ORIGINS=["http://localhost:3000", "https://tu-frontend.com"]

# Database (opcional)
DATABASE_URL=postgresql://user:password@localhost:5432/afolu_db

# External APIs
OPEN_METEO_API_URL=https://api.open-meteo.com/v1/forecast

# File Storage
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760  # 10MB

# AFOLU Configuration
DEFAULT_PERIODO_ANALISIS=20
PRECIO_CARBONO_CONSERVADOR=15  # USD/tCO2
PRECIO_CARBONO_OPTIMISTA=40    # USD/tCO2
```

## 📖 Uso Ejemplo

### Python Client

```python
import requests

# 1. Upload campo
with open('campo.kmz', 'rb') as f:
    response = requests.post(
        'http://localhost:8000/api/v1/campos/upload',
        files={'file': f}
    )
    campo = response.json()
    print(f"Campo ID: {campo['campo_id']}")

# 2. Análisis completo
response = requests.post(
    'http://localhost:8000/api/v1/analisis/completo',
    json={'campo_id': campo['campo_id']}
)
analisis = response.json()

# 3. Mostrar resultados
print(f"Área: {analisis['campo']['area_ha']:.2f} ha")
print(f"NDVI promedio: {analisis['ndvi']['promedio']:.3f}")
print(f"Estratos: {analisis['estratificacion']['num_estratos']}")

# 4. Mejor escenario AFOLU
mejor = analisis['afolu']['escenarios'][0]
print(f"Mejor escenario: {mejor['nombre']}")
print(f"Secuestro anual: {mejor['secuestro_anual_tCO2']:.1f} tCO2/año")
```

### JavaScript/TypeScript Client

```typescript
const API_URL = 'http://localhost:8000/api/v1';

// 1. Upload campo
const formData = new FormData();
formData.append('file', campoFile);

const campoResponse = await fetch(`${API_URL}/campos/upload`, {
  method: 'POST',
  body: formData
});
const campo = await campoResponse.json();

// 2. Análisis AFOLU
const afolusResponse = await fetch(`${API_URL}/afolu/calcular`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    campo_id: campo.campo_id,
    periodo_años: 20
  })
});
const afoluData = await afolusResponse.json();

console.log('Escenarios AFOLU:', afolusData.escenarios);
```

## 🧪 Testing

```bash
# Ejecutar tests
pytest tests/ -v

# Con coverage
pytest tests/ --cov=app --cov-report=html

# Test específico
pytest tests/test_afolu_calculator.py::test_calcular_escenarios -v
```

## 📊 Documentación API

La documentación interactiva está disponible en:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- OpenAPI JSON: http://localhost:8000/openapi.json

## 🚢 Despliegue

### Google Cloud Run

```bash
# Build imagen
gcloud builds submit --tag gcr.io/PROJECT_ID/afolu-api

# Deploy
gcloud run deploy afolu-api \
  --image gcr.io/PROJECT_ID/afolu-api \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

### AWS ECS/Fargate

```bash
# Ver docker/aws-deploy.sh para instrucciones completas
```

### Heroku

```bash
heroku create afolu-analysis-api
heroku config:set DEBUG=False
git push heroku main
```

## 🔒 Seguridad

- ✅ Validación de archivos (tipo, tamaño)
- ✅ Rate limiting configurado
- ✅ CORS configurado
- ✅ Input sanitization
- 🔜 Autenticación JWT (próximamente)
- 🔜 API Keys (próximamente)

## 📈 Roadmap

- [ ] Autenticación y autorización
- [ ] Persistencia en base de datos
- [ ] Cache con Redis
- [ ] Procesamiento asíncrono con Celery
- [ ] Integración con Google Earth Engine
- [ ] Exportación de reportes PDF
- [ ] WebSocket para updates en tiempo real
- [ ] Panel de administración

## 🤝 Contribuir

Las contribuciones son bienvenidas! Por favor:

1. Fork el repositorio
2. Crea una rama (`git checkout -b feature/nueva-funcionalidad`)
3. Commit cambios (`git commit -am 'Agregar nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abre un Pull Request

## 📄 Licencia

MIT License - ver archivo LICENSE para detalles

## 👥 Autor

Christian Fernandez Farjat
- GitHub: [@christianfarjat](https://github.com/christianfarjat)

## 🙏 Agradecimientos

- Datos climáticos: [Open-Meteo API](https://open-meteo.com/)
- Metodología AFOLU: IPCC Guidelines for National GHG Inventories (2019)
- Inspirado en el análisis geoespacial de Campo Don José, Río Negro, Patagonia

---

**Estado del Proyecto**: 🚧 En Desarrollo Activo

**Versión**: 0.1.0
