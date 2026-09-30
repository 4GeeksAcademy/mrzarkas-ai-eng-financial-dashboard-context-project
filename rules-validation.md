# Validación de reglas del repositorio

## Validación de estructura

Tras la revisión del formato, se reestructuraron los archivos de `.agents/rules/` con la estructura solicitada:

1. Frontmatter YAML al inicio con `title`, `description`, `scope: project`, globs concretos y `alwaysApply: false`.
2. Separador `---` entre metadatos y contenido.
3. Instrucciones en español organizadas como pasos numerados y evidencia contextual al final.

Las reglas se aplican por patrón de archivo en lugar de cargarse para cada tarea del proyecto. Se usa `alwaysApply: false` para que los globs sean los que delimiten el contexto; los patrones se mantuvieron acotados a los archivos donde esas instrucciones resultan pertinentes.

Se mantuvieron las convenciones respaldadas por el repositorio y se conservaron advertencias que evitan afirmar que existe infraestructura no demostrada. No se agregó una dependencia a `memory-bank/conventions.md` ni a `memory-bank/proposals.md`, porque esos archivos no forman parte de la estructura existente ni de la evidencia disponible.

## Archivos revisados

- `.agents/rules/contracts-and-architecture.md`: sincronización del contrato Python/TypeScript, organización existente del código y coherencia de la etiqueta temporal.
- `.agents/rules/api-and-data.md`: contratos FastAPI/Pydantic, reproducibilidad de datos mock y aislamiento de cálculos financieros.
- `.agents/rules/testing-and-tooling.md`: pruebas backend con pytest, pruebas frontend con Vitest y reporte de validaciones ejecutadas.
- `.agents/rules/frontend-and-configuration.md`: coordinación entre API, proxy Vite y Compose; secretos en variables Vite; CORS; configuración de desarrollo frente a producción.
- `.agents/rules/naming-conventions.md`: naming Python `snake_case`/`PascalCase`, naming TypeScript/React `camelCase`/`PascalCase` y archivos de componentes `kebab-case`.

Cada archivo declara globs explícitos para el backend (`*.py` bajo `backend/`), el frontend (fuentes TypeScript/TSX y configuración pertinente) o ambos cuando la convención cruza el contrato.

## Comprobaciones y límites

- Se inspeccionó cada regla para confirmar que los metadatos requeridos, el separador y las instrucciones numeradas están presentes.
- Se contrastaron las instrucciones con los archivos citados, incluidos `backend/app/routes.py`, `backend/app/main.py`, `backend/tests/`, `frontend/src/`, `frontend/package.json`, `frontend/vite.config.ts`, Dockerfiles y `docker-compose.yml`.
- Se verificó el whitespace de los cuatro archivos de reglas y de este informe.
- La validación es estructural y estática; no se modificó código funcional, tests ni configuración del proyecto.
- `pytest` no pudo recoger tests porque falta `fastapi` en el Python activo. `npm test` no pudo iniciar porque `vitest` no está instalado en `frontend/`. ESLint y build no se ejecutaron; no se afirma que las suites pasen.
- No se instalaron dependencias ni se levantaron servicios.

## Archivos de reglas e informe de Fase 3

- `.agents/rules/contracts-and-architecture.md`
- `.agents/rules/api-and-data.md`
- `.agents/rules/testing-and-tooling.md`
- `.agents/rules/frontend-and-configuration.md`
- `.agents/rules/naming-conventions.md`
- `rules-validation.md`
