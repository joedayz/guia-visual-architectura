# Marco de seguridad — Plataforma de Pagos Minoristas (BCRP)

**Uso interno de arquitectura. Interbank · 26-nov-2026.**
**Fuente:** fotos `seguridad/IMG_0243.HEIC` a `IMG_0271.HEIC`. El visor muestra el membrete **tapp | Transferencias Automáticas de Pagos Peruanos**. El cuerpo lo nombra *Marco de Cumplimiento de Seguridad de la Información de la Plataforma*. El operador (PSO) es el BCRP. Se leen «Página 21 de 32» y «Página 26 de 32».
**Cobertura de las fotos:** 29 páginas fotografiadas. El PDF declara 32. No están en la carpeta la carátula completa, y hay un salto entre los criterios de excepción y el Anexo 2. Donde la foto corta una frase, queda marcado.

Este archivo es la transcripción de trabajo para diseñar pagos P2P. El original dice que es confidencial, de propiedad del BCRP, entregado solo a participantes de la Plataforma, y que no se reproduce ni se publica.

---

## Declaración del documento

El contenido es confidencial y de propiedad del Banco Central de Reserva del Perú. Está destinado a los participantes de la Plataforma de Pagos Minoristas, para facilitar su conexión y operación. Se entrega con fines informativos y circulación privada, protegido por el acuerdo de confidencialidad del receptor. No se reproduce, redistribuye, publica ni comparte. El uso no autorizado está prohibido. La información puede cambiar sin aviso. BCRP 2026, todos los derechos reservados.

---

## 1. Introducción

La Plataforma de Pagos Minoristas es una infraestructura de pagos inmediatos en tiempo real, operada por el BCRP. Integra cuentas de depósito y de dinero electrónico en una aplicación móvil de cualquier entidad participante. Cubre transferencias inmediatas de fondos, pagos a comercios y la solicitud de cobros entre personas y comercios.

Ofrece protocolos abiertos y especificaciones estándar de API. El objetivo es una interfaz segura y eficiente, interoperabilidad, menos fricción entre participantes y una experiencia homogénea, con seguridad, disponibilidad y resiliencia.

El documento fija requisitos mínimos obligatorios de:

- gobierno y gestión de la seguridad
- protección y tratamiento de datos
- cifrado de información y canales
- controles sobre APIs y claves criptográficas
- seguridad de infraestructura, aplicaciones, SDK y dispositivos
- monitoreo, continuidad, gestión de riesgos
- auditoría y certificación

Aplican desde la incorporación y durante toda la permanencia. Son la base de la evaluación de cumplimiento, las revisiones de seguridad y la supervisión del PSO y de las autoridades. Las secciones siguientes detallan las obligaciones para mantenerse conectado.

## 2. Glosario

- **Participante:** entidad financiera o no financiera (Miembro, Submiembro, PAC o TPAP) autorizada por el PSO para operar en la Plataforma.
- **PSO (Payment System Operator):** operador del sistema de pagos. Lo desempeña el BCRP.
- **CISO:** responsable de alto nivel designado por el Participante para coordinar la seguridad de la información y la ciberseguridad.
- **TSP (Technology Service Provider):** proveedor tecnológico (nubes, centros de datos, procesadores) que soporta la infraestructura, bajo responsabilidad integral del Participante.
- **API:** definiciones y protocolos de interacción técnica entre los sistemas de los Participantes y la Plataforma.
- **SDK:** kit que entrega el PSO y que debe integrarse en las aplicaciones móviles para habilitar funciones de seguridad y operación.
- **Common Library (CL):** biblioteca común de uso obligatorio para el firmado de mensajes y la protección de etiquetas de datos sensibles, conforme al TSD (Documento de Especificaciones Técnicas).
- **Device Binding:** vínculo unívoco entre la identidad del usuario y un dispositivo móvil, con controles criptográficos y de red.
- **NTP:** sincronización de relojes de los sistemas.
- **Cifrado en reposo:** protección de datos almacenados con algoritmos criptográficos.
- **Cifrado en tránsito:** protección de la información en la red (TLS 1.2 o superior).
- **HSM:** dispositivo de hardware que gestiona, procesa y almacena claves.
- **Superficie de exposición:** puntos de entrada, protocolos y servicios por los que un no autorizado puede ingresar o extraer datos.
- **VAPT:** evaluación de vulnerabilidades y pruebas de intrusión.
- **Utilización de servicios:** interacción técnica y operativa de los Participantes con la Plataforma. El texto excluye una lectura de «relación de consumo» bajo el derecho del consumidor.
- **Derechos ARCO:** acceso, rectificación, cancelación y oposición de los titulares de datos personales.
- **Firma digital XML:** estándar de integridad, autenticidad y no repudio de los mensajes XML.

## 3. Objetivo

Establecer los requisitos mínimos obligatorios de seguridad y cumplimiento de todas las entidades de la Plataforma, para:

- proteger confidencialidad, integridad y disponibilidad de la información y de las transacciones
- reducir la superficie de exposición a ciberataques, fraudes y accesos no autorizados, con controles técnicos, organizativos y procedimentales
- mantener una postura alineada a ISO 27001, PCI DSS, OWASP y a la regulación del BCRP y demás autoridades
- dar una base común a la auditoría, la certificación, la gestión de riesgos y la supervisión

## 4. Aplicabilidad

- **Ámbito funcional:** componentes, procesos y flujos de pagos inmediatos, incluidas transferencias entre personas, pagos a comercios, solicitudes de cobro y los casos de uso que el PSO habilite.
- **Ámbito tecnológico:** front-end y back-end, APIs, SDK, dispositivos móviles, servidores, bases de datos, redes, firewalls, WAF, EDR, monitoreo, registro y continuidad vinculados a la Plataforma.
- **Ámbito organizacional:** quienes operan, proveen o consumen servicios y procesan, transmiten o almacenan información de esas operaciones. Enuncia a proveedores de cuentas (miembros y submiembros), PAC que ofrecen aplicaciones u otros canales, TPAP que desarrollan u operan aplicaciones o componentes, y TSP (centros de datos, nubes, procesadores, integradores).

Obliga desde la solicitud de incorporación y conexión hasta el offboarding. El PSO o las autoridades pueden sumar requisitos por tipo de entidad o modelo de negocio. Cada participante cumple según su rol, conforme a la matriz del Anexo 1.

## 5. Lineamientos de gobierno y gestión

### 5.1. Gobierno de la seguridad y ciberseguridad

**5.1.1.** Política de Seguridad de la Información y Ciberseguridad, aprobada por el máximo órgano de gobierno (Directorio o Consejo). Debe establecer objetivos, alcance, roles y responsabilidades, estructura de seguridad, tratamiento de excepciones, revisión de cumplimiento y medidas por incumplimiento. Busca el compromiso de la Alta Dirección con la salvaguarda de la información, la integridad de las operaciones y la confianza, y la identificación de los roles de la política del banco.

**5.1.2.** Designar un CISO, o equivalente, que coordine la gestión de seguridad en toda la organización, mida la eficacia de los controles y reporte a los comités de alto nivel.

**5.1.3.** Garantizar recursos humanos, tecnológicos y presupuestales para la gestión de seguridad de la información y ciberseguridad.

### 5.2. Gestión de riesgos y vulnerabilidades

**5.2.1.** Proceso de gestión de riesgos de ciberseguridad y seguridad de la información, alineado a NIST e ISO 27001.

**5.2.2.** Evaluaciones de riesgo que documenten impacto al negocio, probabilidad y criticidad de los activos.

**5.2.3.** Extender la gestión de riesgos a los proveedores TI críticos con acceso a infraestructura que interactúe con la Plataforma.

**5.2.4.** Programa de gestión de vulnerabilidades sobre infraestructura, aplicaciones y APIs.

**5.2.5.** Remediar vulnerabilidades según CVSS versión 4. **Críticas: máximo 15 días calendario. Altas: máximo 30 días calendario.**

**5.2.6.** Reportes regulares a la Alta Dirección sobre la evolución de los riesgos y la efectividad de los controles.

**5.2.7.** Revisiones de seguridad de aplicaciones y componentes que interactúan o se integran con la Plataforma: SAST, DAST y evaluación de API. Semestral, o ante un cambio significativo de código, arquitectura o integraciones. Los hallazgos se cierran según criticidad y los plazos de la política de la entidad.

### 5.3. Certificaciones y cumplimiento normativo

**5.3.1.** Framework de gestión de ciberseguridad y seguridad de la información alineado a **ISO/IEC 27001:2022** (o equivalente). El alcance incluye todos los componentes de la Plataforma.

**5.3.2.** Controles de ciberseguridad y seguridad de la información según **PCI DSS** si se procesan, transmiten o almacenan datos de tarjetas en cualquier etapa del flujo de pagos o del enrolamiento.

### 5.4. Auditoría y reporte al PSO

**5.4.1.** Reportes anuales de cumplimiento ante el PSO. El periodo fiscal va del **1 de abril al 31 de marzo**.

**5.4.2.** Evaluación inicial, con los hallazgos, a más tardar el **31 de mayo** de cada año.

**5.4.3.** Reporte final que demuestre el cierre total de hallazgos críticos y altos, a más tardar el **31 de julio**.

**5.4.4.** En el enrolamiento inicial y en la validación anual, el CISO remite al PSO la **Carta de Cumplimiento de Seguridad** del Anexo 2, con el reporte de auditoría y el checklist.

## 6. Protección de datos y estándares criptográficos

### 6.1. Privacidad y protección de datos

**6.1.1.** Cumplir la normativa de protección de datos personales y privacidad, y las disposiciones del PSO.

**6.1.2.** Obtener y conservar evidencia del consentimiento expreso e informado para recopilar y tratar datos personales y financieros. Plazo de conservación: **no menor a 10 años**.

**6.1.3.** Procedimientos para que el cliente ejerza derechos ARCO, claros, accesibles y comunicados, sin afectar la seguridad o integridad de la Plataforma.

**6.1.4.** Políticas de eliminación o anonimización segura cuando se cumple la finalidad o vencen los plazos legales.

**6.1.5.** Cifrado robusto, en tránsito y en reposo, de datos personales y financieros: identificadores de cliente, cuentas de fondos, alias (en la foto se lee «alias/DPI»), números de teléfono, identificadores de dispositivo y detalle de transacciones.

**6.1.6.** No almacenar ni registrar factores de autenticación sensibles: claves de acceso, OTP o credenciales de autenticación dinámica, salvo autorización expresa del PSO en el TSD.

**6.1.7.** La comunicación de datos sensibles entre PAC y TPAP cumple los mismos estándares de cifrado y protección que la conexión directa con la Plataforma. El cuerpo remite a un «Anexo 2»; el anexo fotografiado con esas reglas es el **Anexo 3**. Queda la diferencia de número.

### 6.2. Estándares de cifrado y gestión criptográfica

**6.2.1.** Cifrar en reposo toda la información sensible (datos personales y financieros, transacciones y cualquier atributo que identifique al usuario) con **AES y claves de 256 bits como mínimo**.

**6.2.2.** Toda comunicación con la Plataforma por canales cifrados con **TLS 1.2 o superior**, solo con los cipher suites que el PSO autorice en el TSD.

**6.2.3.** Llaves y secretos en **HSM certificado FIPS 140-3 nivel 3**.

**6.2.4.** Control dual de llaves y rotación en plazos **no mayores a seis meses**.

## 7. Seguridad de la infraestructura y conectividad

### 7.1. Conectividad y perímetro de red

**7.1.1.** Conexión con la Plataforma **exclusivamente por la red MPLS de Asbanc**, con enlaces seguros y dedicados autorizados por el PSO, aislada de otros servicios. Los estándares criptográficos de esa conexión se detallan, según este numeral, en el Anexo 3. En las fotos, el anexo de criptografía y TLS es el **Anexo 4**, y el Anexo 3 es la comunicación PAC–TPAP.

**7.1.2.** Listas blancas de IP en ambos extremos. Solo tráfico desde y hacia los puntos autorizados.

### 7.2. Gestión de infraestructura y disponibilidad

**7.2.1.** Hardening de servidores, bases de datos y dispositivos de red, alineado a NIST e ISO 27001.

**7.2.2.** Sincronizar todos los sistemas de la Plataforma por NTP. La fuente de tiempo oficial es la de la **Marina de Guerra del Perú**.

**7.2.3.** Seguridad física de centros de datos y salas de comunicaciones: acceso solo a personal autorizado y registro de ingreso auditable.

**7.2.4.** Alta disponibilidad y redundancia, sin punto único de falla que corte los pagos inmediatos. El PSO recomienda **al menos dos datacenters**.

### 7.3. Registro, monitoreo y gestión de incidentes

**7.3.1.** SOC propio o tercerizado, con SIEM que recolecte eventos de firewalls, VPN, servidores, aplicaciones y demás soluciones de la Plataforma.

**7.3.2.** Registrar como mínimo: intentos de acceso fallidos, cambios de configuración de red, acceso a llaves criptográficas y alertas de los sistemas de detección de intrusos.

**7.3.3.** Integridad de los registros de auditoría: protegerlos contra alteración o eliminación no autorizada.

**7.3.4.** Alertas automáticas ante comportamiento anómalo: picos de tráfico en APIs, múltiples intentos fallidos de Device Binding.

**7.3.5.** Monitoreo continuo SOC / SIEM / XDR, análisis de alertas críticas y respuesta inicial, **24x7**.

**7.3.6.** Playbooks de detección, contención, erradicación y recuperación.

**7.3.7.** Reportar al PSO todo incidente tecnológico o de ciberseguridad que afecte o pueda afectar la operación, en un plazo **no mayor a 1 hora** desde la identificación, con el formato del anexo de reporte inicial. El cuerpo lo llama Anexo 4; las fotos lo titulan **Anexo 5**.

**7.3.8.** Plan de acción de mitigación en un plazo **no mayor a 24 horas**.

## 8. Seguridad de aplicaciones y canales digitales

### 8.1. Seguridad de APIs

**8.1.1.** Controles específicos en todas las APIs que interactúan con la Plataforma, incluido **rate limiting** contra abuso y DDoS.

**8.1.2.** Validación y sanitización de todos los parámetros de entrada, contra inyección y manipulación, conforme al **OWASP API Security Top 10** vigente.

**8.1.3.** Respuestas de error que no revelen infraestructura, rutas internas ni detalle de bases de datos.

**8.1.4.** Autenticación y autorización con **scopes** definidos por caso de uso y por el TSD.

### 8.2. Ciclo de vida de desarrollo seguro

**8.2.1.** SAST antes de la compilación y DAST en entornos de prueba.

**8.2.2.** SCA para CVE en bibliotecas de terceros y código abierto.

**8.2.3.** Revisiones integrales de seguridad **semestrales**, o ante un cambio significativo de código o arquitectura.

### 8.3. Seguridad en aplicaciones móviles y SDK

**8.3.1.** Detección de rooting y jailbreak, y **RASP** contra inyección de código o alteración de memoria.

**8.3.2.** **SSL/TLS pinning**, para que la app solo hable con servidores autorizados y se mitigue MITM.

**8.3.3.** Bloqueo de depuración y ofuscación contra ingeniería inversa.

**8.3.4.** Distribución solo por tiendas oficiales, y forzar la actualización a la última versión.

**8.3.5.** Detección y prevención de keyloggers y screen overlays, para que una app maliciosa no capture credenciales.

**8.3.6.** Si hay lectura automática de OTP, validar el remitente y procesar solo mensajes de fuentes autorizadas por el PSO.

**8.3.7.** Detección de proxies y VPN no autorizadas, con bloqueo o alerta.

**8.3.8.** Cobertura contra el **OWASP Mobile Top 10** vigente, en la app y en el SDK.

**8.3.9.** Integrar SDK y bibliotecas de la Plataforma solo conforme al TSD.

Además, la entidad participante debe:

- mantener los SDK en la última versión estable que libere el PSO
- remediar una vulnerabilidad crítica en componentes o bibliotecas de terceros en **máximo 15 días calendario**
- remediar una de severidad alta en **máximo 30 días calendario**
- no usar versiones de SDK que el PSO haya declarado obsoletas

### 8.4. Controles adicionales de resiliencia (recomendados)

**8.4.1.** Detectar y bloquear dispositivos virtuales o emuladores.

**8.4.2.** Listas de exclusión de dispositivos ya identificados como comprometidos o ligados a fraude.

**8.4.3.** APK locking, para impedir versiones modificadas o clonadas de la app oficial.

### 8.5. Device Binding

**8.5.1.** El dispositivo debe tener **conectividad de datos móviles activa durante todo el proceso de vinculación**. Se prohíbe el registro en modo avión.

**8.5.2.** Tokens SMS dinámicos para la vinculación, **longitud mínima de 35 caracteres alfanuméricos**.

**8.5.3.** El proceso completo de vinculación **no excede 45 segundos**. La sesión se invalida si el usuario abandona la aplicación durante el proceso.

**8.5.4.** Controles antifraude que incluyan el bloqueo temporal del dispositivo tras intentos fallidos y la validación de que el… **La foto corta la frase.**

## 9. Seguridad en puntos de venta (POS)

Aplica a entidades adquirentes que faciliten pagos de la Plataforma con POS.

### 9.1. Controles físicos y de hardware

**9.1.1.** Puertos USB no utilizados, bloqueados física o lógicamente, para impedir periféricos no autorizados.

**9.1.2.** Lista blanca de dispositivos USB. Solo componentes aprobados por la entidad.

**9.1.3.** Ranuras SD desactivadas, salvo caso excepcional justificado y aprobado por el PSO.

El texto añade detección de intrusión física (tamper) con alerta inmediata al centro de monitoreo. En la foto ese párrafo no lleva numeral propio; queda entre 9.1 y 9.2.

### 9.2. Protección de datos y comunicaciones

El título de la foto se lee «9.2. 8.2 Protección de datos y comunicaciones». Se transcribe como 9.2.

**9.2.1.** Cifrar información personal identificable y datos sensibles de pago en el POS, como mínimo **AES-256**.

**9.2.2.** Comunicaciones del POS con **TLS 1.2 o superior**. El numeral remite a la «sección 5.2»; en el cuerpo fotografiado el cifrado está en **6.2**.

## Anexo 1. Matriz de aplicabilidad por rol

Cada entidad identifica su rol e implementa lo marcado **O** (obligatorio). **R** es recomendado.

Roles del anexo:

- **PAC:** interfaz directa al usuario y conexión técnica con la Plataforma.
- **TPAP:** usa la infraestructura de un PAC para ofrecer servicios de pago (el ejemplo del anexo es una billetera móvil de un tercero).
- **Proveedor de Cuentas (Miembro/Submiembro):** entidad financiera o EEDE donde residen los fondos.

| Sección, tal como la numera el anexo | PAC | TPAP | Proveedor de Cuentas |
|---|---|---|---|
| 4.1 Gobierno de la seguridad | O | O | O |
| 4.2 Gestión de riesgos y vulnerabilidades | O | O | O |
| 4.3 Certificaciones (ISO 27001 / PCI) | O | O | O |
| 5.1 Privacidad y protección de datos | O | O | O |
| 5.2 Estándares de cifrado y HSM | O | O | O |
| 6.1 Conectividad (MPLS / VPN IPsec) | O | N/A* | O |
| 6.2 Infraestructura (hardening / NTP) | O | O | O |
| 6.3 Registro y monitoreo (logging) | O | O | O |
| 7.1 Seguridad de APIs y OWASP | O | O | O |
| 7.2 Ciclo de vida de desarrollo (SDLC) | O | O | O |
| 7.3 Seguridad móvil y SDK | O | O | N/A |
| 7.4 Vinculación de dispositivos | O | O | N/A |
| 8.1 Seguridad en puntos de venta | R | N/A | N/A |
| 9.1 Reporte de incidentes (1 h / 24 h) | O | O | O |

\* El MPLS del TPAP lo cumple el PAC que le da el acceso.

La numeración del anexo no coincide con la del cuerpo (gobierno es 5.1 en el cuerpo y 4.1 en el anexo; el incidente es 7.3 en el cuerpo y 9.1 en el anexo). Se deja así, como está en las fotos.

### Proveedores tecnológicos (TSP)

El TSP no es sujeto directo de este marco ante el PSO. Aun así:

- **Responsabilidad integral.** PAC, TPAP y Proveedor de Cuentas responden por la seguridad de lo que delegan.
- **Deber de auditoría.** Deben auditar que el proveedor cumpla custodia de llaves, cifrado y desarrollo seguro.
- **Derecho de inspección.** Los contratos deben permitir que el PSO o las autoridades revisen la seguridad del tercero si la estabilidad de la Plataforma se ve comprometida.

### Criterios de excepción

Cualquier excepción técnica se solicita al PSO, con análisis de riesgo y un plan de controles compensatorios firmado por el CISO.

## Anexo 2. Carta de cumplimiento

Modelo **versión 1.0**. Título: Carta de cumplimiento, Marco de seguridad, Plataforma de Pagos Minoristas.

Campos: fecha, encabezado y logo de la firma auditora, teléfono, correo, entidad auditada (nombre y RUC), PAC asociado si es TPAP/ASP, funcionalidades (`P2P | P2M | Cobros`), periodo inicial y periodo final.

Evaluación inicial, máximo 31 de mayo. Reportes:

- inventario de activos de la Plataforma, con valor y criticidad
- VAPT y auditoría de configuración, con hallazgos abiertos
- seguridad de aplicaciones y revisión de código (OWASP/SANS)
- revisión de arquitectura, hallazgos abiertos
- análisis de aplicaciones (OWASP/SANS)
- registro de riesgos y plan de acción con fechas
- excepciones y controles compensatorios
- ISO 27001:2022 con la Plataforma en el alcance
- PCI-DSS AOC, si aplica

Evaluación final, máximo 31 de julio. Cierre total de:

- VAPT y auditoría de configuración
- seguridad de aplicaciones y revisión de código
- revisión de arquitectura
- análisis de APIs

También un registro de riesgos actualizado.

Texto modelo del auditor, evaluación inicial: la evaluación se completó; todos los hallazgos tienen plan y fecha, revisados con evidencia de la entidad.

Texto modelo, evaluación final: todos los hallazgos iniciales cerrados, sin excepciones; la entidad cumple el marco.

Firmas: auditor responsable (firma digital o sello, fecha) y revisión de Cumplimiento del PSO.

## Anexo 3. Comunicación entre PAC y TPAP

Controles obligatorios del intercambio PAC–TPAP, en cumplimiento del TSD.

### Canal y conectividad

- **Cifrado de transporte.** Toda comunicación entre servidores del TPAP y del PAC por canales seguros, **TLS 1.2 o superior**.
- **Control de acceso por IP.** El PAC aplica listas blancas y solo acepta las IP públicas registradas y autorizadas por el TPAP.

### Integridad y no repudio

- **Firma digital XML.** Todos los mensajes, petición y respuesta, firmados con XML Digital Signature.
- **Common Library.** El TPAP integra la CL de forma obligatoria para firmar las etiquetas de datos sensibles y para que la firma del encabezado (`Head`) cumpla lo que exige el PSO.
- **Custodia de claves.** El TPAP genera y custodia sus llaves privadas. Van en un HSM o en un área protegida. No se comparten con el PAC.

### App móvil y SDK

- El TPAP usa el SDK de la Plataforma en todas las operaciones de seguridad del dispositivo, en especial para la firma digital que valida la identidad del usuario y del dispositivo.
- El TPAP valida la firma digital del PAC en todos los mensajes de respuesta, antes de procesar la información o de confirmar la transacción al usuario.

### Matriz técnica

| Requisito | TPAP | PAC |
|---|---|---|
| Generación de firma XML | X | |
| Validación de la firma del mensaje | | X |
| TLS en el endpoint de la API | X | X |
| Gestión y custodia de llaves privadas | X | X |
| Disponibilidad del servicio de gateway | | X |

### Auditoría y registros

PAC y TPAP guardan logs de todas las comunicaciones, con identificador de mensaje (`msgId`) y sello de tiempo (`ts`), para disputas e incidentes. El anexo remite a la «Sección 9» del cuerpo; en las fotos el registro está en **7.3**.

## Anexo 4. Criptografía y configuración técnica

Configuración obligatoria de la infraestructura interna para hablar con la Plataforma por la red de Asbanc. El anexo dice que corresponde a las secciones 5 y 6 del cuerpo.

### Interfaz de red y perímetro

La red MPLS y la terminación de la VPN son de Asbanc. El participante asegura sus dispositivos internos de borde:

- ACL estrictas: solo segmentos autorizados por el PSO y por Asbanc
- la inspección de tráfico del perímetro no degrada ni altera las firmas XML que van a la Plataforma

### TLS

Toda comunicación HTTPS que se origine o termine en la infraestructura del participante:

- versiones permitidas: **TLS 1.2 o TLS 1.3**
- prohibidos: SSL v2, SSL v3, TLS 1.0 y TLS 1.1
- cipher suites obligatorios:
  - `TLS_AES_256_GCM_SHA384`
  - `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
  - `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`

### Integridad de mensajes (TSD)

- Firma XML con **SHA-256 o superior** para el digest. La foto se lee «SHA-256 0»; se toma como SHA-256 o superior.
- La Common Library asegura que el firmado de etiquetas críticas use la criptografía asimétrica del TSD.

### NTP

- Sincronizar con el servidor NTP que indique el PSO.
- Desviación máxima permitida: **1 segundo** respecto del tiempo oficial. Sirve para que el mensaje no se rechace por validación de timestamp.

## Anexo 5. Reporte de incidentes

Formato mínimo hacia el PSO ante un evento que afecte la continuidad o la seguridad. El anexo lo ancla en la «Sección 9.3»; en el cuerpo fotografiado el plazo está en **7.3.7 y 7.3.8**.

### Reporte inicial (máximo 1 hora)

Por los canales oficiales (correo). Asunto: `[INCIDENTE] - [Nombre de la Entidad] - [Clasificación de Impacto]`.

- ID interno del incidente
- fecha y hora de detección en **ISO 8601** (ejemplo de la foto: `2026-01-27T08:45:00-05:00`)
- tipo: interrupción de servicios cloud, fallo en el firmado XML, indisponibilidad de API, DDoS, fuga de datos
- componentes afectados: SDK de la Common Library, endpoint de pagos, módulo de liquidación
- impacto preliminar: degradación, interrupción total, afectación a un porcentaje de usuarios
- punto de contacto 24/7: nombre y teléfono

### Informe de mitigación y plan de acción (máximo 24 horas)

- análisis de causa raíz preliminar: vector de ataque o falla técnica
- contención ya realizada (bloqueo de IP, aislamiento, rotación de llaves)
- plan de recuperación y cronograma para volver a la operación normal
- indicadores de compromiso, para que el PSO pueda alertar a otros participantes: IP atacantes, hashes, dominios
- cronología desde la detección hasta la recuperación
- medidas preventivas para evitar la recurrencia
- evidencia de cierre: integridad de sistemas y validación de firmas digitales tras la limpieza

### Informe de lecciones aprendidas

Documento formal al cierre técnico del incidente. La foto lo enuncia y no desarrolla apartados adicionales.
