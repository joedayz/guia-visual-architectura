# Diseño de canal — instrucción de pago P2P

**Interbank · 26-nov-2026.**
**Aporte de arquitectura de canal.** Cubre la instrucción del pago desde el APP Personas. El PAC, el switch y el Proveedor de Cuentas entran como cajas con las que el canal habla. Su interior queda en el diseño de referencia del deck.

La base que explica estos dibujos está en [tapp-pagos-p2p-base-2026-11-26.md](../tapp-pagos-p2p-base-2026-11-26.md).

La revisión de fuentes está en [revision.md](revision.md). Lo que falta cerrar, y quién lo cierra, está solo en [pendientes.md](pendientes.md). El PDF está leído en [documento-tpap-pagos.md](documento-tpap-pagos.md). El marco del BCRP está en [seguridad-bcrp.md](seguridad-bcrp.md). Lo que la sala dijo está en [explicacion-malena.md](explicacion-malena.md).

No hay contrato OpenAPI en la fuente. Los nombres que aparecen abajo son los que la lámina 18 escribe (`validateVPAAddress`, `contactSwipeRequest`, `QRValidationRequest`, `PaymentRequest`). No se inventan campos.

## Qué se dibujó

| Entregable | Archivo | Qué responde |
|---|---|---|
| Alto nivel | [componentes-alto-nivel.mmd](componentes-alto-nivel.mmd) | Zonas y quién llama a quién. El PAC y el switch quedan de referencia |
| Bajo nivel | [componentes-bajo-nivel.mmd](componentes-bajo-nivel.mmd) | Qué ejecuta el canal, con el nombre de la lámina y si es síncrono |
| Modelo | [modelo-datos.md](modelo-datos.md) | Conceptos del pago y lo que no se persiste |
| Secuencia | [secuencia-consulta.mmd](secuencia-consulta.mmd) | TAPP ID, teléfono, contacto y QR |
| Secuencia | [secuencia-instruccion-ibk.mmd](secuencia-instruccion-ibk.mmd) | Cuenta Interbank: MFA, real time, límites y PaymentRequest |
| Secuencia | [secuencia-instruccion-externa.mmd](secuencia-instruccion-externa.mmd) | Cuenta de otro banco: Clave Tapp dentro del mismo pago |
| Secuencia | [secuencia-cierre.mmd](secuencia-cierre.mmd) | Callback, pantalla, constancia y favorito |
| Pendientes | [pendientes.md](pendientes.md) | Lo que las historias o un contrato pueden mover |

## Decisiones que el diseño ya toma

- El pago empieza con un cliente que ya entró a la zona TAPP. Las cuentas asociadas a su TAPP ID se recuperaron en el home. Este flujo las muestra.
- El canal resuelve al beneficiario con tres llamadas que Malena nombra. QR usa `QRValidation`. TAPP ID usa `ValidateVPAAddress`, por el API Gateway de terceros PCI en AWS, hacia el PAC. Teléfono y contacto usan `contactSwipeRequest`, que devuelve los directorios donde está ese número. Cuando la persona elige uno, la misma `ValidateVPAAddress` resuelve quién es en ese directorio. Elegir TAPP es un pago TAPP a TAPP. Elegir Plin, Yape u otro es TAPP hacia fuera, y esa bifurcación la resuelve el PAC.
- El monto lo valida el canal contra el límite TAPP por transacción. En la explicación ese tope es S/ 500, configurado en el canal, para cortar antes de que el BCRP o el PAC rechacen el pago. El límite acumulado del canal se evalúa al instruir, junto con la autorización.
- Si la cuenta cargo es Interbank, la autorización usa MFA, push, token digital y notificaciones nativas. El MFA va **directo a CIAM**, porque esta salida no lleva MPA. CIAM dispara la trama real time (5.2). Si el real time rechaza, igual sale el post mortem (5.3) hacia PRE DBFD. El post mortem monetario sale también con el éxito o la falla ya conocidos. Si la cuenta es de otro banco, la app captura la Clave Tapp con la Common Library embebida y el PAC responde éxito, falla o bloqueo por intentos dentro de la misma API de pago. Ese camino queda marcado porque la sala no cerró si hace falta un segundo factor.
- La instrucción sale hacia el PAC con `PaymentRequest`. Hacia el canal la interfaz es síncrona. Por detrás el PAC trabaja asíncrono con el cargo, el abono, los otros PAC y la CCE. Por eso el backend del APP expone un callback y recibe la respuesta. Ese callback no estaba en el diseño original: es conexión nueva, junto con la del backend hacia la aplicación de favoritos. El resto de integraciones del pago ya existe. Si el cliente cierra la app mientras espera, el hilo sigue en el backend y completa constancia, evidencia y post mortem. Esos datos no viven solo en la sesión del teléfono.
- La alerta de cargo y de abono la emite el Proveedor de Cuentas. La constancia por correo del APP se envía cuando la operación no tocó una cuenta Interbank, y el texto remite el detalle a la entidad que debitó.
- Tres capas de límites. Este diseño solo nombra la del canal.
- Favoritos se apoyan en la aplicación cross canal que el negocio ya está sacando a producción para One App. Consultar el tope y crear desde la constancia van contra esa aplicación. Si la persona ya llegó al tope, el botón no se muestra. La regla de la zona TAPP sigue fuera del detalle de este entregable. Quedó sin respuesta si guardar favorito pide OTP.
- MPA no participa.
- La Common Library viaja con la app. La firma del mensaje y la validación de la respuesta del PAC son parte de la instrucción, no un proyecto aparte. El device binding se da por hecho desde el onboarding.

## Qué ve cada paso

1. **Consulta.** APP y PAC. YP si la llave es teléfono. Resultado: beneficiario identificado, y directorios solo en ese caso.
2. **Monto.** Solo el APP. Resultado: monto dentro del límite de canal por transacción.
3. **Directorio.** Solo si el paso 1 fue por teléfono. APP, PAC y directorio.
4. **Cuenta cargo.** Solo el APP, con la lista del home. La respuesta de cuentas afiliadas trae la entidad, y la pantalla agrupa Interbank y otros bancos. Una cuenta que no puede recibir el cargo se muestra en gris: inactiva, bloqueada, sin saldo, o en modo calle. El modo calle lo resuelve el propio app.
5. **Instrucción.** Autorización según el tipo de cuenta, límite acumulado de canal, y envío al PAC.
6. **Procesamiento.** Caja negra desde el canal: BCRP y proveedores de cuenta si es intra TAPP; cámara si es fuera de TAPP; on-us en el camino que el PAC y el Proveedor de Cuentas ya tienen.
7. **Aviso y pantalla final.** Alerta del proveedor de cuentas para el cargo y el abono. Constancia de correo del APP solo sin cuenta Interbank. En pantalla: éxito si el callback llega a tiempo y el pago salió; «en proceso» si el PAC responde ese estado; «tappeo enviado» si vence el timeout del canal, sin pintarlo como éxito. El tope de espera del ecosistema que nombra la sala es 45 segundos. El timeout del canal es menor y queda por fijar. Si el PAC devuelve fallido, la pantalla de la operación muestra solo el mensaje de error. Si el cliente cerró la app, el backend igual completa los pasos posteriores.

## Seguridad aplicada a estas cajas

- El canal es TPAP. Genera la firma XML con la Common Library y valida la firma que vuelve del PAC antes de pintar éxito.
- No guarda Clave Tapp, OTP ni token. La pantalla los captura y los entrega al componente que ya los valida (PAC o el factor del canal). No hay tabla nueva para eso.
- TLS 1.2 o 1.3, con los tres cipher suites del Anexo 4, y pinning en la app. El reloj del dispositivo y el de los servicios que firman quedan dentro de un segundo del NTP del PSO.
- Alias, teléfono, identificador de dispositivo y datos de la operación se tratan como dato que el marco manda cifrar. El canal no los escribe en claro en un log de constancia.
- El PAN que PRE MONITOR pide a TRJ no cruza hacia el APP. El canal no lo modela.
- Un incidente que corte la instrucción o el firmado se reporta al PSO en una hora, con el informe de mitigación en veinticuatro. El canal aporta el `msgId` y el `ts` que el anexo pide en el log.

## Lo que la misma sesión dejó junto al pago

En el buscador general no se busca por TAPP ID. Se puede mostrar el alias con arroba y la lista de PSP, y al toque entrar a la zona, solo si la persona ya está afiliada. Esa lista se consulta una vez al día y antes de entrar a TAPP, porque la pantalla de Plin también la usa. Saber si un teléfono está en el directorio TAPP es `contactSwipeRequest`. Para un destino Plin, el canal incorpora ese directorio y Plin absorbe el resto del cambio.

## Qué no se cierra en este diseño

Los diez puntos, y el hueco de páginas que no están en las fotos, están en [pendientes.md](pendientes.md). Cuando lleguen las historias, se ajusta la secuencia del paso que la historia contradiga.
