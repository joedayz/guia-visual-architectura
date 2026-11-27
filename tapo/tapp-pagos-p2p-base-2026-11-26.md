# TAPP — Base de diseño: instrucción de pago P2P

**Interbank · 26-nov-2026**
**Aporte:** arquitectura de canal (José). Alcance dibujado: alto nivel, bajo nivel, modelo de datos y cuatro secuencias de la instrucción de pago.
**Estado de los dibujos:** están en `docs/architecture/p2p-2026-11-26/`. Este archivo explica qué se dibujó y por qué. No inventa contratos ni cajas que el PDF o la sala no respalden. Las historias de usuario todavía no llegan. Cuando lleguen, se ajusta la secuencia del paso que contradigan.

Guía de los entregables: [Guia-para-elaborar-diagramas-tecnicos.md](../../Guia-para-elaborar-diagramas-tecnicos.md). Índice de los dibujos: [p2p-2026-11-26/diseno.md](p2p-2026-11-26/diseno.md).

---

## 1. Cómo usar este archivo

1. Los nombres de servicio del **deck** mandan sobre el audio. La sala confirmó cuatro: `QRValidationRequest`, `ValidateVPAAddress`, `contactSwipeRequest` y `PaymentRequest`.
2. La **explicación de sala** manda sobre la lámina cuando Malena corrige un paso. El caso ya cerrado es el límite: la lámina 8 pone el acumulado en el ingreso de monto, y la sala lo mueve a la instrucción.
3. No hay historias. Hasta que lleguen, el mock manda en lo que ve el cliente y la sala manda en la regla. Una historia futura no reabre lo que la sala cerró y ella no mencione.
4. Lo que no está cerrado se dibuja punteado. La lista, y quién la cierra, está solo en [pendientes.md](p2p-2026-11-26/pendientes.md).

---

## 2. Alcance

Entra un proceso: instruir un pago inmediato UPI desde el APP Personas. Dos lecturas, TAPP a TAPP y TAPP a teléfono. El teléfono entra por número, por contacto o por QR.

El canal hace cuatro cosas, y cada una tiene su secuencia:

1. Resolver al beneficiario.
2. Autorizar e instruir si la cuenta cargo es Interbank.
3. Autorizar e instruir si la cuenta cargo es de otro banco.
4. Cerrar: callback, pantalla, constancia y favorito.

Queda fuera:

- Interior del PAC, del UPI Switch y del Proveedor de Cuentas. El deck los pide como referencia. On-us y off-us ya se diseñaron ahí.
- Onboarding, device binding y el home de TAPP. Este pago los consume. Las cuentas y la consulta integrada ya corrieron.
- P2M, comercios y POS.
- Alta de no clientes. La sala lo dejó para un paso posterior, fuera de la volumetría de esta salida.
- MPA. La arquitectura es transitoria y sin MPA.
- Administración de favoritos. Aquí solo se consulta el tope y se ofrece el alta desde la constancia, contra la aplicación de One App.
- El buscador general y la pantalla de Plin. Salieron en la misma sesión y no son pasos del pago. Quedan nombrados en el pendiente 9.

Canal: App Personas, Android e iOS. Público: cliente Interbank que ya entró a la zona TAPP.

Roles del deck: Interbank es TPAP y Proveedor de Cuentas. El PSP es Izipay. El switch es del BCRP.

---

## 3. Fuentes

| Fuente | Qué fija |
|---|---|
| `TPAP - Pagos.pdf`, 21 páginas. Fotos en `p2p/`. Lectura en [documento-tpap-pagos.md](p2p-2026-11-26/documento-tpap-pagos.md) | Zonas, roles, premisas, siete pasos, mocks y los nombres de la lámina 18 |
| Reunión, `parte8.MOV`. Lectura en [explicacion-malena.md](p2p-2026-11-26/explicacion-malena.md). Transcripción en `trabajo/transcripciones/parte8-p2p.md` | Correcciones de sala: límite, CIAM, callback, pantallas, favoritos |
| Marco de seguridad de la Plataforma. Lectura en [seguridad-bcrp.md](p2p-2026-11-26/seguridad-bcrp.md) | Common Library, firma XML, lo que el canal no persiste |
| Onboarding ya cerrado en [tapp-onboarding-base-2026-11-26.md](tapp-onboarding-base-2026-11-26.md) | Home, dispositivo y Tapp ID. Este pago no los redibuja |

Presenta Malena. José ve el canal. La página 21 del PDF de pagos no está en las fotos. La 20 es el cierre.

---

## 4. Premisas que el dibujo ya toma

Texto de la lámina 6, más lo que la sala cerró encima:

- La asociación de cuentas a TAPP parte del teléfono de **$RM**. El pago no vuelve a afiliar.
- Estas operaciones se suman al ecosistema **TAPP (CP4I)** y **PLIN (CICS)**. El CICS de Plin se usa por ahora. El C2C va al clúster nuevo. Hay catálogo y recursos de integración exclusivos de TAPP.
- La arquitectura es **transitoria y sin MPA**.
- Favoritos queda fuera del detalle de este entregable. El soporte técnico del botón de la constancia sí entra: la aplicación de favoritos de One App.
- Este entregable no detalla al Proveedor de Cuentas ni al PAC.
- El canal no absorbe lógica de TAPP. Deja login, factores, evidencia, lectura de QR, la consulta de beneficiario y los límites que ya conoce.
- Hay tres capas de límites. El dibujo solo nombra la del canal. Por transacción: **S/ 500**, en el canal, para cortar antes de que el BCRP o el PAC rechacen el pago. Acumulado: **BPI HOST**, antes de `PaymentRequest`.
- El público es cliente con producto Interbank.

El pago empieza con las cuentas que el home ya recuperó. No es una consulta nueva de afiliación.

---

## 5. Catálogo que usan los diagramas

Toda consulta al PAC es síncrona y pasa por el API Gateway de terceros PCI. `PaymentRequest` se dibuja síncrono hacia el PAC y asíncrono por detrás. La respuesta de negocio vuelve por callback.

### 5.1 Consulta del beneficiario

| Llave | Servicio | Qué devuelve |
|---|---|---|
| QR | `QRValidationRequest` | Beneficiario. Falta el campo que dice si el QR lo generó TAPP |
| TAPP ID | `ValidateVPAAddress` | Beneficiario ya resuelto |
| Teléfono o contacto | `contactSwipeRequest` y después `ValidateVPAAddress` | Primero los directorios. Luego, sobre el directorio elegido, la persona |

El contacto es un teléfono. Elegir directorio solo ocurre en esa llave. Elegir TAPP es TAPP a TAPP. Elegir Plin, Yape u otro es TAPP hacia fuera. Esa bifurcación la resuelve el PAC. El canal no la orquesta.

### 5.2 Instrucción con cuenta Interbank

| Momento | Dónde | Efecto |
|---|---|---|
| Cruce de cuentas | Backend | Consulta integrada del home contra cuentas afiliadas al TAPP ID. En gris: inactiva, bloqueada, sin saldo o modo calle |
| Monto | Backend | Límite por transacción, S/ 500 |
| Afiliaciones | Backend | Push, token digital y notificaciones nativas. Si falta una, no se avanza |
| MFA | CIAM | Directo, sin MPA |
| 5.2 | PRE DBFD | Real time, síncrono. Si no aprueba, no se avanza |
| 5.3 | PRE DBFD | Post mortem si el real time rechaza, y también al cierre cuando ya hay éxito o falla |
| Acumulado | BPI HOST | Antes de instruir |
| Pago | PAC | `PaymentRequest`. On-us: el PAC habla directo con el Proveedor de Cuentas |

Quién origina el real time quedó abierto. La lámina 18 lo entra a PRE DBFD. La sala pide el MFA a CIAM y dice que CIAM envía esas tramas. La secuencia dibuja las dos llamadas y el pendiente 3 guarda la flecha que falta.

### 5.3 Instrucción con cuenta de otro banco

No hay API de PIN. La Clave Tapp la captura la Common Library, no se guarda, y viaja en el mismo `PaymentRequest`. El PAC responde éxito, falla o bloqueo por intentos.

Off-us: el BCRP enruta cargo y abono. El canal no llama al switch ni a la CCE.

No se dibuja un OTP. El pendiente 1 es si el reglamento SBS pide un segundo factor además del PIN.

### 5.4 Cierre

| Resultado | Pantalla | Qué sigue en el backend |
|---|---|---|
| Callback de éxito mientras el canal espera | Tappeo exitoso | Post mortem monetario de éxito |
| El PAC responde en proceso | Tappeo en proceso | El callback posterior completa los avisos |
| Vence el timeout del canal | Tappeo enviado | Igual. No se pinta éxito |
| El PAC responde fallido | Solo el mensaje de error, sin referencia | Post mortem monetario fallido |

El tope del ecosistema que nombró la sala es 45 segundos. El timeout del canal es menor y no tiene número. El ejemplo hablado fue diez segundos.

Si el cliente cierra la app, el hilo no se corta. El backend ya tiene el pedido y completa constancia, evidencia y post mortem.

Constancia por correo, vía SMTP, solo cuando el cargo no usó cuenta Interbank. El texto remite el detalle a la entidad que debitó. La alerta de cargo y de abono la emite el Proveedor de Cuentas. No sale del canal.

Evidencia digital hacia IAP, sin nombre de servicio. Favoritos: consulta de tope y, si no llegó, el botón de alta. Las dos operaciones están sin nombre.

---

## 6. Reglas de sala que el diseño muestra

No hay DHU en este paquete. Estas son las reglas que las secuencias ya enseñan.

- El monto se corta en el canal, en S/ 500, antes de elegir cuenta.
- El acumulado no se valida en la pantalla de monto.
- Una cuenta en gris no se puede elegir. El modo calle lo resuelve el app.
- Cuenta Interbank: sin push, sin token o sin notificaciones nativas, no se avanza. Sin aprobación del real time, tampoco.
- Cuenta de otro banco: la Clave Tapp va en `PaymentRequest`. No hay segunda llamada.
- Cerrar la app no borra el pedido. La sesión del teléfono no es el único lugar donde vive.
- Fallido de PAC no arma constancia. Otro estado, el de cargo sin abono, se regulariza en conciliación y no se pinta como fallido.
- El botón de favorito se oculta si la persona ya llegó al tope.

---

## 7. Qué se decidió al dibujar, frente a la lámina

| La lámina decía | Qué quedó en el diagrama |
|---|---|
| Límite por transacción y acumulado, los dos en el paso 2 | El acumulado se movió a la instrucción, contra BPI HOST |
| MFA, push, token y fraude, sin decir CIAM | El MFA entra directo a CIAM porque no hay MPA |
| `PaymentRequest` síncrono | Síncrono de cara al canal, asíncrono por detrás, con callback al backend |
| Consultar y crear favorito en la lámina 18, y la premisa que los saca | Solo tope y alta, contra Favoritos de One App, en punteado |
| Pasos 9 y 10 de favoritos como parte del pago | La regla de la zona TAPP no se diseña |
| HMAC como única consideración de la lámina 19 | Los dibujos usan la Common Library, que es lo que la sala operó. El HMAC no se dibuja |

Se conservan tal como la lámina y la sala coinciden: API Gateway de terceros PCI en el camino al PAC, Common Library en el app, tres capas de límites con solo la del canal en estos dibujos, y el Proveedor de Cuentas como emisor de la alerta de cargo y abono.

---

## 8. Pendientes que no se dibujan como hechos

El detalle y el responsable están en [pendientes.md](p2p-2026-11-26/pendientes.md).

- Segundo factor además de la Clave Tapp cuando la cuenta no es Interbank.
- Nombre del callback, patrón de red y número del timeout. También si un PIN rechazado vuelve en la respuesta de `PaymentRequest` o en el callback.
- Si el real time lo dispara CIAM o el backend hacia PRE DBFD.
- Nombre de la operación de tope y de alta, y si el alta pide OTP.
- Campo de la trama que identifica un QR generado por TAPP.
- En qué momento del app ya están saldo, bloqueo y modo calle.
- Dónde se retiene el pedido en curso.
- Si una fallida aparece en el historial. Fuera de esta pantalla.
- Cuándo se consulta la lista de PSP del buscador. Fuera de este pago.
- HMAC de la lámina 19 frente a la firma XML.

---

## 9. Qué contiene cada diagrama

Los siete archivos ya están dibujados. No hay contrato OpenAPI. No se inventaron campos.

### Alto nivel

[componentes-alto-nivel.mmd](p2p-2026-11-26/componentes-alto-nivel.mmd). Sistemas y zonas, sin el detalle de cada servicio.

- Canal: cliente y App Personas, con el SDK UPI CL.
- Azure: backend del app. Orquesta la instrucción y conserva el pedido.
- AWS: API Gateway de terceros PCI. Es la puerta al PAC.
- On-premise: CIAM, PRE DBFD, BPI HOST y SMTP.
- Izipay: PAC.
- Fuera del canal, en sólido porque la sala lo dijo y el canal no los llama: Proveedor de Cuentas (on-us y alertas) y BCRP con CCE (off-us).
- Punteado: Favoritos de One App e IAP. Están nombrados y no tienen servicio.

### Bajo nivel

[componentes-bajo-nivel.mmd](p2p-2026-11-26/componentes-bajo-nivel.mmd). Lo que el canal ejecuta.

- App: pantallas del pago, lectura de QR, Common Library para la firma y para capturar la Clave Tapp.
- Backend: orquestación, el S/ 500, el cruce de cuentas, el timeout de pantalla, y el pedido en curso.
- Cada llamada al PAC con su nombre, síncrona. `PaymentRequest` lleva la nota de que por detrás el PAC es asíncrono.
- El callback entra al pedido en curso, punteado en el nombre de la operación.
- Favoritos e IAP punteados. El almacén del pedido también: el comportamiento está cerrado y el sitio no.

### Modelo de datos

[modelo-datos.md](p2p-2026-11-26/modelo-datos.md). Conceptos del canal durante la instrucción. Sin esquema físico.

- `InstruccionEnCurso`: llave, monto, directorio, si la cuenta cargo es IBK, estado de pantalla, `msgId` y `ts`.
- Beneficiario, directorio destino y cuenta cargo. El directorio solo existe si la llave fue un teléfono.
- Estados de pantalla: exitoso, en proceso, enviado, fallido.
- Límite del canal en dos controles. El acumulado no es un dato de la instrucción: vive en BPI HOST.
- Clave Tapp, OTP y PAN no se modelan como datos guardados.

### Secuencias

Una por caso. Nombres de la sección 5. Ramas de la sección 6.

1. [secuencia-consulta.mmd](p2p-2026-11-26/secuencia-consulta.mmd). QR, TAPP ID, y teléfono con las dos llamadas.
2. [secuencia-instruccion-ibk.mmd](p2p-2026-11-26/secuencia-instruccion-ibk.mmd). Gris, S/ 500, afiliaciones, CIAM, real time, acumulado y `PaymentRequest`.
3. [secuencia-instruccion-externa.mmd](p2p-2026-11-26/secuencia-instruccion-externa.mmd). La misma instrucción con la Clave Tapp dentro del pago, sin OTP.
4. [secuencia-cierre.mmd](p2p-2026-11-26/secuencia-cierre.mmd). Callback, las cuatro pantallas, constancia solo si el cargo no es IBK, evidencia y tope de favoritos.

El éxito de pantalla exige callback de éxito mientras el canal todavía espera. Un timeout no se dibuja como tappeo exitoso.
