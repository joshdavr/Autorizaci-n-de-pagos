# ADR-002 — Idempotencia con clave única en PostgreSQL (no en una caché)

| | |
|---|---|
| **Estado** | Aceptada |
| **Tipo** | tecnológica |
| **Fecha** | 2026-10-04 |
| **Ejercicio** | Ejercicio 2 — Autorización de pagos |

## Contexto

El cliente móvil reintenta ante redes inestables; un reintento nunca puede cobrar dos veces.

## Decisión

La app genera un UUID por intención de pago y lo envía como `Idempotency-Key`. Una restricción `UNIQUE (socio_id, idempotency_key)` más el hash del cuerpo decide si la solicitud es repetida, conflictiva o nueva, dentro de la misma transacción que registra el intento.

## Alternativas consideradas

- `SETNX` en Redis (rechazada: componente adicional, volátil, puede divergir del estado real)
- Confiar solo en la deduplicación del banco (rechazada: no cubre rechazos previos ni límites internos).

## Consecuencias

- (+) garantía fuerte y simple, sin infraestructura nueva.
- (−) la tabla de claves crece: se define retención (p. ej. 30 días) y limpieza programada.
