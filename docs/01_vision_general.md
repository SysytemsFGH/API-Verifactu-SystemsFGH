# 1. Visión General del Middleware VeriFactu systemsFGH

Bienvenido a la documentación oficial del **Middleware Inteligente de VeriFactu systemsFGH**.

Este sistema ha sido diseñado específicamente para aislar y resolver el 100% de la complejidad legislativa, algorítmica y de comunicaciones que exige la normativa técnica de Sistemas Informáticos de Facturación ("VeriFactu") de la Agencia Tributaria Española.

## ¿Qué problema resuelve esta API?
Integrar la especificación completa de *VeriFactu* dictada por Hacienda directamente en un ERP o herramienta de facturación preexistente supone meses de estudio y re-ingeniería:
* Implementación de firma electrónica *XAdES-EN* incrustrada en binarios.
* Tratamiento de algoritmos hash de encadenamiento continuo (bloques de facturas referenciando matemáticamente a las huellas alteradas de facturas del pasado).
* Codificación Base64, encriptación RSA y conexiones TLS síncronas/asíncronas al entorno seguro de la AEAT. 
* Construcción hiperestricta de árboles XML complejos con multitud de regulaciones fiscales, validaciones de cuotas, trazabilidad de operaciones por fechas con recargos de equivalencia, IVA y sus complejas restricciones de campos.
* Gestión de reintentos, tiempos de espera, rechazos tributarios con código específico AEAT e integridad relacional a nivel del operador.
* Gestión segura y trazable del archivo en frío ("modo retención local sin envío automático").

## Filosofía del Desarrollo: Cero Fricción al ERP
Como ingenieros y consultores, nuestro principio directivo para este Middleware es **Cero Fricción (`Zero Friction`)** a la hora de inocular sus capacidades en cualquier sistema de facturación.  

Al instalar este Middleware (que opera silenciosamente como un servicio en su Servidor o de forma alojada), usted, como compañía desarrolladora de software o equipo de integración, simplifica su mundo a un simple formato entendible globalmente: **Una llamada JSON tradicional de tipo "dispara y olvida" a nuestra API REST local**. 

El flujo global se resume en 2 pasos:
1. Su software (escrito en cualquier lenguaje: PHP, Delphi, Python, .NET...) finaliza su proceso natural de facturación que ya tiene.  
2. Su software realiza un POST con un plano JSON básico y limpio (`factura_simple.json`) a nuestra URI residente `http://hostname:8000/ingesta/`. ¡Y termina su trabajo!

## Identidad del Software ante la AEAT
Es vital dejar claro que **el usuario o empresa que instala y configura esta API se convierte en el "dueño y señor" absoluto de los envíos ante la Agencia Tributaria**. 

Este Middleware actúa puramente como un motor tecnológico en la sombra. A través de la pestaña de configuración de este Panel de Administración, en el bloque **"Identificación del Software"**, usted definirá parámetros como:
* Su propia **Razón Social (Desarrollador)** y **NIF (Desarrollador)**.
* El **Nombre del Sistema** comercial (el nombre de su propio ERP) y su **Versión**.

Al inyectar estos datos, **a todos los efectos legales y técnicos frente a la AEAT, es como si usted hubiera programado y montado todo el ecosistema VeriFactu desde cero dentro de su ERP**. Sus clientes y Hacienda verán exclusivamente la marca, el NIF y el prestigio de su empresa desarrolladora, sin que quede rastro de nuestro Middleware intermedio en el XML firmado.

Internamente, tras esta máscara legal, este Middleware recibirá su carga. Verificará de golpe todos los importes, calculará y fabricará sin errores su encadenamiento tributario en cascada, estampará firmas legalmente válidas y dialogará con Hacienda ininterrumpidamente recuperando el C.S.V. final. Todo el rastro y auditorías quedarán permanentemente protegidas y reflejadas en tiempo real en este mismo Panel de Administración para que la infraestructura técnica de la AEAT no le quite tiempo y se concentre en su negocio funcional.

---

> Continúe leyendo el capítulo _"Integración API"_ para descubrir la llamada específica con la que se insertan facturas en milisegundos.
