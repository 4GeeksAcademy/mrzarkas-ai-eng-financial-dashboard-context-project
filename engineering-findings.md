# Hallazgos de ingeniería y reglas propuestas

## Alcance y método

Análisis estático del código, configuración, documentación y pruebas presentes en el repositorio. No se ejecutaron tests ni servicios. Las reglas de este documento son **propuestas**: no se han instalado en `.agents/rules` ni modifican las convenciones existentes.

## 1. Hallazgos y evidencia

### Arquitectura

#### A1. Separación por aplicación y presentación

- **Hallazgo:** El repositorio separa las aplicaciones bajo `backend/` y `frontend/`; Compose coordina ambas. En el backend, el punto de entrada registra un router. En el frontend, `App` coordina la carga y delega la presentación en componentes dashboard.
- **Evidencia:** `docker-compose.yml` declara `frontend` y `backend`; `backend/app/main.py` incluye `router` desde `backend/app/routes.py`; `frontend/src/App.tsx` importa componentes de `frontend/src/components/dashboard/`.
- **Impacto:** Los cambios nuevos pueden localizarse en la aplicación correspondiente y mantener la interacción API/UI explícita.

#### A2. Código backend concentrado en un solo módulo de rutas

- **Hallazgo:** `backend/app/routes.py` reúne modelos Pydantic, tipos de dominio, generación de fixtures, filtros, agregaciones y todas las rutas HTTP.
- **Evidencia:** Clases `FinancialMovement`, `MetricsSummaryItem`, funciones `generate_mock_movements`, `filter_movements`, `summarize_movements` y handlers `get_metrics*` están en `backend/app/routes.py`.
- **Impacto:** Al ampliar la API, el módulo puede crecer y mezclar lógica de dominio con transporte. Conviene reconocer la organización actual antes de introducir módulos o dependencias adicionales.

#### A3. No hay persistencia demostrada

- **Hallazgo:** Los datos que consume la API son generados en memoria; no se observan modelos/cliente de base de datos, migraciones o servicio de almacenamiento en los archivos del repositorio.
- **Evidencia:** `backend/app/routes.py` genera movimientos con `generate_mock_movements(seed=42)` en cada endpoint. `docker-compose.yml` solo define frontend y backend; `backend/requirements.txt` no incluye un driver de base de datos.
- **Impacto:** No se deben asumir persistencia ni una tecnología de base de datos al modificar métricas. Cualquier diseño de persistencia sería una decisión nueva, no una convención existente.

### Naming y estructura

#### N1. Naming idiomático por lenguaje, con módulos frontend kebab-case

- **Hallazgo:** En Python aparecen funciones y variables `snake_case`; en TypeScript/React, componentes y tipos usan `PascalCase`, funciones/variables `camelCase` y los nombres de archivos de componentes están en `kebab-case`.
- **Evidencia:** `backend/app/routes.py` (`generate_mock_movements`, `MetricsSummaryItem`); `frontend/src/components/dashboard/kpi-row.tsx` exporta `KPIRow`; `frontend/src/lib/financial-utils.ts` exporta `computeMonthlyData`; `frontend/src/lib/financial-types.ts` declara `FinancialMovement`.
- **Impacto:** Respetar estas formas reduce inconsistencias en imports y dificulta menos la navegación del repositorio.

#### N2. Tipos de movimiento compartidos entre API y frontend por correspondencia, no por generación automática

- **Hallazgo:** El esquema de movimiento existe en backend y frontend con campos y nombres paralelos; no se observa generación de TypeScript desde OpenAPI ni validación runtime compartida.
- **Evidencia:** `FinancialMovement` en `backend/app/routes.py` define `create_date`, `amount`, `operation_type`, `category`, `business_type`; `frontend/src/lib/financial-types.ts` expone los mismos campos.
- **Impacto:** Los cambios de contrato pueden compilar en un lado y dejar al otro desactualizado si no se revisan ambos explícitamente.

### Backend / API

#### B1. Pydantic y tipos literales expresan el contrato HTTP

- **Hallazgo:** Los endpoints tipan parámetros y respuestas, usan `Literal` para enumeraciones y declaran `response_model` en las rutas que entregan modelos.
- **Evidencia:** `OperationType`, `Category`, `BusinessType`, `GroupBy` y modelos Pydantic en `backend/app/routes.py`; decoradores `@router.get(..., response_model=...)`.
- **Impacto:** Mantener estos contratos favorece validación de parámetros y documentación OpenAPI coherente.

#### B2. Filtrado y transformación se realizan explícitamente antes de responder

- **Hallazgo:** Las rutas obtienen la lista de movimientos, aplican filtros y transformaciones en funciones separadas antes de devolver datos.
- **Evidencia:** `get_metrics` llama a `generate_mock_movements`, `filter_movements` y `ensure_chronological_order`; los agregados usan `summarize_movements` y `build_top_categories` en `backend/app/routes.py`.
- **Impacto:** Lógica reutilizable puede probarse por función y mantenerse separada del parsing de parámetros HTTP.

#### B3. Reutilizar el generador con semilla fija produce datos repetibles, pero modifica estado aleatorio global

- **Hallazgo:** Cada handler llama a `generate_mock_movements(seed=42)` para obtener datos consistentes. La función invoca `random.seed(seed)`, que modifica el generador global de Python.
- **Evidencia:** `generate_mock_movements` y sus llamadas en cada endpoint de `backend/app/routes.py`; `backend/tests/test_routes.py::test_generate_mock_movements_returns_full_year_sorted_data` valida cantidad y orden.
- **Impacto:** La reproducibilidad es útil para el dashboard y pruebas; a la vez, el estado global podría afectar código futuro que use `random` en el mismo proceso y una llamada vuelve a generar la colección completa.

### Frontend

#### F1. `App` concentra la petición API y la composición de la pantalla

- **Hallazgo:** El frontend realiza la única petición observada en `App`, guarda datos/KPI/loading/error y pasa estado a componentes de presentación.
- **Evidencia:** `fetchFinancialData`, `useEffect`, estados React y renderizado en `frontend/src/App.tsx`; props tipadas en los componentes de `frontend/src/components/dashboard/`.
- **Impacto:** Cambios de carga y contrato de API afectan este punto central; nuevos visuales pueden seguir el patrón de datos por props.

#### F2. Los cálculos financieros están en helpers puros separados de la UI

- **Hallazgo:** El cálculo de totales y agregaciones mensuales está fuera de los componentes, en funciones que reciben movimientos y devuelven resultados.
- **Evidencia:** `computeKPIs` y `computeMonthlyData` en `frontend/src/lib/financial-utils.ts`; `App.tsx` las ejecuta antes de pasar sus resultados a KPI y charts.
- **Impacto:** La lógica puede verificarse con pruebas de entrada/salida y los componentes permanecen enfocados en presentación.

#### F3. Estados de carga tienen placeholders visuales; el error de red se normaliza en UI

- **Hallazgo:** Los gráficos y KPI muestran skeletons durante carga. `App` muestra un mensaje ante error, pero el `catch` descarta el error original y el detalle HTTP solo se usa para construir una excepción que luego no se presenta.
- **Evidencia:** `loading`/`error` y `.catch(() => setError(...))` en `frontend/src/App.tsx`; ramas `if (loading)` en `kpi-card.tsx`, `income-outcome-chart.tsx` y `profit-percent-chart.tsx`.
- **Impacto:** La pantalla mantiene feedback, pero se pierde diagnóstico específico ante errores de red/API; los futuros cambios de manejo de fallos deberían conservar la experiencia actual y considerar observabilidad útil.

### Testing

#### T1. Backend: pytest con `TestClient`, pruebas de helpers y endpoints

- **Hallazgo:** Las pruebas combinan invocación directa de utilidades y solicitudes HTTP in-process. Los nombres siguen el patrón `test_...` y verifican valores, filtros, orden y estructura de respuestas.
- **Evidencia:** `backend/tests/test_routes.py`; `TestClient(app)` y configuración del path en `backend/tests/conftest.py`; dependencias `pytest`, `httpx` en `backend/requirements.txt`.
- **Impacto:** Nuevas rutas o cambios en filtros pueden cubrirse de forma consistente sin levantar el servidor.

#### T2. Frontend: Vitest cubre funciones financieras, no componentes ni petición API

- **Hallazgo:** El test frontend disponible prueba KPI, agregación y formatos. No hay pruebas visibles de `App`, componentes React ni petición a `/api/metrics`.
- **Evidencia:** `frontend/src/lib/financial-utils.test.ts`; script `test: vitest run` en `frontend/package.json`; la petición está en `frontend/src/App.tsx`.
- **Impacto:** Los cálculos tienen cobertura observable, pero cambios de renderizado, estados async o integración API no tienen una estrategia de test representada actualmente.

#### T3. Convención visible: tests cerca del área o en carpeta dedicada

- **Hallazgo:** Los tests backend están en `backend/tests/`; el test frontend está al lado del helper en `frontend/src/lib/` con sufijo `.test.ts`.
- **Evidencia:** `backend/tests/test_routes.py`, `frontend/src/lib/financial-utils.test.ts`.
- **Impacto:** No hay una única ubicación común entre stacks; nuevos tests deberían seguir la organización del stack que están cubriendo.

### Configuración y DX

#### C1. Proxy de desarrollo y override opcional de base URL

- **Hallazgo:** Vite enruta `/api` al nombre de servicio Compose `backend`; la UI antepone `VITE_API_BASE_URL` si está configurada, vacío de forma predeterminada.
- **Evidencia:** `frontend/vite.config.ts`, `frontend/src/App.tsx`, `frontend/.env.example`, `docker-compose.yml`.
- **Impacto:** El modo Compose funciona con rutas relativas; cambios en nombre de servicio/puerto/proxy pueden romper la API desde el navegador si no se actualizan conjuntamente.

#### C2. Scripts de desarrollo, calidad y pruebas son específicos por frontend

- **Hallazgo:** npm declara `dev`, `build`, `lint`, `test`, `test:watch` y `test:coverage`; TypeScript activa chequeos de locales/parámetros sin uso.
- **Evidencia:** `frontend/package.json`, `frontend/tsconfig.app.json`, `frontend/eslint.config.js`.
- **Impacto:** Los comandos existentes son la ruta conocida para validar cambios frontend; no se detecta script raíz que orqueste validaciones de ambos stacks.

#### C3. Contenedores están orientados a desarrollo

- **Hallazgo:** El backend arranca Uvicorn con `--reload` y debugpy; el frontend ejecuta el servidor de desarrollo Vite. Compose monta el código local como volumen.
- **Evidencia:** `backend/Dockerfile`, `frontend/Dockerfile`, `docker-compose.yml`.
- **Impacto:** Estos contenedores facilitan iteración local, pero el repositorio no aporta evidencia de imágenes o comandos de producción. No se debe asumir que el Compose mostrado constituye un despliegue productivo.

### Seguridad

#### S1. CORS permite todos los orígenes y credenciales

- **Hallazgo:** La aplicación habilita `allow_origins=["*"]` junto con `allow_credentials=True`, además de permitir todos los métodos y headers.
- **Evidencia:** Configuración `CORSMiddleware` de `backend/app/main.py`.
- **Impacto:** Es una política amplia. Antes de utilizar el backend fuera del entorno local, los orígenes y permisos deben revisarse de acuerdo con el despliegue previsto; el repositorio no define una configuración diferenciada por entorno.

#### S2. No hay credenciales o secretos de servicios declarados en los archivos inspeccionados

- **Hallazgo:** La configuración de Compose no declara variables backend ni credenciales; la única variable de entorno expuesta a la interfaz es `VITE_API_BASE_URL`.
- **Evidencia:** `docker-compose.yml`, `frontend/.env.example`, `frontend/src/App.tsx`.
- **Impacto:** No hay mecanismo visible para configurar secretos o servicios externos; no se debe colocar información sensible en variables `VITE_*` dado que se usan en el bundle del navegador.

### Documentación

#### D1. README bilingüe describe arquitectura y Compose, pero la documentación de `.env` es ambigua

- **Hallazgo:** Los README describen React/TypeScript, FastAPI, el proxy `/api` y el comando Compose. La instrucción dice copiar `frontend/.env.example` a `.env` pero no indica destino, mientras que Vite se ejecuta en el proyecto frontend.
- **Evidencia:** `README.md`, `README.es.md`; `frontend/vite.config.ts`; `frontend/Dockerfile` (`WORKDIR /app` y comando Vite); `frontend/.env.example`.
- **Impacto:** Un desarrollador puede crear el archivo en una ubicación que Vite no lea. Esta misma ambigüedad está registrada en `verification.md`.

#### D2. README presenta estructura de agentes esperada, pero aún no existe

- **Hallazgo:** README propone `.agents/rules` y `.agents/skills`; `AGENTS.md` remite a esas carpetas y a `memory-bank`, pero esas rutas están ausentes en el estado inspeccionado.
- **Evidencia:** `README.md`, `README.es.md`, `AGENTS.md`; inspección del árbol del repositorio sin `.agents/` ni `memory-bank/`.
- **Impacto:** No hay reglas de repositorio instaladas que complementen `AGENTS.md`. La tarea actual deriva propuestas, pero no las implementa.

## 2. Riesgos o inconsistencias prioritarios

1. **Contrato frontend/backend duplicado:** el esquema `FinancialMovement` vive en TypeScript y Pydantic por separado; no se observa generación o test de contrato compartido (N2, T2).
2. **Datos mock y periodo visible potencialmente discordantes:** el backend selecciona año según la fecha actual (`backend/app/routes.py`), pero `frontend/src/App.tsx` muestra `2024 - Full Year`. Es un problema de consistencia visible; la UI tampoco solicita actualmente parámetros de fecha.
3. **Random global en generador:** la semilla constante aporta estabilidad pero cambia el estado aleatorio compartido del proceso (B3).
4. **Política CORS amplia:** todos los orígenes, métodos y headers junto con credenciales (S1).
5. **Diagnóstico frontend limitado:** se oculta el error original de fetch y no hay tests observables de interacción API/UI (F3, T2).
6. **Entorno de despliegue no documentado:** contenedores usan servidores de desarrollo y recarga; no hay evidencia de variante de producción (C3).
7. **Configuración env ambigua:** el README omite la ubicación destino precisa de `.env` (D1).
8. **Convenciones de formato no uniformes:** `frontend/src/App.tsx` emplea comillas dobles y punto y coma, mientras varios componentes dashboard usan comillas simples y omiten punto y coma. Los archivos Python tampoco evidencian formatter/linter configurado. No se identifica una política uniforme que deba imponerse; revisar el estilo cercano antes de editar evita diffs ruidosos.

## 3. Reglas propuestas para el repositorio

### R1. Mantener los contratos de movimiento sincronizados

- **Alcance:** Cambios en campos, categorías, tipos de operación o tipo de negocio que crucen API y UI.
- **Regla:** Al modificar el modelo `FinancialMovement` de backend, revisar y actualizar su representación en `frontend/src/lib/financial-types.ts` y los consumidores/tests pertinentes; conservar los nombres serializados actuales salvo cambio coordinado.
- **Justificación:** Evita que frontend y backend compilen independientemente con expectativas de datos incompatibles.
- **Evidencia:** Modelo con campos coincidentes en `backend/app/routes.py` y `frontend/src/lib/financial-types.ts`; petición/consumo en `frontend/src/App.tsx`.

### R2. Tipar y validar las rutas con el patrón existente

- **Alcance:** Nuevos endpoints y cambios en parámetros/respuestas FastAPI.
- **Regla:** Declarar tipos de parámetros y respuesta; para respuestas estructuradas definir/reutilizar un modelo Pydantic y asociarlo como `response_model`.
- **Justificación:** El patrón actual usa FastAPI/Pydantic para validar y describir el contrato API.
- **Evidencia:** `backend/app/routes.py` usa `Literal`, `Query`, modelos Pydantic y `response_model` en endpoints de métricas.

### R3. Separar cálculos verificables de handlers y presentación

- **Alcance:** Nueva lógica de agregación de métricas o cálculos financieros.
- **Regla:** Mantener el cálculo en funciones que reciben datos y producen resultado, en lugar de duplicar fórmulas en los handlers API o componentes React; añadir o ampliar pruebas de esa lógica.
- **Justificación:** El repositorio ya permite verificar los cálculos de forma aislada en ambas aplicaciones.
- **Evidencia:** Helpers `summarize_movements`, `calculate_net_value` en `backend/app/routes.py`; `computeKPIs`, `computeMonthlyData` en `frontend/src/lib/financial-utils.ts`; pruebas asociadas en los dos stacks.

### R4. Acompañar cambios de endpoints/filtros con pruebas backend

- **Alcance:** Nuevas rutas, filtros, agregaciones o cambios de forma de respuesta.
- **Regla:** Añadir pruebas pytest con `TestClient` para el comportamiento HTTP, y pruebas directas de helper cuando la lógica pueda aislarse; cubrir los filtros relevantes y forma de respuesta.
- **Justificación:** La suite backend existente combina justamente ambos niveles y ya protege orden, filtros y campos.
- **Evidencia:** `backend/tests/test_routes.py`; `backend/tests/conftest.py`.

### R5. Acompañar cambios de cálculos financieros con tests Vitest

- **Alcance:** Funciones de `frontend/src/lib/financial-utils.ts` o reglas de cálculo consumidas por el dashboard.
- **Regla:** Añadir o actualizar pruebas de entrada/salida en `frontend/src/lib/*.test.ts`, incluyendo casos límite introducidos por el cambio.
- **Justificación:** Esta es la ubicación y estrategia existente para KPI, fechas, agregados y formatos.
- **Evidencia:** `frontend/src/lib/financial-utils.test.ts` y script `test` en `frontend/package.json`.

### R6. Conservar el proxy API como contrato de desarrollo Compose

- **Alcance:** Cambios de puerto/nombre de servicio backend, base URL de frontend o routing API de desarrollo.
- **Regla:** Mantener coordinados el prefijo `/api`, target proxy de Vite, servicio `backend` de Compose y el uso opcional de `VITE_API_BASE_URL`; verificar la llamada desde navegador/dev server.
- **Justificación:** El frontend depende de esta cadena para alcanzar el backend local sin CORS de navegador entre orígenes.
- **Evidencia:** `frontend/src/App.tsx`, `frontend/vite.config.ts`, `docker-compose.yml`, `frontend/.env.example`.

### R7. Tratar `VITE_*` como valores públicos del cliente

- **Alcance:** Configuración incluida en el frontend.
- **Regla:** No añadir secretos o credenciales a variables `VITE_*`; utilizarlas solo para valores que puedan exponerse al navegador.
- **Justificación:** La aplicación accede a la variable en código de frontend mediante `import.meta.env`.
- **Evidencia:** `frontend/src/App.tsx` consume `VITE_API_BASE_URL`; `frontend/.env.example` la documenta como configuración del cliente.

### R8. Revisar CORS al cambiar contexto de despliegue

- **Alcance:** Cambios de middleware CORS o uso del backend en otros entornos.
- **Regla:** No ampliar ni replicar automáticamente los comodines actuales; confirmar qué orígenes, métodos y headers requiere el contexto y hacer explícita la política si deja de ser solo desarrollo.
- **Justificación:** La política presente permite todos los orígenes/métodos/headers y credenciales sin distinción de entorno.
- **Evidencia:** `backend/app/main.py`, configuración `CORSMiddleware`.

### R9. Ejecutar las herramientas del stack afectado

- **Alcance:** Cambios de frontend; cambios backend.
- **Regla:** Para frontend, usar los scripts definidos (`npm test`, `npm run lint`, `npm run build`) según alcance. Para backend, ejecutar `pytest` desde `backend/`. No asumir que existe un comando raíz que cubra ambos.
- **Justificación:** Los scripts se declaran en el manifiesto frontend y pytest está declarado en las dependencias backend; no hay orquestación raíz de tests/lint.
- **Evidencia:** `frontend/package.json`, `backend/requirements.txt`, `backend/tests/`, `frontend/src/lib/financial-utils.test.ts`.

### R10. Seguir la organización y naming del stack al añadir módulos

- **Alcance:** Nuevos módulos, componentes y pruebas.
- **Regla:** En Python seguir `snake_case` para funciones/módulos y `PascalCase` para modelos/clases. En React mantener archivos de componentes `kebab-case`, exports de componentes `PascalCase`, helpers `camelCase`; ubicar UI en `frontend/src/components/` y tipos/lógica reusable en `frontend/src/lib/`.
- **Justificación:** Refleja la estructura y nombres ya usados, haciendo las nuevas piezas localizables.
- **Evidencia:** `backend/app/routes.py`, `frontend/src/components/dashboard/`, `frontend/src/lib/financial-utils.ts`, `frontend/src/lib/financial-types.ts`.

### R11. Mantener periodo presentado coherente con los datos solicitados

- **Alcance:** Cambios de periodo, filtros temporales o generación de datos en el dashboard.
- **Regla:** Si cambia el rango de datos mostrado, actualizar la etiqueta del periodo a partir del mismo rango o mantener explícito que es solo un título fijo; no presentar un año no vinculado al conjunto servido.
- **Justificación:** Actualmente la UI etiqueta `2024 - Full Year`, mientras el backend genera fechas relativas al año actual y la carga frontend no envía rango.
- **Evidencia:** `frontend/src/App.tsx`; `_year_for_month` y `generate_mock_movements` en `backend/app/routes.py`.

## 4. Límites de lo que puede concluirse

- No hay convención de base de datos que documentar porque el repositorio inspeccionado no incluye persistencia.
- No se puede concluir cómo se despliega en producción ni qué restricciones CORS/entornos reales requiere el usuario.
- El análisis documenta pruebas existentes pero no afirma que pasen; no se ejecutaron en esta fase.
- No se propone una regla de formato automática porque no hay configuración Python de formatter y el frontend contiene diferencias visibles de estilo entre archivos.
