# Explicación de Malena — pagos P2P

**Interbank · 26-nov-2026.**
**Fuente:** `parte8.MOV`, 59 minutos. Malena recorre `TPAP - Pagos.pdf`. La transcripción cruda, con las dudas del audio, está en `trabajo/transcripciones/parte8-p2p.md`. La pasada anterior (`parte8.md`) no sirve: el modelo se quedó en un bucle.
**Cómo leer esto:** lo que ella afirma sobre el diseño. Donde el audio deforma un nombre, se usa el de la lámina (`ValidateVPAAddress`, `contactSwipeRequest`, `QRValidation`, CIAM, PAC). Los minutos son de esa transcripción.

El público de la sala incluye a José, por la parte de canal. Ella dice que el foco del entregable es el canal, y que el Proveedor de Cuentas y el PAC van dibujados como referencia.

## Alcance y premisas (00:09–06:40)

José pregunta si TAPP podrá usarlo alguien sin producto Interbank. La respuesta es que por ahora no. Un no cliente es un paso posterior, y todavía no entra en la volumetría.

La lectura es pagos de TAPP a TAPP y de TAPP a teléfono, con distintos modos de ingreso.

Todo TAPP funciona a partir del teléfono de $RM. Estas operaciones usan recursos exclusivos del paquete de integración en la nube. Ya hay un catálogo exclusivo de TAPP. A nivel CICS, por ahora se usa el de Plin. Aunque el pago nazca en el app, cae en el ecosistema TAPP / Plin, igual que hoy Plin se enruta a su propio ecosistema aunque nazca en cualquier canal. La volumetría se piensa creciente, del orden de Plin. El C2C va al clúster nuevo, no al actual.

La arquitectura es transitoria y sin MPA.

La administración de favoritos queda fuera del detalle de este entregable. En la zona TAPP habrá una sección para modificar. Aquí solo se muestra el signo de crear. La regla del favorito se especifica en esa zona.

El entregable no detalla al Proveedor de Cuentas ni al PAC. Pone interfaces de referencia, porque ese diseño ya existe. El foco es el canal, para no cargar el diagrama.

La división es canal, PAC (PSP Izipay) y Proveedor de Cuentas. En el canal no se mete lógica de TAPP. Se dejan las reglas que el canal ya maneja: factores, lecturas, onboarding, no financieras y financieras.

En el PAC viven el directorio local de alias, su administración y la integración con el BCRP. La billetera habla con el PAC. La excepción que ella menciona es la actualización de una réplica.

En el Proveedor de Cuentas viven los PIN de las cuentas, las orquestaciones, las validaciones y consultas, el fraude de otras billeteras, el cargo y el abono, y el diálogo con el resto de aplicativos del banco.

Hay tres capas de límites. El canal solo modela los límites del canal. El PAC, el PSP y el Proveedor de Cuentas tienen los suyos, cada uno en su alcance. On-us y off-us no se redescriben: ya se abordaron en el diseño del proveedor con el PAC.

## Los siete pasos (06:51–15:00)

El pago resuelve primero la llave: TAPP ID, teléfono, QR o contacto del teléfono. El contacto, por detrás, es un teléfono.

Participan el app y el PAC. El directorio participa si la consulta fue por teléfono.

En el monto, el canal valida solo el límite por transacción. Ella corrige la lámina: el acumulado va después. El tope que da es **S/ 500**, configurado en el canal, para que la persona no llegue al final con 600 y lo rechacen el BCRP o el PAC. Si el número cambia, se avisa al canal.

Elegir directorio solo aplica si la búsqueda fue por teléfono. Con TAPP ID o con QR ya se sabe quién es. Si fue teléfono, la persona elige TAPP, Plin, Yape u otro. Elegir TAPP es TAPP a TAPP. Elegir otro es TAPP hacia fuera. Para la billetera es transparente: la bifurcación la resuelve el PAC hacia atrás.

La cuenta cargo sale de lo que el home ya recuperó para el TAPP ID de quien está logueado. El canal muestra esa lista. No es una consulta nueva del pago.

Al autorizar se validan los límites acumulados del canal, y ahí se parte:

- Cuenta Interbank: afiliación a push, token digital, notificaciones nativas, MFA y fraude real time y post mortem. Si eso no está bien, no se deja avanzar.
- Cuenta de otro banco: solo Clave Tapp. Ella lo siente corto frente al reglamento SBS de canales, que pide algo que la persona sabe y algo que tiene. Hoy eso es la clave y el OTP. Deja la pregunta a Alina, Aranza y Rodrigo. En la sala no se cierra.

Del PAC hacia adentro el resultado vuelve resuelto. Intra TAPP pasa por el BCRP y por los dos proveedores de cuenta. Fuera de TAPP, por la cámara.

El Proveedor de Cuentas manda alertas de cargo y de abono, a billeteras Interbank y a otras. El app queda libre de sumar las suyas. Experiencia pidió constancia por correo solo cuando la operación no tocó cuenta Interbank. El texto remite el detalle a la entidad que debitó. Si sí tocó cuenta Interbank, el aviso es el del proveedor de cuentas.

## Cómo se captura la llave (15:15–21:04)

Leer un QR debe sentirse como hoy al leer un QR de Plin o de Yape. Dentro de la trama tiene que venir un campo que diga si el QR lo generó TAPP. Si lo generó TAPP, se abre este flujo. Ella dice que el PSP todavía no lo aterrizó.

Por TAPP ID se escribe el alias y se despliegan los arrobas. Esa lista sale de lo que ya mostraba la consulta de Obed: los directorios. Hay que ver si vive en el teléfono o si se refresca al día.

Por contacto se filtra la agenda. La llave es el teléfono. Escribir el número a mano es la misma búsqueda.

Por favorito no detalla la zona. Por detrás el favorito guarda un teléfono o un TAPP ID, y el pago sigue como si se hubiera empezado por esa llave. Solo cambia cómo se capturó.

Si la llave es TAPP ID, de una se sabe quién es. Si es teléfono, son dos APIs. Una trae a la persona. La otra trae los directorios posibles, y esos se cargan en el selector (Yape, Plin, Bim y los demás). En la misma pantalla se muestran las cuentas cargo del home.

## Cuentas cargo (21:04–24:00)

Experiencia pide agrupar Interbank y otros bancos. Se puede, porque la respuesta de cuentas afiliadas al TAPP ID trae la entidad. Hay pantallas de mezcla, de solo Interbank y de solo otros bancos.

Las que no pueden recibir el cargo van en gris: inactiva, bloqueada, sin saldo, o en modo calle. El modo calle lo sabe y lo resuelve el app. El saldo y el estado de la cuenta Interbank también los sabe el app, porque en el home ya corrió la consulta integrada. El app hace un cruce: consulta integrada contra las cuentas que el PAC devolvió como afiliadas. Si la cuenta Interbank está inhabilitada, inactiva, bloqueada o sin saldo, se grisó. Es lógica del app sobre servicios que ya se llamaron. Pide que Luis, que tiene el detalle de los momentos del app, deje escrito dónde vive cada dato.

## Autorización y una sola API de pago (24:01–27:00)

Cuenta Interbank: MFA (fraude y segundo factor), push, token digital y notificaciones activas. Si falla, no se avanza.

Cuenta de otro banco: se valida la Clave Tapp. El PAC lo resuelve por dentro. A la billetera le importa la respuesta: éxito, se avanza; falla o bloqueo por intentos, no se avanza. En la prueba de TAPP todo eso viaja en la misma API, que es la del pago. No hay una API aparte para enterarse de la validación.

## Constancia (25:35–31:24)

Además de las alertas del proveedor de cuentas, el app pinta una constancia y ofrece guardar favorito.

Si la persona ya llegó al tope de favoritos, el botón no se muestra. El app consulta ese tope antes. Ella había preguntado si guardar favorito exige OTP. Dice que IBEX no lo contestó.

El ecosistema, según lo que les dijo Xipei, tiene un máximo de 45 segundos, y el timeout del app es menor. Estados de pantalla:

- Si el PAC responde «en proceso», se muestra en proceso.
- Si vence el timeout del canal, se muestra «tappeo enviado». No se pinta como exitoso, y lleva un texto corto de espera.
- Si el PAC responde fallido, solo el mensaje de error, sin id de referencia en esa pantalla. En Plin un fallido tampoco queda en movimientos. En TAPP un fallido de verdad implica que se hizo el extorno. Hay otro estado, que en el audio se oye como de prueba o regularización: pudo haber cargo y no abono, y eso se resuelve en conciliación. No se pinta como fallido.
- Si el movimiento fallido se ve en el historial no lo decide esta pantalla. Lo devuelve el PAC. Fernando lo ve en el caso de uso de consulta de historial, en una mesa con Izipay, contrastando la experiencia con lo que las APIs pueden devolver.

El correo de constancia sale de la billetera solo cuando el cargo no tocó cuenta Interbank.

## Financieras, on-us y fraude (31:49–34:50)

El fraude del canal Interbank se gatilla en real time y en post mortem, como hoy con PRE DBFD. BCO cubre los comportamientos que les pidieron. Si el real time desaprueba, igual se gatilla el post mortem. También hay post mortem monetario: falla conocida, se manda fallido; éxito, se manda el éxito.

Toda interacción del app con el mundo TAPP pasa por el PSP / PAC. On-us: el PAC integra directo con el Proveedor de Cuentas. Off-us: pasa por el BCRP, y el BCRP enruta cargo y abono a las entidades que correspondan. El proveedor valida fraudes de otras billeteras, gestiona PIN, límites y transacciones, y habla con el resto del banco. Esas validaciones no las resuelve el canal. Bajan al PAC y al proveedor.

Las tramas de fraude existen porque el equipo de fraude pidió datos del dispositivo. En la solución UPI, esas notificaciones salen desde el proveedor y el PAC, así que hubo que pasar una variante por el canal. En no financieras y en onboarding el esquema es el mismo: real time, y el resto por el PAC.

## Recorrido de la lámina de arquitectura (35:00–43:12)

El home de TAPP no lo recorre. Dice que esos pasos hay que confirmarlos. Sí afirma que ahí se gatilla la consulta integrada de cuentas.

1. Consulta del beneficiario.
   - QR: API del PAC `QRValidation`.
   - TAPP ID: `ValidateVPAAddress`, por el API Gateway de terceros PCI que está en AWS, hacia el PAC.
   - Teléfono: `contactSwipeRequest`. Devuelve en qué directorios está el número.
   - Si la persona elige un directorio, la misma `ValidateVPAAddress` resuelve quién es ese teléfono en Yape, en Plin o en TAPP.
2. Monto: solo límite por transacción.
3. Destino: viene de esa consulta. Elegir destino implica la consulta al directorio elegido.
4. Cuentas: las asociadas al TAPP ID, ya recuperadas.
5. Cargo a cuenta Interbank. Push, y el MFA hacia **CIAM**. Ella le dice a José que ajuste la arquitectura: el llamado no es el componente que tenía dibujado. CIAM resuelve o envía las tramas real time. El 5.2 es real time. El 5.3 es post mortem, también el no feliz. Si el real time se rechaza, se notifica el post mortem a PRE DBFD.
6. Cargo a otro banco: se captura el PIN TAPP con la **Common Library** del SDK, embebida en el app.
7. Instrucción del pago hacia el PAC. La interfaz es síncrona vista desde el canal, y por detrás es asíncrona: cargo, abono, otros PAC, proveedores de cuenta y CCE. Por eso el canal tiene que esperar una respuesta. La propuesta es que el **backend del app exponga una API y el PAC lo llame de vuelta** con el response del pago. Es un callback. No estaba en el diseño original.

Después del callback vienen los pasos 8: post mortem, evidencia digital, alertas por correo y la constancia cuando el cargo es de otro banco.

Favoritos: hay una aplicación que el negocio está sacando a producción para One App, pensada cross canal. Consultar y crear favoritos de TAPP tiene que hacerse sobre esa aplicación.

Ella no ahonda en el on-us. De cara al canal, el recorrido es consulta, monto, destino, validaciones de la cuenta cargo e instrucción.

## Si la persona cierra la app (43:12–46:50)

La duda de la sala: el PAC puede tardar hasta 45 segundos, el app podría cortar a los diez, y la persona cierra. El backend del app sigue escuchando la confirmación. Los datos de la constancia, el correo y la evidencia no pueden vivir solo en la sesión del teléfono. Al recibir la solicitud, el backend sigue el hilo. Alguien de la sala lo confirma: el backend ya tiene el pedido y procesa aunque el cliente se vaya.

Lo nuevo de conectividad, para el pago, es el callback del PAC al backend del app, y el backend hacia la aplicación de favoritos. IAP ya estaba provisionado. El resto el canal ya lo tiene. Los límites digitales pasan por BPI HOST, y se validan antes de disparar la transferencia. El patrón de red del callback queda para Manuel García. En la sala dicen que al día siguiente le mandan la nota. Pedro ya había dicho que la comunicación con IAP existe.

## MFA sin MPA (47:25–49:04)

José dice que en su arquitectura no iba por CIAM porque ese llamado lo hacía MPA. Esta salida no lleva MPA, así que el MFA se llama directo. El multifactor dice si va por biometría o por OTP, y en el medio está la validación de PRE DBFD. En creación de alias y asociación de cuentas, el canal no tiene autorizador: solo registro de evidencia. Malena coincide y pide reconfirmarlo. Biometría se había mencionado y se descartó.

## Plin y el buscador (50:39–59:05)

El flujo de Plin lo ve Fernando. No es limitante de esta semana. Cuando el destino es Plin, el impacto lo absorbe Plin. Para el canal no hay más cambio que incorporar el directorio TAPP, más lo que la experiencia pida y se contraste con Fernando.

Rodrigo confirmó que en el buscador general no habrá búsqueda por TAPP ID. Sí se puede, en visual, armar «Sol Cisneros @» con los PSP disponibles y, al toque, llevar a la zona TAPP. Solo si la persona ya está afiliada. Si no, el camino es el onboarding.

Para saber si un teléfono está en TAPP, Plin u otro directorio, el servicio es `contactSwipeRequest`. Si la lista trae TAPP, se puede redirigir. Si no trae TAPP, se invita a afiliarse. Esa invocación, si se quiere usar desde la pantalla de Plin, no puede esperar a que la persona entre a la zona TAPP.

La lista de PSP (los arrobas) es otro servicio. No se puede consultar por primera vez dentro del home de TAPP: tiene que correr antes, una vez al día, porque también la quiere la pantalla de Plin. El cuidado de en qué momento se consulta queda con Roberto.
