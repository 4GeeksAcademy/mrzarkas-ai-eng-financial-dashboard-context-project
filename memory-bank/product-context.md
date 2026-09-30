# Contexto del producto

## Propósito documentado

El repositorio presenta un **dashboard de métricas financieras** construido con frontend React + TypeScript y backend FastAPI (`README.md`, `README.es.md`). La interfaz muestra ingresos, egresos, beneficio y margen, con gráficos mensuales (`frontend/src/App.tsx`, `frontend/src/components/dashboard/kpi-row.tsx`, `income-outcome-chart.tsx`, `profit-percent-chart.tsx`).

**Problema de negocio concreto que resuelve:** ❓ No verificado. La documentación identifica el producto como dashboard financiero, pero no define una persona usuaria, proceso de negocio, organización cliente ni problema operacional más específico.

## Actores identificados

- **Desarrolladores/estudiantes que ejecutan y modifican el proyecto:** los README describen el uso educativo, fork, Codespaces/entorno local y ejecución de un agente (`README.md`, `README.es.md`).
- **Usuario de la interfaz del dashboard:** la aplicación está construida para mostrar métricas, pero no se identifica formalmente un rol, tipo de empresa ni permisos (`frontend/src/App.tsx`). Por ello, el perfil del usuario final es ❓ No verificado.
- No se observa autenticación ni autorización en los archivos consultados (`backend/app/main.py`, `backend/app/routes.py`, `frontend/src/App.tsx`); esto describe el repositorio, no descarta controles externos.

## Capacidades implementadas y verificables

### Interfaz

- Solicita `GET /api/metrics` al cargar, mantiene estados de carga/error y transforma la respuesta con `computeKPIs` y `computeMonthlyData` (`frontend/src/App.tsx`, `frontend/src/lib/financial-utils.ts`).
- Presenta cuatro KPI: total de ingresos, total de egresos, beneficio y margen de beneficio (`frontend/src/components/dashboard/kpi-row.tsx`).
- Presenta gráficos mensuales de ingresos/egresos y porcentaje de beneficio; utiliza placeholders mientras carga (`frontend/src/components/dashboard/income-outcome-chart.tsx`, `profit-percent-chart.tsx`, `kpi-card.tsx`).
- El cliente formatea importes en USD y porcentajes con un decimal (`frontend/src/lib/financial-utils.ts`).

### API y datos

- `/health` devuelve `{"status":"ok"}` (`backend/app/routes.py`).
- `/api/metrics` ofrece movimientos y acepta filtros por fechas, categoría y tipo de operación (`backend/app/routes.py`).
- La API también implementa facetas, resúmenes agrupados, categorías principales, comparación entre periodos, alertas y endpoints B2B/B2C (`backend/app/routes.py`).
- Las respuestas usan modelos Pydantic. Cada handler genera 360 movimientos en memoria con semilla `42`; no se lee ni escribe una base de datos en la implementación inspeccionada (`backend/app/routes.py`).
- El frontend React inspeccionado llama únicamente a `/api/metrics`; no se verificó integración visual para las rutas agregadas restantes (`frontend/src/App.tsx`).

## Flujos principales

1. Compose inicia frontend y backend. El navegador accede al servidor Vite en el puerto `5173` (`docker-compose.yml`, `frontend/Dockerfile`).
2. `frontend/src/main.tsx` monta `App`; `App` pide `/api/metrics`.
3. En desarrollo, Vite envía `/api` a `http://backend:8000`, destino que corresponde al servicio Compose (`frontend/vite.config.ts`, `docker-compose.yml`).
4. FastAPI genera los movimientos simulados, aplica filtros y devuelve JSON (`backend/app/routes.py`).
5. React calcula KPI/series mensuales en el navegador y los pasa a componentes de presentación (`frontend/src/App.tsx`, `frontend/src/lib/financial-utils.ts`, `frontend/src/components/dashboard/`).

## Límites funcionales y discrepancias

- La cabecera del frontend fija el periodo en `2024 - Full Year`, mientras que el generador backend deriva los años de la fecha actual y `App` no envía fechas. La etiqueta puede no describir los movimientos recibidos (`frontend/src/App.tsx`, `backend/app/routes.py`).
- La API ofrece filtros y endpoints de análisis adicionales, pero la pantalla principal observada no presenta controles de filtros ni consume esos endpoints (`frontend/src/App.tsx`).
- `frontend/src/lib/mock-data.ts` conserva una muestra local fechada en 2024, pero `App` no la importa y usa la API; no se observó una ruta UI alternativa que la use (`frontend/src/lib/mock-data.ts`, `frontend/src/App.tsx`).
- Los errores de fetch se reemplazan por un mensaje genérico; el detalle de la excepción no se muestra (`frontend/src/App.tsx`).
- Los datos son mock y volátiles en memoria; uso de datos financieros reales, persistencia e integraciones externas: ❓ No verificado (`backend/app/routes.py`, `docker-compose.yml`).

## Referencias

Para estado, riesgos y comandos de ejecución, consulta `memory-bank/project-status.md` y `memory-bank/technology-stack.md`. La documentación del producto está en `README.md` y `README.es.md`; la descripción técnica contrastada está en `verification.md`.
