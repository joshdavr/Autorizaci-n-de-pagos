# ADR-001 — Registro previo del intento y estado `PENDIENTE_CONFIRMACION` (autorización síncrona corta, cierre asíncrono)

| | |
|---|---|
| **Estado** | Aceptada |
| **Tipo** | arquitectónica |
| **Fecha** | 2026-10-04 |
| **Ejercicio** | Ejercicio 2 — Autorización de pagos |

## Contexto

El banco es un tercero que puede tardar o no responder, y el socio necesita una respuesta rápida sin riesgo de doble cobro.

## Decisión

Persistir el intento antes de cualquier llamada externa; llamar al banco con timeout de 5 s; si no hay respuesta, responder `202`, conservar la reserva y resolver con consulta de estado y conciliación. Modelo de estados: `RECIBIDO → VALIDADO → AUTORIZADO | RECHAZADO | PENDIENTE_CONFIRMACION → AUTORIZADO | RECHAZADO | REVERSADO`.

## Alternativas consideradas

- Esperar al banco sin límite (rechazada: bloquea hilos y la app)
- Confirmación distribuida 2PC (inviable con un banco externo)
- Todo asíncrono (rechazada: peor experiencia para pagos que el banco resuelve en milisegundos).

## Consecuencias

- (+) cada intento queda trazado, no se cobra dos veces, tolera fallos del banco.
- (−) el cliente puede ver un estado intermedio; exige reconsulta, conciliación y buena comunicación en la app.
