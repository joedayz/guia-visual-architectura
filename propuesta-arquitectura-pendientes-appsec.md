# Definiciones Arquitectura — Device Integrity (puntos 2, 3, 4, 6)

**Fecha:** 2026-11-17  
**Audiencia:** Arquitectura / APPSEC / APIM / BFF / Móvil / Backends de negocio  
**Alcance:** Definiciones para **empezar implementación**, con conformidad explícita a:

| Plataforma | Tier 1 | Tier 2 |
|------------|--------|--------|
| Android | Revisado | Revisado (foco implementación) |
| Huawei | Revisado | Revisado (foco implementación) |
| iOS | Revisado | Revisado |

**Fuera de alcance de este doc (dueño APPSEC):**  
(1) Timeout `decodeIntegrityToken` — pruebas de latencia / SLO  
(5) Frescura timestamp — validación post-implementación con latencia real  

**Baseline asumido (informe APPSEC V3 + consultas):**

- Android T2 = Play Integrity **Standard** (no Classic).
- Android/Huawei = **sin enroll**; evidencia por operación.
- iOS = enroll + reconcile (`CHECK`/`VERIFY`) + App Attest.
- Huawei T2 = nonce/`requestId` generados por la **app**; anti-replay vía Redis `SET NX`.
- Challenges T1 = servidor, TTL 120 s, un solo uso.
- `SKIPPED` ≠ evidencia válida ≠ `ALLOW`.
- Sin contingencia DEGRADED aprobada y desplegada → **rechazo**.

---

## Mapa de conformidad (lectura rápida)

| Punto | Android T1 | Android T2 | Huawei T1 | Huawei T2 | iOS T1 | iOS T2 |
|-------|------------|------------|-----------|-----------|--------|--------|
| **2 Redis / anti-replay** | Challenge en Redis (emitir/consumir) | Anti-replay del **Standard** (Google); Redis **no** es el store de nonce de op. | Challenge en Redis | **Redis obligatorio** (`SET NX` nonce/`requestId`) | Challenge + counter en Cosmos; Redis para challenge lifecycle | Assertion + counter Cosmos; challenge solo ENROLL/RECOVER |
| **3 HMAC APIM→backend** | **Obligatorio** | **Obligatorio** | **Obligatorio** | **Obligatorio** | **Obligatorio** | **Obligatorio** |
| **4 Pre-login** | N/A (no enroll); `initiate` con JWT de sesión | N/A; eval vía APIM negocio | N/A; `initiate` con JWT | N/A; eval vía APIM negocio | **Bootstrap token** en `enroll`/`reconcile`/`initiate` ENROLL|RECOVER | Igual pre-login para ciclo de vida; ops T2 con JWT de sesión + assertion |
| **6 `SKIPPED_LEGACY_CLIENT`** | Sí (ventana MONITOR) | Sí, **más estricto** (post-corte → 426/DENY) | Sí (ventana MONITOR) | Sí, **más estricto** | Sí (diseño actual) | Sí, **más estricto** en ops protegidas |

---

## Punto 2 — Redis, nonce y anti-replay (todas las plataformas/tiers)

### 2.1 Principio

| Concepto | Definición |
|----------|------------|
| Anti-replay de **evidencia** | Impedir reutilizar token/JWS/assertion ya presentada |
| Anti-replay de **challenge** | Challenge de servidor: un solo uso, TTL, purpose/contexto |
| Idempotencia de **negocio** | Distinta; no sustituye anti-replay |

### 2.2 Matriz por plataforma y tier

#### Android

| | Tier 1 | Tier 2 |
|---|--------|--------|
| Origen del vínculo | Challenge servidor (`initiate`) | `requestHash` ligado a método\|ruta\|SHA256(cuerpo)\|deviceId\|timestamp |
| Store anti-replay | **Redis:** emitir challenge (`SET` + TTL) y consumir atómico al evaluar | **Play Integrity Standard** (anti-replay del proveedor). Redis **no** guarda nonce de la operación T2 |
| Si Redis cae | **DENY / 503–504**; no asumir consumo correcto del challenge | No bloquea por Redis de nonce; sí aplica política de dependencia Integrity/Google (APPSEC timeout/SLO). Si hay challenge residual (ENROLL N/A) → N/A |
| Despliegue multi-región | Challenges visibles en la región que emite y consume (sticky o Redis compartido del ciclo T1) | No gate Redis por nonce; gate = cuota/egress Google + política ENFORCE |

**Hash canónico (implementación):**

- T1: `requestHash = SHA256(UTF8(challenge | payload_canónico))`
- T2: `requestHash = SHA256(método | ruta | SHA256(cuerpo) | deviceId | timestamp)`

#### Huawei

| | Tier 1 | Tier 2 |
|---|--------|--------|
| Origen del vínculo | Challenge servidor (`initiate`), TTL 120 s, un uso | Nonce + `requestId` generados por la **app** |
| Store anti-replay | **Redis:** challenge emit/consume atómico | **Redis `SET NX`:** `integrity:nonce:huawei:{requestId}` (o hash del nonce) |
| Si Redis cae | **DENY / 503–504**; no asumir consumo | **DENY / 503–504**; no asumir nonce nuevo. Fallback DEGRADED solo si aprobado y desplegado |
| Despliegue incremental multi-región | Igual que Android T1: Redis del ciclo challenge listo en región | Ver §2.3 (gate ENFORCE) |

#### iOS

| | Tier 1 | Tier 2 |
|---|--------|--------|
| Origen del vínculo | Challenge ENROLL/ops + `clientDataHash`; counter App Attest | Assertion ligada a petición (`clientDataHash`); counter atómico en Cosmos |
| Store anti-replay | Redis (challenge) + **Cosmos** (keyId, counter, historial) | Cosmos counter; challenge Redis solo para ENROLL/RECOVER, **no** en flujo base T2 de ops |
| Si Redis cae | No emitir/consumir challenge → rechazo ciclo de vida / T1 challenge | Ops T2 sin challenge de servidor: Redis down no abre bypass; fallos Cosmos → rechazo (no omitir counter) |
| Si Cosmos cae | No confirmar enroll/counter → rechazo | No omitir historial/counter como válidos → rechazo |

**Hash canónico (implementación):**

- T1: `clientDataHash = SHA256(UTF8(challenge | canonicalPayload))`
- T2: `datosCanónicos = método | ruta | SHA256(cuerpo) | deviceId | timestamp` → `clientDataHash = SHA256(UTF8(datosCanónicos))`

### 2.3 Estrategia Redis Huawei Tier 2 (despliegue incremental) — definición cerrada

**Objetivo:** garantizar anti-replay del nonce/`requestId` entre regiones.

| Decisión | Valor |
|----------|-------|
| Modelo | **Store de nonces con visibilidad cross-region** antes de ENFORCE multi-región (Redis Active-Active o primary único + routing sticky a la región writer) |
| Clave | `integrity:nonce:huawei:{requestId}` |
| Operación | `SET key value NX EX <ttl>` |
| TTL | `max_age_frescura (120s propuesto) + skew (30s) = 150s` (ajustable tras mediciones APPSEC punto 5) |
| Valor | `region | epoch_ms | correlationId` (solo telemetría; la unicidad la da la clave) |
| Colisión NX | Tratar como **replay** → `DENY` / 403 (reason de replay), nunca DEGRADED |
| Redis ausente en región | Feature flag **Huawei T2 ENFORCE = OFF** en esa región; tráfico T2 Huawei → rechazo controlado o no enrutar ops protegidas |
| Gate de go-live regional | Checklist: Redis healthy + prueba `SET NX` cross-region + alarma de memoria/cardinalidad + runbook |
| Fallback | **No** hay fallback permisivo por defecto. DEGRADED solo con caso aprobado, restricciones desplegadas y decisión **HMAC-autenticada** |

**Android T2:** no usa este store; no condicionar el gate Redis de Huawei al rollout Android.  
**Android/Huawei T1 e iOS challenges:** mismo Redis de challenges puede ser el cluster, **namespace de claves distinto** (`integrity:challenge:...` vs `integrity:nonce:huawei:...`).

### 2.4 Política de fallo Redis (alineada a §5.1 informe)

| Escenario | HTTP (sin contingencia) | T1 | T2 |
|-----------|-------------------------|----|----|
| Redis no disponible al emitir/consumir challenge o registrar nonce | 503 / 504 | Rechazar; no asumir consumo | No asumir nonce nuevo; sin controles desplegados → rechazar |
| Challenge inexistente / vencido / otro purpose / ya consumido | 403 | DENY | N/A en flujo base T2 ops; sí aplica a ENROLL/RECOVER iOS |
| `requestHash` / nonce / `clientDataHash` ≠ reconstruido | 403 | DENY | DENY; nunca contingencia permisiva |
| Replay detectado (Standard Google / NX Huawei / counter iOS) | 403 | DENY | DENY |

---

## Punto 3 — HMAC APIM ↔ backend (contrato de verificación)

### 3.1 Principio

- HMAC es **requisito** (deja de ser “sugerencia”).
- Autentica la **decisión Integrity** que APIM entrega al **backend de la operación de negocio**.
- Aplica a **todas** las plataformas y **ambos** tiers.
- `DEGRADED` y `SKIPPED` también deben ir **autenticados**.
- APIM **elimina** cualquier header de decisión/firma enviado por el cliente y **regenera** solo tras evaluación autorizada.

### 3.2 Dónde aplica por flujo

| Flujo | ¿HMAC? | Quién firma | Quién verifica |
|-------|--------|-------------|----------------|
| Android/Huawei/iOS **Tier 2** op. de negocio (`evaluate/transaction` vía fragmento APIM → Integrity → backend) | Sí | APIM (post-Integrity) | Backend de negocio |
| Android/Huawei/iOS **Tier 1** op. protegida vía BFF/APIM | Sí | APIM | Backend de negocio |
| iOS `enroll` / `reconcile` / `initiate` (ciclo de vida) | HMAC **no** sustituye auth pre-login; la respuesta al app no lleva “decisión de negocio”. Si un backend interno consume resultado de enroll, firmar contrato interno APIM↔BFF/Integrity con mTLS; HMAC de decisión de **negocio** N/A hasta ops |
| Postura `evaluate/posture` | Si el backend actúa sobre restricciones → Sí | APIM | Backend consumidor |

### 3.3 Contrato (implementación)

**Headers (APIM → backend):**

```
X-Integrity-Decision: ALLOW | DENY | DEGRADED | SKIPPED | UNVERIFIED | MONITOR
X-Integrity-Reason: <reason_code>
X-Integrity-Platform: ANDROID | IOS | HUAWEI
X-Integrity-Tier: 1 | 2
X-Integrity-Operation: <catalog_op_id>
X-Integrity-Device-Id: <deviceId>
X-Integrity-Ts: <epoch_ms>
X-Integrity-Nonce: <requestId | challengeId | ->
X-Integrity-Restrictions: <"" | csv/json compacto>
X-Integrity-Signature: base64url(HMAC-SHA256(secret, string_to_sign))
```

**`string_to_sign` (orden fijo, `|`):**

```
decision|reason|platform|tier|operation|deviceId|ts|nonce|restrictions|method|path|sha256(body)
```

**Secret:** Azure Key Vault; dual-key `current` + `previous` (gracia p. ej. 24 h); rotación documentada.

### 3.4 Verificación en backend (todas las plataformas/tiers)

| Caso | Acción | HTTP típico al cliente (sin filtrar internos) |
|------|--------|-----------------------------------------------|
| Signature ausente | No ejecutar negocio; alertar | 403 |
| Signature inválida | No ejecutar; métrica `integrity.hmac.invalid` | 403 |
| `X-Integrity-Ts` fuera de ventana (p. ej. 120s + skew 30s) | Tratar como vencida / replay de decisión | 403 |
| Secret KV indisponible / no se puede autenticar decisión | No invocar lógica de negocio | 503 |
| `DENY` / `UNVERIFIED` sin step-up aprobado | No ejecutar | Según 5.1 / contrato móvil |
| `DEGRADED` | Ejecutar **solo** si restrictions ∈ catálogo aprobado y desplegado | HTTP real del backend + restricciones |
| `SKIPPED` | No es ALLOW; aplicar política legacy/excepción (§4) | Según política |
| `ALLOW` / `MONITOR` | Continuar; MONITOR ≠ éxito de evidencia fallida | HTTP real del backend |
| Headers falsificados por el cliente | APIM los strippea; backend solo confía en lo firmado por APIM | N/A (no llegar) |

**Conformidad Tier 2 Android/Huawei:** el backend de la operación **debe** verificar HMAC antes de debitar/transferir/etc.; no basta con “vino por APIM”.

---

## Punto 4 — Autorización pre-login (iOS) y contraste con Android/Huawei

### 4.1 Alcance

| Plataforma / flujo | ¿Pre-login? | Mecanismo |
|--------------------|-------------|-----------|
| iOS `POST /deviceIntegrity/ios/enroll` | **Sí** | Bootstrap token (§4.2) |
| iOS `POST /deviceIntegrity/ios/reconcile` | **Sí** | Bootstrap token; `purpose` distinto |
| iOS `POST /deviceIntegrity/initiate` con purpose ENROLL/RECOVER | **Sí** | Mismo bootstrap o challenge ligado al token |
| iOS ops Tier 1/2 de negocio | **No** (post-login) | JWT de sesión bancaria + assertion |
| Android `initiate` / evaluate | **No enroll** | JWT de sesión (T1) o contexto APIM de la op. (T2) |
| Huawei `initiate` / evaluate | **No enroll** | Igual que Android |

### 4.2 Mecanismo iOS — Bootstrap token (definición para implementar)

**Nombre lógico:** `device_integrity_bootstrap`  
**Emisor:** BFF device-integrity o IdP (cerrar owner en kickoff; default propuesto: **BFF** tras `initiate` o endpoint dedicado de bootstrap).  
**No es** JWT de banca ni sesión de cliente.

**Claims mínimos:**

```json
{
  "purpose": "device_integrity_enroll | device_integrity_reconcile | device_integrity_recover",
  "platform": "IOS",
  "deviceId": "<id confiable>",
  "jti": "<uuid>",
  "iat": 0,
  "exp": 0
}
```

| Parámetro | Valor propuesto |
|-----------|-----------------|
| TTL | 5–10 minutos |
| Uso | Rate-limited por `deviceId` + IP + global en APIM |
| Binding | `deviceId` del token = `deviceId` del body/challenge |
| Rutas | `purpose` debe coincidir exactamente con el endpoint |

**APIM (antes de BFF):**

1. Validar firma bootstrap (JWKS / KV).
2. Validar `purpose`, `exp`, `platform=IOS`.
3. Rate limit pre-login; si excede → **429**, no llamar BFF, no emitir challenge.
4. Sin token / inválido / purpose incorrecto → **403** (ciclo de vida o reemplazo sin contexto confiable).

**Integrity / BFF:**

- Asociar challenge ENROLL al `keyId`/`deviceId` del token.
- No sobrescribir key existente; reemplazo solo con autorización controlada.
- `CHECK` ≠ enroll confirmado; `VERIFY` exige proof of possession + challenge RECOVER.
- Tras login: vínculo atómico `customerId` ↔ `deviceId`/`keyId` en Cosmos.

### 4.3 Conformidad Android / Huawei (para no inventar enroll)

- No existen `enroll`/`reconcile` Android/Huawei → **no** reutilizar bootstrap iOS.
- Primera evidencia = primer `evaluate` (T2) o `initiate`+evaluate (T1), siempre bajo auth de sesión/op. de negocio.
- Rate limits de `initiate` Android/Huawei usan identidad autenticada (no pre-login).

### 4.4 iOS Tier 1 vs Tier 2 (ciclo de vida vs operación)

| | Tier 1 / ciclo de vida | Tier 2 operación |
|---|------------------------|------------------|
| Auth | Bootstrap pre-login para enroll/reconcile | JWT sesión + assertion |
| Challenge | Sí (ENROLL/RECOVER/ops T1 según catálogo) | Flujo base ops: sin challenge de servidor (riesgo aceptado en diseño); challenge solo ENROLL/RECOVER |
| Counter | Persistido en enroll; actualizado en assertions | Counter atómico obligatorio |
| HMAC decisión negocio | N/A en enroll; sí en ops T1 de negocio | Sí |

---

## Punto 6 — `SKIPPED_LEGACY_CLIENT` (por plataforma y tier)

### 6.1 Semántica (única)

- Reason code: `SKIPPED_LEGACY_CLIENT`.
- Significa: evaluación **omitida por excepción de transición** (cliente antiguo / sin capacidad de evidencia).
- **No** implica evidencia válida.
- Debe viajar como decisión **HMAC-autenticada** si impacta backend.
- Ventana temporal + versión mínima + dueño + fecha de fin; sin eso no hay SKIPPED “eterno”.

### 6.2 Matriz de aplicación

| Plataforma | Tier | ¿Aplica? | Detección | Fase MONITOR | Fase ENFORCE / post-corte |
|------------|------|----------|-----------|--------------|---------------------------|
| **iOS** | T1 | **Sí** | App &lt; versión mínima App Attest; o flujo legacy sin assertion con historial/política | SKIPPED + telemetría; no autoriza op. protegida como si fuera ALLOW | 426 `UPDATE_REQUIRED` o DENY según contrato móvil; step-up solo si aprobado |
| **iOS** | T2 | **Sí (más estricto)** | Igual + op. de alto riesgo sin assertion | SKIPPED solo con switch/excepción vigente | Preferir **426/DENY**; SKIPPED solo kill-switch auditado |
| **Android** | T1 | **Sí** | App &lt; mín. Play Integrity; ausencia de evidencia/headers de integridad | SKIPPED en ventana; APIM clasifica ausencia **sin** omitir JWT/plataforma/anti-downgrade | 426 o DENY; ENFORCE gradual |
| **Android** | T2 | **Sí (más estricto)** | Cliente sin Standard / sin `requestHash` válido en op. catalogada | SKIPPED solo MONITOR o excepción | **426/DENY** por defecto; no DEGRADED por legacy |
| **Huawei** | T1 | **Sí** | App/HMS Core &lt; mín. SysIntegrity (ref. inicial 4.0.0.300 a validar); JWS incompleto | SKIPPED + informar update; no silencio | 426/DENY |
| **Huawei** | T2 | **Sí (más estricto)** | Sin nonce/`requestId` o sin SysIntegrity en op. protegida | SKIPPED solo MONITOR/excepción | **426/DENY**; gate Redis no se bypasea con SKIPPED |

### 6.3 Reglas transversales de implementación

1. Publicar versión móvil con evidencia → medir MONITOR con fecha fin → fijar versión mínima → 426 a incompatibles → ENFORCE gradual (§5.4).
2. No deducir versión desde header inexistente.
3. Switch `false` explícito, autorizado y vigente → SKIPPED autenticado + auditoría; backend aplica política de excepción (no “allow all”).
4. Android/Huawei: legacy ≠ “nunca enrolado”; se detecta por **versión + evidencia**, no por `keyId`.
5. iOS: ausencia de `keyId` local **no** prueba que nunca hubo enroll; consultar historial/reconcile.

### 6.4 Distinguir legacy de otros motivos

| Situación | Reason / tratamiento |
|-----------|----------------------|
| Cliente antiguo en ventana | `SKIPPED_LEGACY_CLIENT` |
| Downgrade / tamper de headers | **DENY** (no SKIPPED) |
| Proveedor caído | Indisponibilidad / DEGRADED solo si aprobado |
| Replay / hash mismatch | **DENY** |
| Kill switch off | SKIPPED autenticado (motivo de switch), no legacy |

---

## Checklist de arranque de implementación (Tier 2 Android + Huawei primero)

### Android Tier 2

- [ ] Standard API + `requestHash` canónico T2
- [ ] Fragmento APIM → Integrity `evaluate/transaction`
- [ ] HMAC obligatorio en backend de la op.
- [ ] Política legacy T2 (MONITOR → 426/DENY)
- [ ] Sin dependencia de Redis nonce (no bloquear por gate Huawei)
- [ ] Cuota/timeout Google: coordinar con APPSEC (puntos 1 y 5)

### Huawei Tier 2

- [ ] Nonce/`requestId` app-generated + validación de entropía/formato
- [ ] Redis `SET NX` + TTL + métricas
- [ ] Gate regional ENFORCE (Redis ready)
- [ ] HMAC obligatorio en backend
- [ ] Legacy T2 estricto
- [ ] Sin Redis → DENY (no bypass)

### Android / Huawei Tier 1 (paralelo o fase 2)

- [ ] `initiate` challenge Redis TTL 120 s consumo atómico
- [ ] Evaluate con challenge consumido
- [ ] HMAC en ops de negocio T1
- [ ] Legacy T1 en MONITOR

### iOS Tier 1 y 2

- [ ] Bootstrap pre-login en enroll/reconcile
- [ ] Cosmos keyId/counter/historial
- [ ] T2 assertion + counter; challenge solo ENROLL/RECOVER
- [ ] HMAC en ops de negocio
- [ ] Legacy + anti-downgrade / historial

---

## Decisiones fijadas (para que el team implemente)

1. **Redis Huawei T2:** store cross-region + `SET NX`; gate ENFORCE por región; sin Redis = DENY.  
2. **HMAC:** requisito universal T1/T2 en decisión hacia backend de negocio; ausente/inválida/vencida = 403; KV dual-key.  
3. **Pre-login:** solo iOS enroll/reconcile/initiate ENROLL|RECOVER vía bootstrap token; Android/Huawei no aplican enroll.  
4. **`SKIPPED_LEGACY_CLIENT`:** aplica a **Android, Huawei e iOS**; T2 más estricto que T1; post-corte → 426/DENY salvo excepción auditada.

## Pendientes menores (no bloquean el diseño; sí el go-live ENFORCE)

| Ítem | Owner |
|------|-------|
| Emisor exacto del bootstrap (BFF vs IdP) | Arquitectura + BFF |
| Valores numéricos rate limit / cardinalidad Redis | APPSEC + APIM + Capacidad |
| Versión mínima app/HMS por plataforma y fechas MONITOR→ENFORCE | Móvil + APPSEC + Arquitectura |
| Timeout Google y ajuste ventana frescura 120s/30s | APPSEC (1 y 5) |
| Secret HMAC: uno global vs por dominio | Arquitectura + Seguridad |

---

## Referencias

- `propuesta_arquitectura_appsec_v3` — §§4.1–4.2, 5.1–5.4, 6.x, glosario, 6.6  
- Consultas APPSEC (latencia Google, Redis Huawei T2, HMAC, pre-login, legacy)  
- Respuesta team: puntos 2/3/4/6 → Arquitectura; 1/5 → APPSEC  
`)