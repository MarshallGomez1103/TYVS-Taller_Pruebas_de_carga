# Registro de defectos y hallazgos de rendimiento

Curso: Testing y Validación de Software

Proyecto: Taller de Pruebas de Carga y Rendimiento

Fecha de las mediciones: 17 de septiembre de 2026

Equipo: Elioth Thomas Gomez Morales

Este registro usa únicamente las corridas conservadas en
[`perf/results/`](perf/results/). Los SLO del taller son: p95 ≤ 300 ms,
p99 ≤ 800 ms, fallos HTTP < 1 %, resultado de negocio incorrecto < 1 % y
throughput de referencia ≥ 100 req/s.

---

## PERF-01 — Creación de conexiones JDBC por operación

| Campo | Registro |
|---|---|
| Estado | **Resuelto y verificado** |
| Prioridad | Alta (eficiencia y capacidad) |
| Capa afectada | Persistencia / acceso a H2 |
| Escenario que lo evidenció | `load`, rampa de 0 a 200 VUs, antes y después del cambio |
| Resultado esperado | Reutilizar conexiones JDBC para evitar el costo repetido de crearlas y cerrarlas bajo concurrencia. |
| Resultado antes de la corrección | p95 = 11.182 ms, p99 = 25.588 ms, 27,911.30 req/s, 0 % de fallos HTTP y 0 % de fallos de negocio. |
| Resultado después de la corrección | p95 = 8.507 ms, p99 = 12.567 ms, 36,698.65 req/s, 0 % de fallos HTTP y 0 % de fallos de negocio. |

### Evidencia y diagnóstico

Antes del cambio, `RegistryRepository.getConnection()` invocaba
`DriverManager.getConnection(...)` en cada operación. Una petición de registro
puede ejecutar más de una operación de persistencia, por lo que el costo de
crear y cerrar conexiones se repetía bajo carga.

Los artefactos comparados son:

- Antes: [`summary-load-50us.json`](perf/results/summary-load-50us.json).
- Después: [`summary-load-pool.json`](perf/results/summary-load-pool.json).

La corrección incorpora HikariCP 5.1.0 y un `HikariDataSource` con máximo 20
conexiones reutilizables. Frente a la corrida anterior, el p95 disminuyó
23.92 %, el p99 disminuyó 50.89 % y el throughput aumentó 31.48 %, sin
introducir fallos HTTP ni de negocio.

### Validación de la corrección

- `mvn clean verify`: exitoso; 4 pruebas de integración sin fallos.
- `GET /actuator/health`: estado `UP`.
- `POST /register` con una persona válida: respuesta `VALID`.
- Corrida `load` posterior al pool: resultados conservados en
  `summary-load-pool.json`.

---

## OBS-01 — Resolución insuficiente de los percentiles de Actuator

| Campo | Registro |
|---|---|
| Estado | **Resuelto y verificado** |
| Prioridad | Media |
| Capa afectada | Métricas Micrometer / Prometheus |
| Escenario observado | `load` antes del pool |
| Resultado esperado | Obtener un p95 del servidor con resolución útil para compararlo con el p95 de k6. |
| Resultado antes | En una corrida `load` sin pool, Prometheus mostró p50, p95 y p99 como `0.0` s y solo el bucket `+Inf`. |
| Resultado después | Con histogramas desde 1 µs, una corrida `ci` registró p95 de Actuator = 0.425 ms y p95 de k6 = 1.383 ms. |

### Impacto

La latencia de k6 mide la experiencia completa del cliente; Actuator empieza a
medir cuando el servidor toma la solicitud. La diferencia observada fue de
aproximadamente 0.958 ms. En esta corrida corta local no hubo señales de
saturación; bajo load, el defecto PERF-01 permitió atribuir la degradación a la
creación repetida de conexiones JDBC antes de usar HikariCP.

### Próxima acción

Se configuraron límites mínimo de 1 µs y máximo de 1 s para
`http.server.requests`, y se validaron los buckets y percentiles mediante una
instancia limpia en el puerto 8081. La evidencia está en
[`perf/results/observability-ci.md`](perf/results/observability-ci.md).

---

## Resumen de seguimiento

| ID | Hallazgo | Estado | Evidencia principal |
|---|---|---|---|
| PERF-01 | Conexiones JDBC sin reutilización | Resuelto y verificado | `summary-load-50us.json` vs. `summary-load-pool.json` |
| OBS-01 | Resolución insuficiente de percentiles del servidor | Resuelto y verificado | `observability-ci.md` |
