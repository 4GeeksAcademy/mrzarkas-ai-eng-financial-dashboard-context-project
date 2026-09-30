# Estado del proyecto

Estado descrito según los archivos actuales del repositorio. “Implementado” significa que el código/configuración existe; no implica que haya sido desplegado ni que los tests pasen.

## Implementado

- Aplicación FastAPI con `/health`, endpoints de movimientos, facetas, resumen, categorías, comparación, alertas y segmentos B2B/B2C (`backend/app/main.py`, `backend/app/routes.py`).
- Generación determinista de 360 movimientos simulados por llamada de endpoint con semilla `42`, filtros por fecha/categoría/tipo y agregaciones backend (`backend/app/routes.py`; prueba de cantidad/orden en `backend/tests/test_routes.py`).
- Interfaz React que carga `/api/metrics`, calcula KPI y series por mes y presenta tarjetas/gráficos (`frontend/src/App.tsx`, `frontend/src/lib/financial-utils.ts`, `frontend/src/components/dashboard/`).
- Tests backend con `TestClient` y tests frontend de helpers financieros con Vitest (`backend/tests/`, `frontend/src/lib/financial-utils.test.ts`).
- Flujo local Docker Compose con proxy Vite y backend (`docker-compose.yml`, `frontend/vite.config.ts`, Dockerfiles).
- Reglas de agentes en `.agents/rules/` y esta memoria bajo `memory-bank/`.

## Parcialmente implementado

- **Capacidades API no integradas en la pantalla principal observada:** la API ofrece facetas, agrupaciones, categorías top, comparación, alertas y filtros; `frontend/src/App.tsx` solo pide `/api/metrics` sin parámetros y no consume las demás rutas. El backend existe; la integración UI no se observa.
- **Gestión de fallos del cliente:** hay indicador de error, pero el `catch` descarta la causa original y solo enseña un mensaje genérico (`frontend/src/App.tsx`).
- **Coherencia del periodo:** hay etiqueta de periodo, pero fija (`2024 - Full Year`), mientras el backend genera fechas relativas a la fecha actual y el cliente no pide un rango (`frontend/src/App.tsx`, `backend/app/routes.py`).

## Gaps conocidos respaldados por el repositorio

- **Sin persistencia implementada:** los endpoints generan datos en memoria; no se ven DB, ORM, migraciones ni servicio de almacenamiento en código/configuración consultados (`backend/app/routes.py`, `backend/requirements.txt`, `docker-compose.yml`). Infraestructura fuera del repositorio: ❓ No verificado.
- **Contrato duplicado manualmente:** `FinancialMovement` aparece en Pydantic y TypeScript sin generación compartida visible (`backend/app/routes.py`, `frontend/src/lib/financial-types.ts`).
- **CORS permisivo:** la app permite todos los orígenes, métodos y headers junto con credenciales; no se ve configuración diferenciada por entorno (`backend/app/main.py`).
- **Pruebas frontend limitadas a helpers:** el test existente cubre cálculos/formato, no componentes ni petición API (`frontend/src/lib/financial-utils.test.ts`, `frontend/src/App.tsx`). Esto es una cobertura no observada, no prueba de ausencia de tests externos.
- **Configuración de contenedores orientada a desarrollo:** Vite dev server, debugpy, Uvicorn `--reload` y montajes de código (`frontend/Dockerfile`, `backend/Dockerfile`, `docker-compose.yml`). Una configuración de producción: ❓ No verificado.
- **Ambigüedad documental del `.env`:** README indica copiar `frontend/.env.example` a `.env` sin especificar destino; `verification.md` documenta la incertidumbre.
- **Las pruebas no se ejecutaron exitosamente en la sesión de análisis anterior:** `pytest` no pudo coleccionar por ausencia local de `fastapi`; `npm test` no inició porque `vitest` no estaba instalado. ESLint/build tampoco se ejecutaron. Esto describe ese entorno/ejecución, no el estado de CI ni otros entornos.

## Siguientes prioridades ya documentadas

No se encontraron prioridades/roadmap de producto aprobados, archivos SPEC/PLAN/TASK, ni TODO/FIXME de producto en la búsqueda de repositorio. Por tanto, **no se inventa un roadmap**.

Sí hay pasos documentales del README, no prioridades de producto:

- El README recomienda ejecutar el proyecto con `docker compose up --build` y ajustar/validar las reglas para el flujo real (`README.md`, `README.es.md`).
- Los riesgos R1–R11 y propuestas de reglas de `engineering-findings.md` son hallazgos/propuestas; no se presentan como trabajo aprobado ni como funcionalidades comprometidas.
- `verification.md` registra como inciertas la ubicación del `.env` y algunas condiciones de entorno; estas notas no establecen tareas aprobadas.

En consecuencia, siguientes pasos específicos de producto: ❓ No verificado.

## Referencias

- Producto y flujos: `memory-bank/product-context.md`.
- Stack y herramientas: `memory-bank/technology-stack.md`.
- Arquitectura y ejecución verificada: `verification.md`.
- Hallazgos y propuestas no aprobadas: `engineering-findings.md`.
