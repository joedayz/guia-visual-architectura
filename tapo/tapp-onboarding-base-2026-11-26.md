# TAPP — Base de diseño: onboarding y post-login

**Interbank · 26-nov-2026**
**Aporte:** arquitectura de canal (José). Alcance de redibujo: alto nivel, bajo nivel, modelo de datos y secuencias de onboarding y post-login.
**Estado de los dibujos:** los `.mmd` del 23-nov siguen en `docs/architecture/` como versión anterior. Este archivo es la fuente para rehacerlos. No inventa contratos ni cajas que las fuentes no respalden.

Guía de los entregables: [Guia-para-elaborar-diagramas-tecnicos.md](../../Guia-para-elaborar-diagramas-tecnicos.md). Índice de la versión anterior: [tapp-onboarding-diagramas.md](tapp-onboarding-diagramas.md).

---

## 1. Cómo usar este archivo

1. Los nombres de servicio del **deck** mandan sobre la transcripción. El audio de la sala deformó varios nombres.
2. Las **historias** mandan sobre el comportamiento que ve el cliente y sobre las reglas de negocio. El deck manda sobre cajas, zonas y rutas `/service/...`.
3. Donde deck e historias usan catálogos distintos, el diseño los muestra como mapeo pendiente. No se unifican por parecido.
4. Lo que no está cerrado se dibuja punteado o se deja fuera del diagrama, con la marca de este archivo.

---

## 2. Alcance

Entran tres procesos, en este orden:

1. Validaciones iniciales al entrar a la zona TAPP (post-login de TAPP, no el login del banco).
2. Creación del Tapp ID.
3. Afiliación de la primera cuenta y Clave Tapp, incluida la consulta y el cambio de principal en el directorio cuando el deck lo dibuja.

Queda fuera:

- Pagos, cargos y abonos. El deck los trae como contexto (láminas No financieras y Financieras) y no son secuencia de onboarding.
- Administración de favoritos. Premisa del deck y de la DHU.
- Alta de cuentas bancarias, emisión o reemplazo de tarjeta, y afiliación de otros productos del banco.
- Vincular cuentas adicionales, modificar afiliaciones y administrarlas después. Eso es otra DHU: Vinculación de Cuentas.
- Arquitectura interna del proveedor de cuentas y del PAC. El propio deck dice que este entregable no la detalla.
- MPA y WhatsApp. Se discutieron sobre una lámina de aplicaciones impactadas que no está en el PDF de 16 páginas.

Canal: App Interbank, Android e iOS. Público de la DHU: clientes Interbank que crean o activan su Tapp ID y vinculan al menos una cuenta elegible.

---

## 3. Fuentes

| Fuente | Qué fija |
|---|---|
| Reunión Teams del 25-nov, `IMG_0130.MOV`. Transcripción en `trabajo/transcripciones/cierre-2026-11-25.md` | Decisiones de sala: dispositivo seguro, caché, timeouts, modo calle, portabilidad, IAP |
| `TAPP-Onboarding.pdf`, 16 páginas. Fotos en `imagenes/IMG_0133.HEIC` a `IMG_0149.HEIC` | Zonas, pasos numerados y rutas `/service/...` |
| `ETCDYP-DHU - Onboarding.pdf`, 26 páginas. Fotos en `imagenes/hus/` | Historias, criterios, reglas de negocio y glosario |
| Diagramas del 23-nov en `docs/architecture/` y `TAPP-onboarding-arquitectura.md` | Versión anterior. Varias decisiones quedaron superadas; la lista está en la sección 7 |

Participantes de la reunión: presenta Obed Zevallos. Luis Alfaro, Diego Viera, Bryan Cipriano y José.

Stakeholders de la DHU: Gabriela Canaval (LPO, Producto), Mayra Chavez (LPO, App), Andrea Escudero (PO, App), Daniella Tapia (LPD, Experiencia), Alibeth Gonzales (CX, Experiencia), Sol Cisneros (PD, Experiencia).

Otras DHU citadas desde las historias, y que no están en este paquete:

- Acceso a Zona Tapp: validaciones mínimas al entrar.
- Configurar TAPP, transacciones no financieras: dispositivo, celular, auditoría y evidencia.
- Vinculación de Cuentas: cuentas adicionales y administración posterior.

---

## 4. Premisas del deck

Texto de la lámina 5, más lo que la sala cerró encima:

- La cuenta se asocia a TAPP a partir del teléfono de **SRM**.
- El timeout del PAC (Izipay) es **3 segundos** en transacciones no financieras. La volumetría y las pruebas de performance de Izipay siguen pendientes. La premisa de diseño es 3 segundos por llamada.
- Los datos hacia el PAC van cifrados con el algoritmo que la lámina nombra **curva elíptica AES256**.
- El flujo contempla el **ecosistema de seguridad del canal**. En sala, Luis pidió dejarlo como consideración y no como caja nueva: dispositivo seguro, modo calle y verificación de token digital. El microservicio de dispositivo seguro ya está en el login. Solo ese dispositivo entra al onboarding y mueve dinero. Cambiarlo pasa por la biometría que el canal ya tiene.
- Fecha de vencimiento y clave de cajero **no se guardan**. La pantalla pide los **últimos 6 dígitos**. El mock de «Ingresa tu Clave Tapp» pinta cuatro posiciones. En sala se dijo que el largo (4 o 6) sigue por validar y no cambia el dibujo.
- Favoritos queda fuera de este entregable.
- Este entregable no detalla la arquitectura del proveedor de cuentas ni del PAC.

El post-login corre **al tocar la zona TAPP**, en cada ingreso, porque el app no es una billetera exclusiva de TAPP.

El API Gateway de terceros PCI es la capa de integración, no una aplicación nueva. Malena pidió separar las cajas. El PAC del post-login queda rotulado **PAC @interbank**, en la nube externa. En creación y afiliación la zona se llama Izipay (PAC), Cloud AWS Izipay.

---

## 5. Catálogo de servicios del deck

Toda llamada al PAC es síncrona y pasa por el API Gateway de terceros PCI, salvo las que el diagrama marca asíncronas (post-mortem, evidencia, comportamental).

### 5.1 Post-login, cliente sin Tapp ID

Solo tres pasos. No llama listas ni directorio.

| Paso | Servicio |
|---|---|
| 1.2 Validar variación de dispositivo | `QueryDeviceDetails` (sync) |
| 2.1 Llaves BCRP | `ListKeys` (`type=ListKeys`, sync) |
| 3.b Obtener token CL | `ListKeys` (`type=GetToken`, sync) |

El SDK del app expone `getCLVersion`, `getChallenge` y `registerApp`. El reto sale del dispositivo. El backend integra el SDK. Si el cliente nunca hizo onboarding, no hay baseline de dispositivo y la variación no se consulta.

### 5.2 Post-login, cliente con Tapp ID

Los mismos tres, más:

| Paso | Servicio |
|---|---|
| 4.b Lista de PSP | `ListPsp` (sync) |
| 5.b Lista de proveedores | `ListAccPvd` (sync) |
| 6.1 Datos del cliente | `QueryVPAs` (sync) |
| 7.1 Tapp ID en el directorio | `getMapperAddressList` (sync) |

Los pasos marcados «A» se resuelven en el backend si el dato sigue vigente. Se sale al PAC solo para crear, rotar o refrescar. El token CL es por cliente. Las listas de PSP y de proveedores son globales: el PAC las refresca una vez al día e Interbank las guarda en **Redis con TTL de una hora**, para no quedar un día atrás si el proceso del PAC falla.

### 5.3 Creación del Tapp ID

| Paso | Dónde | Servicio o efecto |
|---|---|---|
| 1 | App | Obtener número. Android lee el SIM. Si no puede, el cliente lo escribe. iOS siempre lo escribe |
| 2.1 | Backend | Solicita token |
| 2.2 | Pre-DBFD | Registro real-time |
| 2.3 | SRM | Obtiene el celular y lo contrasta |
| 2.4 | PAC | `/service/QueryDeviceDetailsReq`. La flecha sigue al **UPI Switch** |
| 3 | App | Envía el SMS. Android desde el app. iOS desde Mensajes |
| 4.1 | Backend | Consulta la validación del token y la creación |
| 4.2 | PAC | `/service/ValidateDeviceDetailsReq`. Valida el SMS y crea el Tapp ID. La flecha sigue al **UPI Switch** |
| 4.3 | Pre-DBFD | Post-mortem, asíncrono |
| 4.4 | LTD y BCO | Evidencia digital y registro comportamental, asíncronos, por el API Gateway |

La versión del 23-nov decía que la creación no toca el switch. El deck del 25-nov dibuja las dos llamadas del PAC hacia el UPI Switch. Publicar el alias en el directorio sigue estando en la afiliación (pasos 6.a y 6.b), no en estos dos servicios.

Si el cliente abandona en «Antes de comenzar», no hay Tapp ID, no hay escritura en el directorio y no hay afiliación parcial. Si el Tapp ID ya se creó y el cliente regresa, queda **inactivo** hasta afiliar una cuenta.

### 5.4 Afiliación de cuenta

| Paso | Servicio en el deck |
|---|---|
| 1.2 Lista de proveedores | `/service/ReqListAccPvd` |
| 2.1 | Registro real-time en Pre-DBFD, antes de listar cuentas |
| 2.2 Listar cuentas | `/service/ListAccountReq` |
| 2.2 | Si las cuentas on-us difieren de las del PAC, aviso a **IAP**. La flecha no tiene nombre de servicio |
| 3.2 Afiliar la cuenta | `/service/AddAccountReq` |
| 4.1 OTP por SMS o correo | `/service/UpiOtpRequest` |
| 5.a Crear Clave Tapp | `/service/UpiPinReq` |
| 5.b Validar Clave Tapp | `/service/BalEnqReq` |
| 5.a y 5.b | Evidencia digital asíncrona a LTD |
| 6.1 ¿Está en el directorio? | `getMapperAddressListReq` (`type=CHECK`) |
| 6.a Vincular el celular | `createMapperReq` |
| 6.b Consultar portabilidad | `getMapperAddressListReq` (`actionType=PORT`) |
| 6.b Realizar portabilidad | `createMapperReq` (`actionType=MODIFY`) |
| 6.a y 6.b | Post-mortem asíncrono a Pre-DBFD |

`BalEnqReq` es el rótulo de «validar clave» en la foto del deck. El nombre no parece el de una validación de PIN. Se conserva el rótulo y se marca para confirmar en el PDF original.

El common library toma los últimos dígitos, la fecha de vencimiento y la clave de cajero, los cifra y el backend se los entrega a Izipay. El canal no decide si el dígito, el vencimiento o la clave están mal: lo responde el PAC. El switch no maneja datos de tarjeta. Esa frase está en las láminas de contexto de pagos.

---

## 6. Reglas de la DHU que el diseño debe mostrar

Índice de historias. Hay dos HU 03 y no existen en este PDF la 02 ni la 05 a la 07.

| HU | Comportamiento que entra al diseño |
|---|---|
| 01 | Acceso en Mis operaciones y en Más opciones («Gestionar mi Tapp ID»). Modal de invitación al cumplir elegibilidad. Cerrar el modal no inicia el onboarding |
| 03 | Al entrar a TAPP se ejecutan las validaciones mínimas. El detalle está en la DHU Acceso a Zona Tapp |
| 03 | «Afiliarme» acepta términos y política de privacidad. No hay checkbox. Cerrar la bienvenida corta el flujo |
| 04 | «Antes de comenzar» lista tarjeta (últimos 6 y vencimiento) y avisa la clave de cajero. El enlace de cambio de clave sale al flujo del banco y regresa. Abandonar ahí no persiste nada |
| 08 | Solo entidades habilitadas, vía `ReqListAccPvd` / `RespListAccPvd`. La entidad elegida determina el banco de `ListAccount`. No se llama `ListAccount` si la respuesta no trae al menos una entidad válida |
| 09 | `mbeba=Y`: la cuenta ya afiliada no crea un PIN nuevo; se pide el UPI PIN existente. `mbeba=N`: va a creación de PIN. `mbeba` no sustituye la validación de afiliación. Modo calle e Interbank: solo la cuenta que `RespListAccount` marca como modo calle. Cuenta no elegible: se detiene y se informa |
| 10 | Restricción de seguridad de la tarjeta: «Acceso limitado al App» y alternativa de tarjeta de débito digital |
| 11 | `ReqOtp` genera el OTP. La validación efectiva es `ReqRegMob`. El app no lo persiste ni lo reutiliza. No muestra el envío como confirmado si solo salió la solicitud |
| 12 | `ReqRegMob` lleva la validación de OTP y de clave de cajero, y la configuración del UPI PIN cuando corresponde. CBS valida cuenta activa. DCMS valida tarjeta. ATM valida el PIN. Validar la cuenta en CBS no es afiliarla al Tapp ID. El tercer intento bloquea el onboarding 24 horas. Está pendiente de validación e incluye bloqueo de la tarjeta. Las credenciales no quedan en el modelo funcional |
| 13 | Crear o validar la Clave Tapp / UPI PIN |
| 14 | Éxito en pantalla solo con `SUCCESS` de la relación cuenta a Tapp ID de la cuenta elegida. Si no se confirma, pendiente o error, nunca éxito definitivo. Se genera constancia y se envía a la bandeja del cliente |
| 15 | El código técnico se mapea a mensaje, acción y recuperación. Un estado desconocido no se resuelve repitiendo la operación. Las credenciales no viajan en el estado de recuperación. Alias, binding, afiliación, cuenta principal y favorito son dimensiones independientes |

Cierre del onboarding, regla de la HU 14:

- Hay un Tapp ID válido. Si ya existía, no se crea otro.
- Hay device binding vigente cuando aplique.
- Hay al menos una cuenta afiliada.
- Las validaciones del escenario salieron bien.
- Tener UPI PIN no alcanza para dar el flujo por cerrado.

### Glosario de la DHU, y cómo convive con el deck

| Término en la DHU | Significado en la DHU |
|---|---|
| Alias | Identificador registrado en el ecosistema TAPP |
| TAPP ID | Nombre con el que se le presenta ese alias al cliente |
| Favorito | Alias que el cliente prefiere. El cierre de esta DHU no lo crea ni lo cambia |
| Cuenta principal | Cuenta que recibe los fondos. No se infiere del favorito ni se cambia como efecto de la afiliación |

En la reunión y en el deck, «Tapp ID principal» es el alias que el directorio devuelve al buscar por celular, y es el que recibe los abonos de esa búsqueda. La portabilidad lo cambia con `actionType=MODIFY`. La DHU no usa esa frase: separa favorito y cuenta principal, y prohíbe moverlos como efecto secundario del cierre. El diseño nuevo no los fusiona. La portabilidad del directorio se dibuja donde el deck la numera (6.b) y se anota que la DHU todavía no la cubre como historia de favorito ni de cuenta principal.

---

## 7. Qué deja de valer de los diagramas del 23-nov

| Decisión del 23-nov | Qué dicen las fuentes del 25 y 26 |
|---|---|
| La creación no toca el UPI Switch | El deck dibuja `QueryDeviceDetailsReq` y `ValidateDeviceDetailsReq` hacia el switch. El alta en el directorio sigue en la afiliación |
| Un solo post-login con los siete pasos | Hay dos diagramas: sin Tapp ID (pasos 1 a 3) y con Tapp ID (pasos 1 a 7) |
| Notificación de creación la manda la billetera y la de vinculación el proveedor de cuentas | La HU 14 exige constancia en pantalla tras `SUCCESS` y envío a la bandeja. No nombra al componente que envía. Sigue pendiente |
| El paso 7 existe para avisar que no eres el principal | El deck lo mantiene como `getMapperAddressList`. La portabilidad de la afiliación usa `CHECK`, `PORT` y `MODIFY`. La DHU además separa favorito y cuenta principal |
| Réplica de cuentas cruzada con la consulta integrada | El deck la dibuja como diferencia de cuentas on-us hacia IAP, sin nombre de servicio. El contrato lo arma el equipo de VEDS con el PAC |
| Tope de SMS: cuarto intento y 24 horas, contador en el PAC | Sigue siendo lo dicho en sala para el SMS de creación. La HU 12, para clave de cajero y OTP de afiliación, habla de tercer intento y 24 horas, pendiente de validación, con bloqueo de tarjeta. No son el mismo contador |

Se conservan, porque las fuentes nuevas no las contradicen: SRM como dueño del celular, Pre-DBFD en real-time antes del token y antes de listar cuentas, post-mortem y evidencia asíncronos, API Gateway de terceros PCI en el camino al PAC, token CL por cliente, listas en caché.

---

## 8. Pendientes que no se dibujan como hechos

- Mapeo entre el catálogo UPI de la DHU (`ReqOtp`, `ReqRegMob`, `ListAccount`) y las rutas Izipay del deck (`UpiOtpRequest`, `AddAccountReq`, `UpiPinReq`, `ListAccountReq`).
- Confirmar en el PDF original que validar la clave es `/service/BalEnqReq`.
- Nombre del servicio de IAP y el contrato de diferencias de cuentas on-us.
- Quién envía la constancia a la bandeja.
- Largo de la Clave Tapp, 4 o 6.
- Permiso de SMS en Android: no está en el Figma. Puede pedirse al instalar o al usar. Si el cliente lo rechaza, hay que volver a pedirlo.
- Handoff exacto canal, Pre-DBFD y BCO. En sala quedó con duda.
- Persistencia del vínculo cuenta a Tapp ID: la DHU la marca API pendiente. El estado contra el directorio está definido solo a nivel conceptual. El mecanismo del favorito está pendiente.
- Dónde se persiste el token CL. Redis está confirmado para las listas, no para el token. La versión del 23-nov lo ponía en una base del canal cifrada. Esta reunión no lo renombró.
- Catálogo de errores: la DHU pide la matriz final. Check status y recovery están conceptuales. Feature flags están propuestos.
- Volumetría y pruebas de performance del PAC.

---

## 9. Qué debe contener cada entregable nuevo

Los cuatro se rehacen desde esta base. Los archivos del 23-nov no se pisan hasta tener el reemplazo. Contratos OpenAPI siguen fuera: no hay contrato de Izipay ni del backend.

### Alto nivel

Sistemas y límites, sin endpoints.

- Canal: cliente y App Interbank, con SDK UPI CL y SDK BCO. El ecosistema de seguridad del canal es una consideración de entrada, no una caja nueva.
- Interbank: backend en Azure, Redis de listas, SRM, Pre-DBFD, API Gateway de terceros PCI, LTD, BCO, IAP, proveedor de cuentas Interbank.
- Izipay: PAC.
- Externo: UPI Switch y directorio BCRP, proveedores de cuenta de otros bancos.
- Dos entradas al PAC: post-login sin Tapp ID y post-login con Tapp ID. La creación y la afiliación son flujos distintos, los dos con salida al switch donde el deck la dibuja.

### Bajo nivel

Componentes que el deck ya separa y que el canal construye o integra. Sin módulos de despliegue inventados.

- App: lectura de SIM, envío de SMS, SDK, pantallas de bienvenida, «Antes de comenzar», entidad, cuentas, tarjeta y clave.
- Backend SMP: orquestación, caché de listas, contraste con SRM, real-time, post-mortem, evidencia, aviso a IAP.
- Cada llamada al PAC con su ruta, síncrona o asíncrona según la sección 5.
- CBS, DCMS y ATM aparecen como capacidades que la HU 12 nombra detrás de la validación de credenciales cuando la entidad es Interbank. No se les inventa un protocolo.

### Modelo de datos

Dominio visto por Interbank. Sin esquema físico.

- Tapp ID con estados no creado, inactivo y activo. Inactivo es el estado al crearlo y al regresar antes de afiliar.
- Dispositivo y device binding, separados del dispositivo seguro del canal.
- Token CL y versión, persistencia pendiente.
- Listas de PSP y de proveedores, caché de una hora, no son datos del cliente.
- Cuenta elegible, flag `mbeba`, modo calle, afiliación.
- Alias, favorito y cuenta principal como conceptos distintos. El principal del directorio es el del deck, no el favorito de la DHU.
- OTP, últimos 6 dígitos, vencimiento y clave de cajero no se modelan como datos persistidos.
- El estado de recuperación no arrastra credenciales.

### Secuencias

Una por caso, con ramas reales. Nombres de servicio del deck. Reglas de la DHU en notas o en `alt`.

1. Post-login sin Tapp ID.
2. Post-login con Tapp ID, con los atajos de caché.
3. Creación del Tapp ID, con Android e iOS, abandono en «Antes de comenzar», y Tapp ID que queda inactivo.
4. Afiliación, con `mbeba`, modo calle, IAP sin nombre de servicio, OTP, clave, directorio y portabilidad.

El éxito de la afiliación termina en `SUCCESS` de la cuenta elegida y en la constancia. Un estado no confirmado no se dibuja como éxito.
