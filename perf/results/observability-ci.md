# Comparación cliente-servidor — escenario `ci`

Fecha: 17 de septiembre de 2026. Se arrancó una instancia limpia del servicio
en el puerto 8081 y se ejecutó `register_voter_k6.js` con 20 VUs durante 60
segundos. El dataset tiene 512 filas y seis resultados esperados: `VALID`,
`INVALID`, `UNDERAGE`, `INVALID_AGE`, `DEAD` y `DUPLICATED`.

| Fuente | p50 | p95 | p99 | Solicitudes | Errores HTTP | Errores de negocio |
|---|---:|---:|---:|---:|---:|---:|
| k6 (cliente) | — | 1.383 ms | 4.239 ms | 11,839 | 0 % | 0 % |
| Actuator (servidor) | 0.179 ms | 0.425 ms | 2.358 ms | 11,839 | — | — |

El p95 del cliente supera al del servidor en aproximadamente 0.958 ms. En este
ambiente local, la diferencia cubre el recorrido HTTP y el tiempo de espera
antes de que el servidor procese la solicitud. No hubo señales de saturación en
esta corrida corta. En las corridas de carga, el defecto PERF-01 identificó la
creación repetida de conexiones JDBC como causa de degradación y HikariCP lo
mitigó.

Para obtener la resolución del servidor se configuró:

```properties
management.metrics.distribution.minimum-expected-value.http.server.requests=1us
management.metrics.distribution.maximum-expected-value.http.server.requests=1s
```
