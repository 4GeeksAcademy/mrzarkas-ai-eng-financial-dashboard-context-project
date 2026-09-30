# Handover verificado del proyecto

> Este documento describe el estado del repositorio inspeccionado. Los estados califican la evidencia local: **✅ Verificado directamente en el código**; **❌ La documentación existente no coincide con el código**; **❓ No se puede determinar con la información disponible**. La ausencia de una integración en los archivos inspeccionados no demuestra que no exista fuera del repositorio.

## 1. Resumen del proyecto

- ✅ El repositorio contiene un dashboard de métricas financieras con frontend React + TypeScript y API backend FastAPI. Lo indican `README.md`, `README.es.md`, `frontend/package.json` y `backend/requirements.txt`.
- ✅ El frontend solicita `GET /api/metrics`, calcula KPI y agregados mensuales en el navegador y presenta indicadores y gráficos. Evidencia: `frontend/src/App.tsx`, `frontend/src/lib/financial-utils.ts` y `frontend/src/components/dashboard/`.
- ✅ El backend ofrece una API de movimientos y métricas agregadas. Los datos son generados en memoria por `generate_mock_movements(seed=42)` en `backend/app/routes.py`; no se observa lectura/escritura de una base de datos.
- ✅ La API genera 360 movimientos de ejemplo por petición (12 meses por 30 movimientos) y expone filtros, facetas, resúmenes, categorías principales, comparaciones, alertas y listas B2B/B2C. Evidencia: `backend/app/routes.py`.
- ✅ La interfaz tiene un encabezado de periodo fijo `2024 - Full Year`, mientras que el generador backend calcula el año usando `date.today()`. Por tanto, el periodo mostrado no está vinculado al rango que devuelve la API y puede no describir esos datos. Evidencia: `frontend/src/App.tsx` y `backend/app/routes.py` (`_year_for_month`, `generate_mock_movements`).

## 2. Estructura y servicios

### Directorios principales

- ✅ `backend/`: aplicación FastAPI, dependencias Python, imagen de contenedor y pruebas.
- ✅ `frontend/`: aplicación Vite/React, dependencias npm, configuración de TypeScript/ESLint, interfaz y pruebas.
- ✅ `frontend/src/components/dashboard/`: encabezado, tarjetas KPI y gráficos (`dashboard-header.tsx`, `kpi-row.tsx`, `income-outcome-chart.tsx`, `profit-percent-chart.tsx`).
- ✅ `frontend/src/lib/`: tipos financieros, cálculo/formateo, datos mock locales y pruebas de utilidades.
- ✅ `docker-compose.yml`: definición de los servicios `frontend` y `backend`.
- ✅ `AGENTS.md`: directrices para agentes. Indica revisar `.agents/rules`, `.agents/skills` y `memory-bank`; esas ubicaciones no están presentes en el árbol inspeccionado.

### Servicios y comunicación

- ✅ `frontend`: servidor de desarrollo Vite en el puerto `5173`. `frontend/Dockerfile` ejecuta `npm run dev -- --host 0.0.0.0 --port 5173`; `docker-compose.yml` publica `5173:5173`.
- ✅ `backend`: FastAPI/Uvicorn en `8000`; la imagen también inicia debugpy escuchando en `5678`. `backend/Dockerfile` define el comando y `docker-compose.yml` publica ambos puertos.
- ✅ La interfaz hace `fetch` a `/api/metrics` (`frontend/src/App.tsx`). Vite reenvía las solicitudes `/api` a `http://backend:8000` (`frontend/vite.config.ts`); el nombre `backend` coincide con el servicio de Compose. La ruta API está implementada en `backend/app/routes.py`.
- ✅ Compose establece `depends_on: backend` para el frontend y monta los directorios de código como volúmenes. Evidencia: `docker-compose.yml`.
- ✅ No se identifica una base de datos, cliente de base de datos, servicio de persistencia ni integración con proveedores externos en los archivos de configuración/código inspeccionados. El backend usa datos simulados en memoria. No se puede descartar infraestructura externa no declarada en este repositorio.

## 3. Entry points

- ✅ Backend ASGI: `backend/app/main.py`, objeto `app`; el comando de `backend/Dockerfile` lo carga como `app.main:app`.
- ✅ Backend rutas: `backend/app/routes.py`, router incluido por `backend/app/main.py`; contiene modelos, generación de datos, lógica de métricas y endpoints.
- ✅ Frontend: `frontend/index.html` carga `frontend/src/main.tsx`, que monta `frontend/src/App.tsx` en el elemento `root`.
- ✅ Contenedores: `backend/Dockerfile` y `frontend/Dockerfile`; orquestación local: `docker-compose.yml`.
- ✅ Endpoints API identificados en `backend/app/routes.py`: `/health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b` y `/api/metrics/b2c`.

## 4. Flujo general de ejecución

1. ✅ Compose construye e inicia los contenedores frontend y backend (configuración en `docker-compose.yml` y comandos en sus Dockerfiles).
2. ✅ El navegador carga la aplicación servida por Vite en `5173`; `frontend/src/main.tsx` monta `App`.
3. ✅ `App` solicita `/api/metrics`; la configuración proxy de Vite envía `/api` a `backend:8000`.
4. ✅ FastAPI atiende la ruta, genera movimientos de ejemplo con semilla `42`, aplica los filtros solicitados y devuelve JSON. Evidencia: `backend/app/routes.py`.
5. ✅ El frontend convierte los movimientos recibidos a KPI y valores mensuales mediante `computeKPIs` y `computeMonthlyData` (`frontend/src/lib/financial-utils.ts`) y los presenta mediante los componentes bajo `frontend/src/components/dashboard/`.

## 5. Configuración necesaria

- ✅ En el modo Compose documentado, los puertos configurados son `5173` (frontend), `8000` (API) y `5678` (debugpy). El archivo Compose monta `./frontend:/app` y `./backend:/app`.
- ✅ Vite tiene un proxy `/api` a `http://backend:8000` (`frontend/vite.config.ts`). En `frontend/src/App.tsx`, `VITE_API_BASE_URL` es opcional y su valor por defecto es una cadena vacía.
- ✅ `frontend/.env.example` contiene `VITE_API_BASE_URL=` y describe el proxy como predeterminado. No se observa configuración de variables de entorno requerida por el backend.
- ❓ **La documentación no precisa el destino del archivo de entorno.** `README.md` y `README.es.md` indican copiar `frontend/.env.example` a `.env`, pero no especifican si el destino es la raíz del repositorio o `frontend/`. Si se necesita el override, la ubicación coherente con el proyecto Vite ejecutado desde `/app` (`frontend/Dockerfile`) es `frontend/.env` en el repositorio. El override solo hace falta si se quiere usar un origen de backend distinto al proxy.
- ✅ Los valores de `VITE_API_BASE_URL` se incorporan al frontend; no se deben tratar como secretos. Evidencia: uso en `frontend/src/App.tsx`.
- ❓ No se puede determinar desde los archivos del repositorio la versión exacta de Podman instalada o si el entorno requiere opciones particulares para invocar Compose. La definición disponible es `docker-compose.yml`.

## 6. Comandos para ejecutar el proyecto

### Con Docker Compose (comando documentado)

Desde la raíz:

```bash
docker compose up --build
```

La documentación (`README.md`, `README.es.md`) indica las URL locales:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Documentación interactiva de API: `http://localhost:8000/docs` (FastAPI)

✅ Los puertos indicados concuerdan con `docker-compose.yml` y los entry points del backend. Para el entorno del usuario, los contenedores observados publican los mismos puertos; sus comandos coinciden con los comandos de `frontend/Dockerfile` y `backend/Dockerfile`.

### Ejecución local sin contenedores

Los siguientes comandos se derivan de los manifiestos; el repositorio no documenta un procedimiento manual separado:

```bash
# Terminal 1
cd backend
python -m pip install -r requirements.txt
uvicorn app.main:app --reload

# Terminal 2
cd frontend
npm install
npm run dev
```

✅ Los paquetes y scripts necesarios están declarados en `backend/requirements.txt` y `frontend/package.json`; Uvicorn carga `app.main:app` y Vite sirve la aplicación. La ejecución manual simultánea presupone que `backend` es resoluble desde el navegador o que se configura `VITE_API_BASE_URL`; el proxy de Vite configurado apunta al nombre de servicio Compose `backend`.

## 7. Comandos para ejecutar los tests

### Backend

```bash
cd backend
pytest
```

✅ `pytest` está en `backend/requirements.txt`; las pruebas están en `backend/tests/test_routes.py` y el archivo de configuración de importaciones es `backend/tests/conftest.py`. Cubren generación/filtrado de movimientos y endpoints API.

### Frontend

```bash
cd frontend
npm install
npm test
```

✅ `frontend/package.json` define `test` como `vitest run`; la prueba disponible está en `frontend/src/lib/financial-utils.test.ts`. También declara `npm run test:watch` y `npm run test:coverage`.

❓ No se ejecutaron los tests como parte de este análisis; los comandos se documentan a partir de los manifiestos y archivos de prueba.

## 8. Rutas clave del repositorio

| Ruta | Propósito | Relevancia |
|---|---|---|
| `README.md`, `README.es.md` | Descripción del proyecto y guía de ejecución | Referencia de documentación, contrastada con los manifiestos y el código. |
| `AGENTS.md` | Instrucciones para agentes | Indica ubicaciones adicionales de reglas/contexto. |
| `docker-compose.yml` | Servicios, puertos, volúmenes y dependencia | Describe cómo se conectan y levantan los contenedores. |
| `backend/Dockerfile` | Imagen y comando del backend | Acredita Uvicorn, aplicación ASGI, recarga y debugpy. |
| `backend/requirements.txt` | Dependencias Python | Identifica FastAPI, Uvicorn, debugpy, pytest y httpx. |
| `backend/app/main.py` | Creación de FastAPI, CORS y registro del router | Entry point ASGI y conexión de rutas. |
| `backend/app/routes.py` | Modelos, datos de ejemplo, lógica y endpoints | Fuente de verdad de la API actual. |
| `backend/tests/conftest.py` | Preparación del path de imports para tests | Configuración del entorno pytest. |
| `backend/tests/test_routes.py` | Pruebas API/backend | Define cobertura observable del backend. |
| `frontend/Dockerfile` | Imagen y comando Vite | Acredita la ejecución de desarrollo en contenedor. |
| `frontend/package.json`, `frontend/package-lock.json` | Dependencias y scripts npm | Instalación, desarrollo, build, lint y pruebas frontend. |
| `frontend/.env.example` | Ejemplo de variable opcional | Documenta el override del origen API. |
| `frontend/vite.config.ts` | Plugins, alias y proxy `/api` | Acredita la comunicación de desarrollo frontend-backend. |
| `frontend/index.html`, `frontend/src/main.tsx` | Documento HTML y montaje React | Entry point de la interfaz. |
| `frontend/src/App.tsx` | Carga y presentación del dashboard | Coordina la petición a la API, estado y componentes. |
| `frontend/src/lib/financial-types.ts` | Tipos de datos financieros | Contrato de tipos usado por los helpers y la interfaz. |
| `frontend/src/lib/financial-utils.ts` | Cálculos KPI, agregación y formato | Lógica frontend para las métricas visibles. |
| `frontend/src/lib/financial-utils.test.ts` | Pruebas de utilidades | Prueba los cálculos y formatos frontend. |
| `frontend/src/components/dashboard/` | Componentes del dashboard | Presentación de KPI, encabezado y gráficos. |
| `frontend/src/lib/mock-data.ts` | Conjunto de datos mock local | Archivo presente; `App.tsx` usa actualmente la API y no importa este módulo. |
