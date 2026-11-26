# Diseño de onboarding y post-login

**Interbank · 26-nov-2026**
**Aporte de arquitectura de canal.** Reemplaza, para este alcance, los diagramas del 23-nov. Esos archivos siguen en `docs/architecture/` como versión anterior.

La base de decisiones está en [tapp-onboarding-base-2026-11-26.md](../tapp-onboarding-base-2026-11-26.md). Este texto dice qué se dibujó y qué no se cerró.

## Qué se hizo

Se redibujó el onboarding y el post-login con dos fuentes que el juego del 23-nov no tenía: el deck `TAPP-Onboarding.pdf` y la DHU `ETCDYP-DHU - Onboarding.pdf`. Los nombres de servicio salen del deck. Las reglas de pantalla y de cierre salen de las historias. Donde una flecha no tiene contrato, queda punteada.

| Entregable | Archivo | Qué responde |
|---|---|---|
| Alto nivel | [componentes-alto-nivel.mmd](componentes-alto-nivel.mmd) | Quién participa y en qué zona vive |
| Bajo nivel | [componentes-bajo-nivel.mmd](componentes-bajo-nivel.mmd) | Qué llama el backend, con la ruta del deck y si es síncrono o asíncrono |
| Modelo de datos | [modelo-datos.md](modelo-datos.md) | Conceptos que Interbank distingue, y lo que no se persiste |
| Secuencia | [secuencia-post-login-sin-tapp-id.mmd](secuencia-post-login-sin-tapp-id.mmd) | Entrada cuando todavía no hay Tapp ID. Solo dispositivo, llaves y token CL |
| Secuencia | [secuencia-post-login-con-tapp-id.mmd](secuencia-post-login-con-tapp-id.mmd) | Entrada con Tapp ID. Los siete pasos, con caché en las listas y en el token |
| Secuencia | [secuencia-creacion-tapp-id.mmd](secuencia-creacion-tapp-id.mmd) | Del botón Afiliarme al Tapp ID inactivo, con Android, iOS y abandono |
| Secuencia | [secuencia-afiliacion.mmd](secuencia-afiliacion.mmd) | De la entidad a la constancia, con modo calle, `mbeba` y directorio |

No hay contrato OpenAPI. Inventarlo habría ido contra la guía.

Los siete archivos se renderizaron con Mermaid CLI.

## Decisiones que ya están en los dibujos

- El post-login corre al entrar a la zona TAPP. Sin Tapp ID no se llaman listas ni directorio. Con Tapp ID sí.
- El ecosistema de seguridad del canal, incluido el dispositivo seguro, es condición de entrada. No es una caja nueva.
- Las listas de PSP y de proveedores viven en Redis, una hora, y no son datos del cliente. El token CL es por cliente. Su almacén sigue sin nombre.
- La creación compara el celular contra SRM. El real-time va antes del token. Las dos llamadas del PAC, `QueryDeviceDetailsReq` y `ValidateDeviceDetailsReq`, siguen hacia el UPI Switch. El Tapp ID nace inactivo. Abandonar en «Antes de comenzar» no persiste nada.
- La afiliación usa las rutas del deck: `ReqListAccPvd`, `ListAccountReq`, `AddAccountReq`, `UpiOtpRequest`, `UpiPinReq` y `BalEnqReq`. El directorio usa `CHECK`, el alta de celular, `PORT` y `MODIFY`.
- `mbeba=Y` pide la clave que ya existe. `mbeba=N` crea la clave. En modo calle, con entidad Interbank, solo se muestra la cuenta marcada.
- El éxito de pantalla exige `SUCCESS` de la cuenta elegida. Si no se confirma, se muestra pendiente o error.
- Favorito, cuenta principal y alias que devuelve el directorio son tres cosas distintas. Este cierre no mueve el favorito ni deduce la cuenta principal.
- Últimos 6 dígitos, vencimiento, clave de cajero y OTP no se modelan como datos guardados.

## Qué cambió respecto del 23-nov

La creación sí toca el UPI Switch en las dos llamadas de dispositivo. El alta del alias en el directorio sigue en la afiliación.

Hay dos post-login, no uno de siete pasos para todos.

La diferencia de cuentas on-us va a IAP. Ya no se dibuja como un cruce con la consulta integrada.

La constancia de la DHU se muestra tras `SUCCESS` y también va a la bandeja. El componente que la envía no está nombrado, así que no hay caja de notificaciones.

## Dudas y cosas por cerrar

1. **Mapeo de nombres.** La DHU dice `ReqOtp`, `ReqRegMob` y `ListAccount`. El deck dice `UpiOtpRequest`, `AddAccountReq`, `UpiPinReq` y `ListAccountReq`. `ReqListAccPvd` es el único que coincide. Hasta tener el mapeo, las secuencias usan el deck.
2. **`BalEnqReq`.** Es el rótulo de «validar clave» en la foto del deck. El nombre no parece el de un PIN. Hay que leerlo en el PDF original.
3. **IAP.** La flecha de diferencias on-us no trae servicio. El contrato lo está armando VEDS con el PAC.
4. **Constancia.** La HU 14 exige el envío a la bandeja y no dice quién lo hace. En la reunión, el correo de afiliación se lo atribuían al proveedor de cuentas.
5. **Token CL.** Redis está cerrado para las listas. La base del token, no. La versión del 23-nov la ponía en una base del canal y esta ronda no la confirmó ni la cambió.
6. **CBS, DCMS y ATM.** La HU 12 los nombra para cuenta activa, tarjeta y clave de cajero. El deck no muestra si los llama el backend o el PAC. Van en punteado.
7. **Tercer intento y 24 horas.** La HU 12 lo marca pendiente de validación e incluye bloqueo de tarjeta. Es otro contador que el del SMS de creación, que en sala era el cuarto intento y lo lleva el PAC.
8. **Largo de la Clave Tapp.** El mock pinta cuatro posiciones. En sala quedó 4 o 6, sin impacto en el dibujo.
9. **Permiso de SMS en Android.** No está en el Figma. Falta decidir si se pide al instalar o al enviar, y qué pasa si el cliente lo rechaza.
10. **Pre-DBFD y BCO.** El orden del deck está dibujado. El handoff fino entre canal, Pre-DBFD y BCO quedó con duda en la sala.
11. **Favorito y principal del directorio.** El deck porta el alias de la búsqueda. La DHU prohíbe cambiar el favorito y la cuenta principal como efecto del cierre, y deja el mecanismo del favorito pendiente. No se fusionaron.
12. **Persistencia del vínculo** cuenta a Tapp ID: API pendiente. Estado contra el directorio: solo conceptual. Catálogo de errores: falta la matriz. Volumetría del PAC: pendiente.
13. **Otras DHU.** Acceso a Zona Tapp, Configurar TAPP y Vinculación de Cuentas no están en este paquete. Las validaciones mínimas de acceso y las cuentas adicionales no se inventaron aquí.
