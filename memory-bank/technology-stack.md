# Stack tecnológico

Inventario basado en manifiestos, configuración, código y pruebas del repositorio. No se infieren servicios externos no declarados.

## Lenguajes y runtimes

- **Python 3.13** en la imagen backend (`backend/Dockerfile`); fuentes de aplicación y tests Python (`backend/app/`, `backend/tests/`). Versión exacta de Python fuera del contenedor: ❓ No verificado.
- **TypeScript/JavaScript** con frontend configurado como paquete ESM (`frontend/package.json`, `frontend/tsconfig*.json`). La imagen frontend usa Node 24 Alpine (`frontend/Dockerfile`).
- **HTML/CSS** en `frontend/index.html`, `frontend/src/index.css` y clases Tailwind usadas por los componentes.

## Aplicación y librerías

### Backend

- FastAPI para la aplicación HTTP y sus rutas (`backend/requirements.txt`, `backend/app/main.py`, `backend/app/routes.py`).
- Pydantic para validación/serialización de modelos de movimiento, métricas y respuestas (`backend/app/routes.py`).
- Uvicorn como servidor ASGI y debugpy para depuración remota (`backend/requirements.txt`, `backend/Dockerfile`).

### Frontend

- React y React DOM (`frontend/package.json`); Vite y `@vitejs/plugin-react` para servidor/build (`frontend/package.json`, `frontend/vite.config.ts`).
- TypeScript, Vitest y ESLint (`frontend/package.json`, `frontend/tsconfig.app.json`, `frontend/eslint.config.js`).
- Recharts para gráficos; Lucide React para iconos; Tailwind CSS con plugin Vite para estilos (`frontend/package.json`, `frontend/vite.config.ts`).
- `class-variance-authority`, `clsx` y `tailwind-merge` están declarados en el manifiesto del cliente. Su uso en todo el producto no se deduce solo de la declaración.

## API, datos y persistencia

- API propia FastAPI con rutas bajo `/api/metrics` y `/health` (`backend/app/routes.py`).
- Desarrollo navegador-backend: proxy Vite `/api` hacia `http://backend:8000` (`frontend/vite.config.ts`). `VITE_API_BASE_URL` es un override opcional usado por `App` (`frontend/src/App.tsx`); es una variable pública de frontend, no un secreto.
- Los datos se generan en memoria, con semilla `42` al llamar desde endpoints (`backend/app/routes.py`).
- **Base de datos, ORM, migraciones, proveedor de nube, telemetría o API externa:** ❓ No verificado; no aparecen en los manifiestos/configuración/código consultados. No se debe inferir infraestructura externa ausente del repo.

## Contenedores e infraestructura de desarrollo

- Docker Compose define los servicios `frontend` y `backend`, puertos `5173`, `8000` y `5678`, volúmenes de código y dependencia frontend sobre backend (`docker-compose.yml`).
- `frontend/Dockerfile` inicia Vite en modo dev; `backend/Dockerfile` inicia debugpy y Uvicorn con recarga (`--reload`). Los archivos demuestran configuración de desarrollo, no un despliegue de producción.
- Orquestación cloud/producción, CI/CD y gestión de infraestructura: ❓ No verificado en los archivos analizados.

## Dependencias y comandos

- Backend: lista de paquetes sin pins de versión en `backend/requirements.txt`; comando de tests `pytest` desde `backend/`.
- Frontend: dependencias npm y scripts declarados en `frontend/package.json`; lockfile `frontend/package-lock.json`. Scripts: `npm run dev`, `npm run build` (TypeScript y Vite), `npm run lint`, `npm test` (Vitest), `npm run test:watch`, `npm run test:coverage`.
- Configuración estática: TypeScript en `frontend/tsconfig*.json`, ESLint en `frontend/eslint.config.js`, Vite en `frontend/vite.config.ts`.

## Tests observados

- Backend: pytest + httpx/TestClient; pruebas de helpers y rutas en `backend/tests/test_routes.py`, ajuste de ruta de imports en `backend/tests/conftest.py`.
- Frontend: Vitest prueba helpers de KPI, agregación mensual y formato en `frontend/src/lib/financial-utils.test.ts`. No se encontraron pruebas de componentes/renderizado ni de interacción API en el conjunto inspeccionado.
- Estado de ejecución de las suites en el entorno de análisis: consultar `memory-bank/project-status.md`; no afirmar que las pruebas pasan sin ejecutarlas en un entorno con dependencias.
