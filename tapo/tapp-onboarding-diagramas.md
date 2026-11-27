# TAPP Onboarding — Diagramas de arquitectura

**Interbank · 23-nov-2026 · Versión anterior**
**Diseño vigente (26-nov-2026):** [2026-11-26/diseno.md](2026-11-26/diseno.md). Base de decisiones: [tapp-onboarding-base-2026-11-26.md](tapp-onboarding-base-2026-11-26.md). Los `.mmd` de esta página son la versión del 23-nov.
**Alcance:** validaciones iniciales al entrar a la zona TAPP, creación del Tapp ID y afiliación de cuentas. Pagos queda fuera.
**Guía aplicada:** [Guia-para-elaborar-diagramas-tecnicos.md](../../Guia-para-elaborar-diagramas-tecnicos.md) y los skills de [.github/skills](../../.github/skills/README.md).

## Entregables

| # | Entregable | Archivo | Estado |
|---|---|---|---|
| 1 | Componentes de alto nivel | [tapp-onboarding-componentes-alto-nivel.mmd](tapp-onboarding-componentes-alto-nivel.mmd) | Borrador. Líneas punteadas = por confirmar |
| 2 | Componentes de bajo nivel | — | No se elabora: no hay módulos ni despliegue del backend definidos todavía (diseño de software) |
| 3 | Modelo de datos | [tapp-onboarding-modelo-datos.md](tapp-onboarding-modelo-datos.md) | Dominio visto por Interbank; persistencia pendiente (P34) |
| 4a | Secuencia: post-login | [tapp-onboarding-secuencia-post-login.mmd](tapp-onboarding-secuencia-post-login.mmd) | Borrador. Depende de P11 |
| 4b | Secuencia: creación del Tapp ID | [tapp-onboarding-secuencia-creacion-tapp-id.mmd](tapp-onboarding-secuencia-creacion-tapp-id.mmd) | Borrador |
| 4c | Secuencia: afiliación de cuenta | [tapp-onboarding-secuencia-afiliacion.mmd](tapp-onboarding-secuencia-afiliacion.mmd) | Borrador. Depende de P2 y P33 |
| 5 | Contratos de API | — | No se elabora: no hay contrato de Izipay ni del backend. Inventarlo violaría la guía |

Las preguntas abiertas que se citan como `Pn` están en [ronda-preguntas-1.md](../../ronda-preguntas-1.md). La versión para presentar, por criticidad, está en [ronda-preguntas-presentacion.md](../../ronda-preguntas-presentacion.md).

## Fuentes

| Fuente | Qué aporta |
|---|---|
| Capturas `imagenes/` (12-nov): PostLogin, Creación, Afiliación, Aplicaciones impactadas | Cajas, numeración de pasos y nombres de servicio `/service/...` |
| Transcripciones `trabajo/transcripciones/parte1..8.md` | Reglas, condiciones, decisiones y correcciones de la revisión |
| [TAPP-onboarding-arquitectura.md](../../TAPP-onboarding-arquitectura.md) v3 | Consolidado y contrastado contra las dos anteriores |

## Decisiones reflejadas en los diagramas

- Toda llamada a Izipay pasa por el **API Gateway Terceros PCI** (AWS IBK). Las llamadas al PAC son **síncronas**.
- La **creación no toca el UPI Switch**; el directorio solo se toca al afiliar la primera cuenta (pasos 7 a 10).
- **Pre-DBFD real-time** bloquea (antes del token y antes de listar cuentas); post-mortem y evidencia (LTD) no.
- El celular se compara contra **SRM**, no contra la sesión.
- El **paso 7 del post-login** lo agregó Interbank para el aviso de "no eres el principal".
- La **réplica de cuentas** se cruza con la consulta integrada en el paso 2 de la afiliación.
- **Notificaciones:** la de creación la envía la billetera (correo); la de vinculación, el proveedor de cuentas.

## Lo que no se dibujó y por qué

| Tema | Motivo |
|---|---|
| Iconos de Azure/AWS, Front Door, APIM, AKS | No hay evidencia de esos componentes para el backend de TAPP; se agregan cuando el diseño de software los confirme |
| Rutas de cada llamada del PAC (base propia, switch u on-us) | P33 |
| Timeouts y reintentos | P10, P12 |
| Biometría u otro factor adicional | Descartado por negocio; pendiente de ciberseguridad (P22) |

## Revisión de la guía

- [x] Alcance y nombre del proyecto claros.
- [x] Cada elemento tiene fuente o está marcado como pendiente (línea punteada o `Pn`).
- [x] Alto nivel y secuencias no mezclan niveles de detalle.
- [x] Nombres de servicios consistentes con las capturas y con el doc v3.
- [x] Cada secuencia describe un caso de uso y termina en un resultado.
- [x] El modelo distingue dominio de persistencia; la persistencia queda pendiente.
- [ ] OpenAPI: no aplica hasta tener contrato.
- [x] Sin credenciales, tokens ni datos personales reales.
- [x] Los cuatro `.mmd` y el modelo se renderizaron sin errores con `mmdc` (Mermaid CLI).
- [x] Decisiones por confirmar anotadas.
