# Revisión P2P — pagos TAPP

**Interbank · 26-nov-2026.**
**Aporte:** arquitectura de canal. Este folder es independiente del onboarding del 26-nov.
**Estado:** revisión sobre el PDF y sobre la explicación en sala. No hay historias de usuario. Una arquitectura de solución formal puede llegar después; lo que hoy está fotografiado ya es el deck `TPAP - Pagos.pdf`.

## Fuentes

| Fuente | Qué fija |
|---|---|
| [documento-tpap-pagos.md](documento-tpap-pagos.md) | Láminas del PDF: roles, premisas, capacidades, flujo de siete pasos, mocks, aplicaciones, arquitectura |
| [explicacion-malena.md](explicacion-malena.md) | Lo que la sala corrige o precisa encima de esas láminas |
| [seguridad-bcrp.md](seguridad-bcrp.md) | Requisitos mínimos del marco de la Plataforma. Confidencial, uso interno |
| Onboarding ya cerrado en `docs/architecture/2026-11-26/` | Home de TAPP, cuentas recuperadas al entrar, dispositivo y Tapp ID. Este pago los consume; no los vuelve a diseñar |

## Qué pide el deck

Instruir un pago inmediato UPI desde el APP Personas. Dos lecturas: TAPP a TAPP, y TAPP a teléfono. El teléfono puede resolverse por contacto, por número, por QR o por TAPP ID.

Interbank aparece en dos roles del propio mapa: TPAP (el canal) y Proveedor de Cuentas. El PSP es Izipay. El switch es del BCRP. El PAC y el Proveedor de Cuentas se dibujan como referencia. Su diseño no es este entregable.

## Correcciones que la sala ya dejó dichas

Estas mandan sobre la lámina cuando chocan con ella. El respaldo minuto a minuto está en la explicación.

- El público de esta salida es cliente con producto Interbank. Un no cliente queda para un paso posterior, y todavía no entra en la volumetría.
- El canal no absorbe lógica de TAPP. Mantiene login, factores, evidencia, lectura de QR, notificaciones, la consulta de beneficiario y los límites que el canal ya conoce.
- En el monto, el canal valida el límite **por transacción**. El límite acumulado del canal entra al autorizar la instrucción. La lámina 8 pone los dos en el paso 2; Malena lo marca como error de la lámina.
- Elegir directorio solo ocurre si la búsqueda fue por teléfono. Con TAPP ID o con QR el beneficiario ya viene resuelto.
- La lista de cuentas cargo es la que el home de TAPP ya recuperó. El pago la muestra. No es una consulta nueva.
- Cuenta Interbank: push, token digital, notificaciones nativas, MFA y fraude en línea y posterior. Cuenta de otro banco: la lámina deja solo la Clave Tapp. Malena deja abierta la pregunta a Alina, Aranza y Rodrigo: el reglamento SBS de canales pide algo que la persona sabe y algo que tiene. Hoy eso se cubre con clave y con OTP. Con solo PIN puede faltar un factor.
- Del PAC hacia adentro, la billetera recibe el resultado. Intra TAPP pasa por BCRP y por los dos proveedores de cuenta. Fuera de TAPP, por la cámara. On-us y off-us no se redescriben aquí.
- El Proveedor de Cuentas avisa cargos y abonos. El APP manda constancia por correo cuando la operación no usó cuenta Interbank. Si usó cuenta Interbank, el aviso es el del proveedor de cuentas. El correo, en ese caso de cuenta externa, dice que el detalle lo da la entidad que debitó.
- Favoritos: la premisa los saca del alcance y la lámina 18 igual los numera. Malena cierra el soporte técnico: consulta y alta van contra la aplicación de favoritos cross canal que ya sale a producción para One App. El botón de la constancia se oculta si la persona llegó al tope. La regla fina de la zona TAPP no se diseña en este deck. Nadie respondió si el alta pide OTP.
- La respuesta del pago no la espera el teléfono hasta el final. El PAC llama de vuelta al backend del APP. Ese callback es conexión nueva. También es nueva la del backend hacia favoritos. IAP ya estaba pedido.
- El MFA de la cuenta Interbank va directo a CIAM. La arquitectura previa del canal lo hacía pasar por MPA, y esta salida no lleva MPA. El 5.2 es real time. El 5.3 es post mortem, incluido el monetario de éxito o falla.
- On-us: el PAC habla directo con el Proveedor de Cuentas. Off-us: pasa por el BCRP, y el BCRP enruta el cargo y el abono.
- Pantalla final. Éxito si el callback llega a tiempo. «En proceso» si el PAC responde eso. «Tappeo enviado» si vence el timeout del canal, sin marcarlo exitoso. Fallido: solo el mensaje de error, sin id de referencia en esa pantalla. El tope que nombra la sala para el ecosistema es 45 segundos. Si la persona cierra la app, el backend sigue con constancia, evidencia y post mortem.
- Límites en tres capas, cada uno en su sistema: canal, PAC / PSP, y Proveedor de Cuentas. El canal solo modela los suyos.
- Ecosistema: catálogo y recursos de integración exclusivos de TAPP; CICS de Plin por ahora; el C2C va al clúster nuevo. La volumetría se piensa creciente, del orden de Plin.
- MPA no entra en esta arquitectura. En la lámina de aplicaciones queda marcado To Be.

## Seguridad que este pago tiene que poder demostrar

El marco completo está transcrito en [seguridad-bcrp.md](seguridad-bcrp.md). Para el rol TPAP del canal, la matriz marca como obligatorios gobierno, riesgos, ISO 27001 y PCI si hay dato de tarjeta, privacidad, cifrado y HSM, infraestructura y logging, APIs, SDLC, seguridad móvil y SDK, device binding, y el reporte de incidentes en 1 hora y en 24 horas. El MPLS lo cumple el PAC. El POS no aplica al TPAP.

Lo que cambia el diseño del pago, y no solo el cumplimiento de la entidad:

- La Common Library es obligatoria en el TPAP. Firma las etiquetas sensibles y la firma del `Head`. El canal valida la firma de la respuesta del PAC antes de confirmarle el pago al cliente.
- El TPAP genera la firma XML y custodia su llave. No se la entrega al PAC. TLS 1.2 o 1.3 en el endpoint, de los dos lados. Suites AES-256-GCM. Prohibido TLS 1.1 y anteriores.
- No se persisten clave, OTP ni credencial dinámica, salvo que el TSD lo autorice.
- Datos de alias, teléfono, dispositivo, cuenta y transacción van cifrados en tránsito y en reposo, AES-256.
- La app lleva pinning, detección de root y jailbreak, RASP, ofuscación, tienda oficial, y bloqueo de overlay y de VPN no autorizada.
- El device binding pide datos móviles (no modo avión), token SMS de al menos 35 caracteres, y cierre del proceso en 45 segundos. Si el usuario sale de la app, la sesión de vinculación muere. El onboarding ya diseñado es el lugar de ese vínculo; el pago lo asume hecho.
- Reloj a menos de 1 segundo del NTP del PSO. Si no, el switch puede rechazar el mensaje por timestamp.
- HMAC aparece como única consideración de la lámina 19. El marco del BCRP pide firma XML y Common Library. Son dos mecanismos. Hay que decir cuál cubre la integridad del mensaje UPI y cuál, si alguno, cubre otra integración interna.
- La lámina de no financieras dice que no se manejan datos de tarjeta, y a la vez PRE MONITOR pide el PAN a TRJ. Ese PAN no sube al canal. Si en el enrolamiento o en el pago alguien persiste PAN, entra PCI DSS (5.3.2 del marco).

## Puntos abiertos

La lista única, con el responsable de cada uno, está en [pendientes.md](pendientes.md). Las historias que lleguen ajustan el diagrama del paso que contradigan.

## Qué queda fuera de este redibujo

- Interior del PAC, del switch y del Proveedor de Cuentas.
- On-us y off-us, ya tratados en el diseño de esos componentes.
- P2M, comercios y POS.
- Alta de no clientes.
- MPA.
- Administración de favoritos, más allá del signo que Malena acepta mostrar.
- Onboarding y device binding, ya diseñados. Este flujo los usa.
