# 3. Conceptos Clave y Flujo de Trabajo (Workflow)

Antes de empezar a tirar líneas de código contra la API, es vital entender cuatro conceptos fundamentales sobre cómo el Middleware Inteligente procesa la información por debajo y qué asume de antemano.

## 1. Multi-Emisor (El NIF es la llave)
A diferencia de los sistemas tradicionales monopuesto, este Middleware es estructuralmente **Multi-Tenant (Multi-Empresa)**. 
Una sola instancia instalada de VeriFactu systemsFGH es capaz de procesar facturas para 1, 10 o 1000 empresas diferentes simultáneamente sin mezclar sus datos tributarios.

* **Cómo se logra:** El campo `emisor` (NIF) incluido en el JSON de su factura indica a qué empresa pertenece la operación.
* **El Requisito Previo:** Antes de enviar la primera factura por API de una empresa, usted (o el operador del ERP) debe "Dar de Alta" a ese NIF Emisor desde el Panel de Administración de la web o vía API. 
* **¿Qué se guarda en el Alta?** Su Certificado Digital (.p12) obligatorio para firmar ante Hacienda e información estática (Nombre, Razón Social, Máscara de Facturas). Gracias a esto, **usted nunca tendrá que incrustar ni manipular certificados P12 desde su código ERP**. El servidor lo custodiará.

## 2. Ingesta Asíncrona (El concepto de la Cola)
Cuando su ERP realiza un HTTP POST JSON al endpoint de `/ingesta`, la API de este Middleware responde de manera **casi instantánea** (aprox 5-10 milisegundos). 

¿Por qué tan rápido? Porque la inserción es **asíncrona**. 
1. La API valida estructuralmente el JSON, le asigna un número interno de seguimiento y **encola la factura**. Le dice a su ERP: *"Recibido. Me encargo de todo."* Su ERP no tiene que quedarse 2 segundos bloqueado esperando la respuesta de los servidores de Madrid.
2. Segundos después, un `Worker` (proceso en segundo plano del Middleware) la desencola, fabrica sus XML, la encadena con los hashes SHA-256 de las facturas anteriores, firma el binario con el certificado del emisor y habla con la AEAT de forma controlada y segura.

**Acerca de Múltiples Versiones y Fechas (Sobrescritura en Ingesta):**
Es importantísimo dejar claro este concepto. Su sistema ERP, TPV o software base puede lanzar facturas a nuestro Middleware **sin importar la fecha**. Estas facturas entrarán en la cola y estarán disponibles para cuando **su fecha oficial llegue** y sean lanzadas para ser procesadas oficialmente hacia la AEAT. 

Además, su ERP puede lanzar **tantas versiones de la misma factura como estime oportunas** debido a modificaciones normales u operativas antes de que la máquina las firme. El sistema, cuando llegue la hora de la verdad, **utilizará inteligentemente la versión más reciente** que usted haya inyectado.

La Ingesta define unas reglas estrictas de integridad referencial del JSON (estructura validada contra el diccionario, coherencia de cálculos impositivos, campos coherentes con la operación que se busca, etc...). Pero además, el núcleo incluye reglas definidas en su propia configuración (`modif_rules.yml`) que le permiten ser más o menos estrictos con las operaciones de modificación recibidas *incluso después de que una factura ya haya sido enviada y aceptada* por la AEAT.

Obviamente, **la AEAT siempre tiene la última palabra** sobre la validez, pero si es aceptada, la plataforma la registra como una subsanación ordinaria sin necesidad de sufrir un rechazo previo en Hacienda, trazando internamente los encadenamientos.

## 3. Modos (Estados de Operación: Alta, Baja, Modificación)
El sistema reconoce de forma nativa los tres únicos eventos que interesan a Hacienda sobre un documento:
* **(A) Alta:** Una nueva factura estándar. Se encolará para ser firmada con la marca "Alta".
* **(M) Modificación (Alta con subsanación):** Cuando una factura enviada previamente contiene un error vital o alteración de datos mayores y debe someterse a rectificación forzosa según el reglamento. Las subsanaciones frente a las rectificativas están reguladas por el reglamento de la AEAT, no obstante si se desactiva la comprobación estricta por la ingesta de cambios en las bases de cálculo cuando la modificación se hace en un lapso breve de tiempo puede pasar como una Subsanación sin necesidad de rectificativa. Es cuestión que el usuario valore todos los posibles escenarios.
* **(B) Anulación (Baja):** Cuando el documento es cancelado definitivamente.


## 4. Tipos de Entrega y "Simulador"
Aunque el middleware en producción está siempre en **Modo Automático** (toda factura inyectada se encola, se firma y se sube sola por parte del `Worker`), el sistema dispone de "ruedecillas de entrenamiento".
La interfaz web del Admin dispone de una pestaña dedicada a la "Generación y Simulación" manual. Usted podrá hacer pruebas viendo los XML resultantes generados paso a paso sin gastar timbres de red y sin alertar a Hacienda.

---

> Con estos 4 pilares comprendidos, ya estamos listos para adentrarnos en la estructura exacta del código en el próximo capítulo: *"Guía de Integración API Rest"*.
