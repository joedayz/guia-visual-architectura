# TPAP — Pagos. Lectura del PDF

**Interbank · 26-nov-2026.**
**Fuente:** `TPAP - Pagos.pdf`, 21 páginas, fotografiado en `p2p/` (`IMG_0218` a `IMG_0242`). El visor lo abre desde `C:/Users/x16938/Downloads/TPAP%20-%20Pagos.pdf`. Marca de agua 1N. Uso interno.
**Qué entra aquí:** el texto y los diagramas de las láminas. Lo que Malena dice encima de cada lámina va en [explicacion-malena.md](explicacion-malena.md).
**Hueco:** la página 21 no está en las fotos. La 20 es el cierre «Gracias». Varias láminas de experiencia están fotografiadas más de una vez; se leen como una sola página.

El objetivo de la lámina 2 es presentar la arquitectura detallada para instruir pagos inmediatos por el sistema UPI del BCRP desde el APP Personas.

---

## Lámina 3. Qué es UPI

Ecosistema que permite iniciar pagos directamente desde las cuentas de los clientes, integrándose mediante procesadores con cualquier billetera móvil.

Impactos:

- menores costos de procesamiento
- nuevo alias como método de pago: **TAPP ID** (hoy teléfono)
- nuevas reglas de juego y roles: Open Payments
- nuevos perfiles definidos: **P2P** y **P2M**
- nuevas oportunidades de negocio como procesador

Roles:

1. Third Party Application Provider (TPAP)
2. Payment Service Provider (PSP)
3. Proveedor de Cuentas
4. UPI Like Switch

La leyenda de la lámina 7 asigna color: TPAP a Interbank, PSP a Izipay, UPI Like Switch al BCRP, Proveedor de Cuentas a Interbank.

## Láminas 4 y 5. Necesidad

Título de ambas: realizar y recibir pagos automáticos a través de TAPP.

### Lámina 4. Pago entre dos billeteras, por UPI

Participantes, de izquierda a derecha: ordenante, billetera móvil / TPAP, PSP origen, UPI (DNM Switch, Translator, Logical Switch), PSP destino, billetera móvil / TPAP, beneficiario.

Debajo del switch: Proveedor de Cuenta del banco origen (aprueba el cargo a la cuenta del ordenante) y Proveedor de Cuenta del banco destino (aprueba el abono a la cuenta del beneficiario).

Arriba a la derecha, DNM YP con logos Plin, Yape y Bim. Abajo, agregador CCE.

Ejemplo de la lámina. Ordenante `juan@ibk` / `946547136`, con cuenta IBK, cuenta BBVA y cuenta Falabella. Beneficiaria `maria@scotia`, con cuenta BCP, cuenta IBK y cuenta Falabella.

Hay catorce pasos numerados entre el ordenante y el beneficiario. Los pasos **4, 8 y 14** aplican si el usuario ingresó teléfono. La foto no rotula el verbo de cada número; el dibujo muestra ida y vuelta entre billetera, PSP, switch, proveedores de cuenta, DNM YP y CCE.

### Lámina 5. Pago que entra por una entidad externa y el agregador CCE

El ordenante está en una entidad externa (logo de banco rojo). Esa entidad habla con el agregador CCE. El CCE consulta y enruta el pago hacia UPI. UPI sigue al PSP destino y a la billetera del beneficiario. El banco destino aprueba el abono.

Ejemplo. Ordenante `91234568` PLIN Scotia, con cuenta Scotia, cuenta BBVA y cuenta IBK. Beneficiario `946547165`, con destinos PLIN, DALE, YAPE y TAPP. DNM YP vuelve a aparecer arriba, con Plin, Yape, Bim y TAPP.

Pasos visibles: 1 y 2 del ordenante a la entidad y al CCE; 3 consulta y 4 enrutamiento en el CCE hacia UPI; 5 hacia el PSP destino; 6 hacia el banco destino.

## Lámina 6. Premisas

- La asociación de cuentas a TAPP será a partir del teléfono de **$RM**.
- Estas operaciones se suman al ecosistema **TAPP (CP4I)** y **PLIN (CICS)**.
- El diseño del Proveedor de Cuentas toma los requisitos del BCRP para los canales y los requerimientos de UX de Interbank. La foto corta el final de «canales».
- Como excepción, la arquitectura es **transitoria y sin MPA**, para no impactar la salida del proyecto.
- La administración de favoritos está **fuera de alcance** de este entregable.
- Este entregable **no detalla** la arquitectura del Proveedor de Cuentas ni la del PAC.

## Lámina 7. Mapa de capacidades por rol

Seis columnas: Canal, PSP, UPI, Proveedor de Cuenta, Motor de Pago, Core.

### Canal

- Validación del primer factor de autenticación (login)
- Registro de la evidencia digital
- Validación del segundo factor de autenticación
- Lectura de QR tokenizado, CCE y adquirente
- Notificaciones regulatorias al cliente
- Flujos de consulta de alias o teléfono del beneficiario
- Flujos de operaciones financieras
- Registro y seguimiento de reclamos

En la misma columna, como bloques del canal y no como cajas sueltas del PSP: flujos de onboarding, flujos de operaciones no financieras, gestión de límites del canal.

### PSP

- Directorio local de alias
- Administración local de alias
- Integración con la billetera TAPP y con UPI para onboarding
- Integración con la billetera TAPP y con UPI para operaciones no financieras
- Integración con la billetera TAPP y con UPI para consultar alias o teléfono del beneficiario
- Integración con la billetera TAPP y con UPI para operaciones financieras
- Integración con el Proveedor de Cuentas para la instrucción de la transferencia on-us
- Gestión de límites
- Integración con UPI para el registro de reclamos

### UPI

- Directorio centralizado de alias
- Administración centralizada de alias
- Integración con el Proveedor de Cuentas para solicitar validaciones y consultas
- Integración con el Proveedor de Cuentas para la consulta de saldo
- Consulta de directorio de otro PSP, de CCE o de YP
- End to end de la orden de transferencia
- Gestión de límites
- Conciliación y liquidación
- Atención de discrepancias

### Proveedor de Cuenta

- Administración de los PIN asociados a cuentas
- Orquestación de la consulta de cuentas asociadas a un móvil
- Orquestación de la validación de datos de la TD asociada a una cuenta, generación y envío de la OTP
- Orquestación de la validación de OTP, clave ATM de una TD asociada a una cuenta, y creación de la clave PIN
- Validación de la clave PIN de la cuenta
- Orquestación de la consulta de saldo de una cuenta, previa validación del PIN
- Validación de fraude
- Instrucción del cargo, del abono y del extorno
- Registro de información para la constancia de pago
- Notificación al cliente, configurable
- Instrucción de la transferencia on-us
- Gestión de límites
- Conciliación y liquidación
- Atención de discrepancias

### Motor de Pago

- Instrucción del cargo, del abono y del extorno
- End to end de la transferencia on-us
- Gestión de límites
- Conciliación y liquidación
- Atención de discrepancias
- Liquidación

### Core

- Consulta de cuentas asociadas a un móvil
- Validación de datos de la TD asociada a una cuenta
- Validación de la clave ATM de una TD asociada a una cuenta
- Consulta de saldo de una cuenta
- Ejecución del cargo, del abono y del extorno

## Lámina 8. Proceso de pago TAPP a TAPP / teléfono

Flujo de punta a punta, desde la consulta del beneficiario hasta la notificación.

Principio: validar identidad, límites, seguridad y destino antes de procesar el pago.

Pregunta escrita en rojo en la lámina, sin respuesta: el reglamento de ciberseguridad de la SBS para canales digitales, ¿se cumple validando solo el PIN TAPP cuando la cuenta es de otro banco?

Controles transversales: seguridad, fraude, límites, trazabilidad, conciliación, disponibilidad. Salida: pago ejecutado y partes notificadas.

| Paso | Qué hace | Aplicaciones que la lámina nombra | Resultado |
|---|---|---|---|
| 1. Consulta de beneficiario | Por TAPP ID, contacto, teléfono o QR. Se consulta al beneficiario | APP, PAC, YP | Beneficiario identificado |
| 2. Ingreso de monto | Límite por transacción y límite acumulado. Se validan límites de canal por transacción | APP | Monto validado |
| 3. Selección del directorio destino | Elegir directorio o ruta. Solo si la consulta fue por teléfono | APP, PAC, YP | Destino seleccionado |
| 4. Selección de cuenta cargo | Cuentas IBK o no IBK | APP | Cuenta y seguridad validadas |
| 5. Instrucción del pago | Confirmar datos y autorizar. Límites acumulados de canal. Si la cuenta cargo es IBK: afiliación a push, token digital, notificaciones nativas activas, MFA y fraudes. Si no es IBK: solo PIN TAPP | APP, PAC, BPI, GNT, Autho, PAC, BCRP, otro PC | Pago instruido |
| 6. Procesamiento | Ejecutar, contabilizar y registrar trazabilidad | BCRP, PC origen, PC destino, CCE | Pago procesado |
| 7. Notificaciones | Informar al ordenante y al beneficiario. El PC envía alertas de cargos y abonos. El APP IBK envía constancia si la transacción no involucró cuentas IBK | APP, SMTP, IAP, GNA, GNT | Partes notificadas |

La lámina 8 pone el límite acumulado en el paso 2. En la reunión Malena corrige eso: en el paso 2 el canal valida solo el límite por transacción, y el acumulado entra en la instrucción. El detalle verbal está en la explicación.

## Láminas 9 a 11. Experiencia de consulta, monto y cuenta

Pantallas «Paga con Tapp». Lo que se lee en los mocks, no como regla cerrada de negocio:

**Consulta.** Búsqueda por TAPP ID, por contacto del teléfono, por teléfono, por favorito y por lectura de QR. Hay un estado en el que el beneficiario existe solo en otros directorios, y otro en el que existe en el directorio TAPP. Hay error al cargar la cuenta principal, con acción de actualizar. Se muestra favoritos, con gestionar. El teclado y «Mostrar mi código / subir código QR» aparecen en la misma familia de pantallas.

**Monto.** Campo de monto, leyenda «Hasta S/ 500 por operación» en el mock, cuenta de cargo con consultar saldo, mensaje opcional, y «Selecciona el destino» cuando hay más de un directorio (Tapp, Plin, Yape en el mock). Un monto de S/ 501 se muestra como caso de la pantalla de límite.

**Cuenta cargo.** Dos grupos: Interbank y Otros bancos. En otros bancos el texto dice que se deberá ingresar la Clave Tapp. Hay cuenta principal, cuenta inactiva y cuenta bloqueada. Consultar saldo. «¿Olvidaste tu Clave Tapp? Puedes editarla en Configuración.» El botón de la selección es Guardar.

El S/ 500 y los nombres de las pantallas (Sol Cisneros, cuentas enmascaradas) son del mock. No hay una lámina que fije S/ 500 como límite normativo.

## Lámina 12. Validaciones de cargo a cuenta IBK

Lista de la lámina:

- MFA (fraudes y 2FA)
- afiliación a push
- afiliación a token digital
- notificaciones nativas activas

El mock de al lado muestra la cuenta IBK elegida y el paso de confirmar monto, mensaje y cuenta de cargo.

## Lámina 13. Validaciones de cargo a otras cuentas

El mock pide **Ingresa tu Clave Tapp** y muestra «Powered by UPI». Una nota manuscrita en la foto, sobre la constancia, dice «Falta incluir OTP como 2FA?» y «PIN TAPP». Es una nota de la reunión, no un texto de la lámina.

## Láminas 14 y 15. Constancia

**En el app.** Estados «Tappeo exitoso» y «Tappeo en proceso». Datos que se leen: monto, nombre del beneficiario, fecha y hora, «Enviado a» con TAPP ID, guardar favorito, TAPP ID de cargo, entidad de cargo, cuenta de cargo enmascarada, código de referencia, compartir, volver a Tapp. Hay un toast de favorito guardado.

**Por correo, cuenta de otro banco.** Asunto de pago enviado. El cuerpo dice que el pago salió desde una cuenta cargo fuera de Interbank y que la constancia de la entidad que debitó llegará aparte. Campos: código de referencia, fecha y hora, destinatario, destino Tapp, TAPP ID de cargo, entidad de cargo, cuenta de cargo, monto. Una etiqueta del mock dice «(LEGACY) fase-1/15» y «Sólo para operaciones con cargo en otras cuentas».

## Lámina 16. Aplicaciones impactadas

Leyenda: marrón = con cambio, verde = sin cambio, azul = nuevo Interbank, y un cuarto estado para externo. En la foto el azul de «nuevo» casi no se distingue del marrón; MPA es el único recuadro marcado **To Be**.

| Zona | Aplicaciones |
|---|---|
| Canales | APP Personas (con cambio), WhatsApp (con cambio) |
| Producto | PLIN; Reclamos: GIRU; Notificación: GNT (con cambio) y SMTP (sin cambio); Fraude: PRE DBFD y ACF |
| Data y analytics | LAKE, GDC, SDL, RNI |
| Cross | CIAM, LÍMITES, FAVORITOS |
| Contabilidad | EBS |
| Cumplimiento | PLD |
| Procesadores | PSP @ibk, Proveedor de Cuentas, y **MPA marcado To Be** |
| Core | TOLD, DEP (sin cambio), $$T, $IM, $RM, CDI, CDS, CARDS, TRJ |
| UPI Like | DNM Switch, UPI Like Translator, UPI Like Logical Switch, BCRP Directory, URCS, BCRP |
| Otros | CCE, YP, TPAP externa, PSP externo, PC externo, comercio |

## Lámina 17, hoja de onboarding y no financieras

Misma estructura para una billetera IBK y para Yape, como contraste.

Si hay datos del dispositivo, el camino es APP IBK → PSP IZI IBK → UPI Switch BCRP. Si no hay datos del dispositivo, el camino dibujado es APP YAPE → PSP CULQI YAPE → el mismo switch. El switch dice **«NO manejamos datos de tarjeta»**.

APP IBK también habla con BCO por el SDK BCO, y con PRE DBFD por tramas RT y tramas PM de la billetera IBK.

Debajo del switch, el PC IBK hace validaciones y consultas. Tiene generador de OTP y PIN TAPP. Hacia el mainframe: PRE MONITOR y ACF. PRE MONITOR llama `ObtenerTarjeta(Cu): PAN` por un enlace directo a la rutina de TRJ, hacia CDD. Una caja punteada agrupa TRJ, CDI, CARDS, $RM, $$T y $IM. GNT sale de las notificaciones: billeteras IBK por push de la app, otras billeteras por SMS.

El PC Yape está al otro lado del switch, sin el detalle interno de Interbank.

## Lámina 17, hoja de financieras

Misma partición IBK / Yape y la misma leyenda de que no se manejan datos de tarjeta.

APP IBK → PSP IZI IBK. Desde el PSP hay un camino **ON US** directo al PC IBK, y el camino de cargos y abonos que baja del UPI Switch al PC IBK. Junto al PC IBK hay un almacén de PIN, límites y transacciones. Junto al PSP, un almacén de transacciones TAPP.

PRE DBFD recibe, solo si el cargo va a cuenta IBK: trama RT, PM no felices y PM monetario.

El PC IBK habla con el mainframe (PRE MONITOR, ACF, `ObtenerTarjeta(Cu): PAN` hacia TRJ) por tramas RT y tramas PM de otras billeteras, felices y no felices, monetarias. Hacia el core punteado (TOLD, DEP, $$T, $IM) van cargos, abonos, extornos, consultas y on-us. GNT notifica igual: push para billeteras IBK, SMS para las otras.

## Lámina 18. Arquitectura de solución

Zonas que se leen en la foto:

- Cloud Azure IBK
- Cloud IBK AWS. La foto se lee «Cloud IEIK AWS»; se toma como la nube AWS de IBK
- Cloud externo Izipay
- IBK on premise: cloud privada, distribuido y mainframe

El pie dice que la arquitectura del Proveedor de Cuentas Interbank y del PAC @IBK es referencial, y que las transferencias on-us son síncronas.

Recorrido numerado, como está rotulado. La foto está tomada de lado; un sufijo de servicio puede leerse mal.

1. En el home de TAPP se gatilla la consulta integrada de cuentas (BCO).
2. Ingreso de monto.
3. Consulta de beneficiario y selección de destino. UPI CL está junto al cliente en este tramo.
   - 1.a consulta por QR
   - 1.b consulta por TAPP ID
   - 1.c consulta por teléfono
   - 3.1 consulta por teléfono más directorio
4. Muestra de cuentas. Sale de lo ya recuperado.
5. Validaciones de cargo a cuenta IBK.
   - 5.1 valida afiliación a push, token y notificaciones, y MFA
   - 5.2 tramas RT hacia PRE DBFD, por APIM interno
   - 5.2 valida límites digitales, hacia el mainframe (BPI HOST / LÍMITES), por APIC + ACE
   - 5.3 tramas PM no monetarias, por CIAM y APIM interno
6. Validaciones de cargo a otros bancos: PIN TAPP.
7. Instrucción de pago.
   - 7. PaymentRequest, síncrono, hacia el service bus de AWS
   - 7.2 response
   - Servicios nombrados en el tramo de consulta, síncronos, vía API Gateway: `validateVPAAddress`, `contactSwipeRequest`, `QRValidationRequest`
8. Procesamiento y avisos.
   - tramas PM monetarias
   - cargo, abono y extorno hacia TOLD, DEP, $$T y $IM, por APIC + ACE
   - envío de alerta por correo (GNA / GNT)
   - envío de constancia para cargos en otros bancos, por SMTP, saliendo de APIC + ACE
   - evidencia digital hacia IAP
9. Consultar favoritos, por APIM interno hacia IAP.
10. Crear favorito.

En la zona AWS: Service Bus, LTD, y las llamadas síncronas del API Gateway hacia PAC @IBK.

En la zona Izipay: PAC @IBK, BCRP, CCE, YP, PAC externo y proveedor de cuenta externo. `ConsultaBeneficiario/Cargo/Abono` hacia el PAC va **asíncrono**. El cargo y el abono siguen a BCRP, al proveedor externo y a CCE. YP resuelve directorio y beneficiario. Hay una etiqueta de transferencias on-us síncronas junto al PAC @IBK.

La lámina 6 dice que favoritos está fuera de alcance. La lámina 18 igual dibuja consultar y crear favorito. Queda como conflicto de la propia fuente, no como decisión.

## Lámina 19. Consideraciones

Un solo bullet:

Usar HMAC (Hash-based Message Authentication Code) para verificar la integridad de los datos.

El resto de la lámina está vacío.

## Lámina 20

Cierre: Gracias.

## Lámina 21

No está en las fotos.
