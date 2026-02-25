# 8. Monitorización, Auditoría y Logs (Panel de Administración)

El Middleware VeriFactu systemsFGH es un motor robusto y silencioso que funciona en segundo plano ("Demonio" o "Servicio"), pero está acompañado de un moderno **Dashboard de Administración Web** (este panel) que le permite tener control absoluto y visión de rayos X sobre todo lo que ocurre.

Al no depender de componentes cerrados y ofuscados, usted puede supervisar su facturación legal en tiempo real y diagnosticar fallos al instante.

---

## 1. El Panel Principal (Monitor de Tráfico)

Al entrar en el panel, la tabla central de **"Estado de Transacciones"** se encarga de mostrar la realidad asíncrona de los envíos a la AEAT. 

### Interpretación de los Estados
Cuando su ERP inyecta una factura (`POST /ingesta`), esta aparece inmediatamente en la tabla. Observe la columna **"Estado AEAT"**:
*   **🔵 `P` (Pendiente):** Significa que el Middleware ya ha atrapado su JSON y ha fabricado el XML, pero la conexión con Hacienda aún está encolada (el "Worker" asíncrono no la ha evacuado todavía).
*   **🟢 `0` (Aceptada / OK):** La comunicación con la AEAT fue exitosa. La factura es cien por cien legal. Se le ha asignado el CSV correspondiente.
*   **🟡 `1` (Aceptada con Errores):** Hacienda ha tragado el documento, pero emite un aviso (suele ser por temas de censos autonómicos, direcciones raras, etc.). Es legal, pero conviene revisarla.
*   **🔴 `3` (Rechazada / Error):** La estructura del XML es incorrecta (fallos de NIF no existentes, sumas matemáticas erróneas de desglose, hashes que no cuadran). Hacienda repudia el envío.

A través de esta tabla usted y sus técnicos saben al segundo si la facturación de su cliente está retenida o si fluye hacia el servidor gubernamental sin problemas.

---

## 2. Auditoría Forense: Los 3 Niveles de XML

El Middleware VeriFactu systemsFGH es insuperable en transparencia gracias a su **Exportador de Evidencias (Backup Operador)**. Si una factura resulta `Rechazada` o necesita aportar pruebas ante la Agencia Tributaria en caso de conflicto, la tabla web le permite acceder al "ADN" de cada envío.

### ¿Cómo descargarlo en el Panel?
Busque en la sección de mantenimiento o en el registro visual la función de descargar o expandir su archivo de Backup.

Para proteger la inviolabilidad legal impuesta por la ley antifraude, los archivos generados viajan **criptográficamente sellados en Base64**. Nuestro motor almacena **3 niveles** de información bruta para cada registro para que usted pueda depurar el error o comprobar los XML exactos:

1.  `json_original_det` *(Vía JSON)*: Es una copia del texto JSON crudo que mandó su programa. Si su ERP fallaba al generar un decimal o enviaba un documento de "Abono" en vez de "Registro", aquí tiene la prueba original irrefutable.
2.  `xml_log` *(Vía XML Crudo)*: Es el andamiaje del árbol que fabrica nuestro Middleware.
3.  `xml_log_firmado` *(Vía XML Sellado)*: **El Archivo Custodia.** Este documento **no viaja a la AEAT en el momento**. Queda almacenado en la base de datos para que los sistemas configurados como "No VeriFactu" lo extraigan mediante backup y lo custodien de forma segura. En este modo de retención local, los registros se firman criptográficamente uno por uno. Si se respeta su orden de salida secuencial, el encadenamiento de *hashes* es idéntico y tan válido legalmente como el de los envíos directos, listos para juntarse cronológicamente y enviarse a la AEAT únicamente en caso de requerimiento expreso.

> *Nota Técnica:* Si usted extrae el `xml_log_firmado` de la capa de BD, verá una larga cadena de texto inconexa y aparentemente sin sentido (Ej: `PD94bWwgdm...`). Esto es Base64. Utilizando la línea de comandos de su sistema operativo (`base64 -decode archivo.txt`) o conversores universales de texto-a-binario, obtendrá el archivo `.xml` digital impoluto y nativo preparado para llevar ante el supervisor fiscal. *(Consulte su correspondiente guía de recuperación de XML que le enseñará como desencapsular Base64).*

---

## 3. Logs de Mantenimiento (`sys_logs.txt`)

Usted, como instalador, será quien afronte los errores de certificado digital, los bloqueos de Firewall y los reinicios del sistema.

Para evitar adivinar qué ocurre "por debajo", existe una carpeta reservada de diagnósticos llamada `/logs/`.

Desde la pestaña de conectividad/mantenimiento, su ojo técnico principal será el archivo de servidor general, a menudo denominado `sys_logs.txt` o similar: 
*   **Problemas de Certificado (.p12):** Le cantará enérgicamente si la ruta hacia el certificado AEAT es incorrecta, si requiere PIN/contraseña desactualizado o si el certificado oficial caducó en el sistema.
*   **Caídas Temporales de Hacienda:** Le notificará de códigos `HTTP 502/503` cuando los portales de la Agencia Tributaria caigan en mantenimiento las noches de fin de semana impidiendo que el motor pueda trabajar.
*   **Errores Transaccionales:** Excepciones graves de PyDantic, reinicios de demonios internos, etc.

Si su sistema está configurado y los JSON de inicio emiten luz verde, este será un panel totalmente "aburrido" que usted revisará eventualmente sabiendo que tiene bajo el brazo un blindaje absoluto de sus obligaciones antifraude.
