# Evidencia de las corridas ejecutadas

Esta carpeta conserva únicamente los cinco resúmenes JSON que sustentan la
entrega. Los demás archivos de resultados locales permanecen ignorados para no
mezclar repeticiones, pruebas exploratorias o volcados potencialmente grandes.

| Archivo | Escenario y propósito |
|---|---|
| `summary-baseline-personas.json` | Línea base de `POST /register` con personas válidas: 20 VUs durante 5 minutos. |
| `summary-voters-baseline.json` | Validación del resultado de negocio bajo la misma línea base. |
| `summary-load-50us.json` | Carga 0→200 VUs antes del pool de conexiones. El sufijo conserva el nombre original de la corrida. |
| `summary-load-pool.json` | Misma carga después de incorporar HikariCP con máximo 20 conexiones. |
| `summary-stress-pool.json` | Estrés 200→600 VUs con el pool aplicado. |
| `summary-voters-ci.json` | Verificación corta de las seis reglas de negocio y comparación cliente-servidor. |

Los scripts que generaron los artefactos son
[`../scripts/register_person_k6.js`](../scripts/register_person_k6.js) y
[`../scripts/register_voter_k6.js`](../scripts/register_voter_k6.js). Las
interpretaciones y la matriz de rendimiento están en la Wiki del repositorio.
