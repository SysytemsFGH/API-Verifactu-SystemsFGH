# 4. Entorno de Simulación y Pruebas Reales

Antes de integrar la comunicación automatizada por código (API) desde un ERP comercial, el Middleware Inteligente de VeriFactu systemsFGH provee un sofisticado **Panel de Administración Web** dotado de un **Simulador Interactivo de Envíos**.

Este simulador es el centro de operaciones perfecto para desarrolladores, consultores y técnicos a la hora de comprender la metamorfosis de los datos: cómo se construye un JSON amigable y cómo este se transforma en el ininteligible XML firmado (`XAdES`) exigido por Hacienda.

Es muy importante recalcar que **las pruebas sí se realizan contra los servidores de la AEAT en tiempo real**, atacando al entorno que tenga usted seleccionado en su configuración (que normalmente, durante la fase de integración, será el Entorno de Pruebas oficial de Hacienda). De esta forma, las respuestas obtenidas son legal y técnicamente vinculantes.

## 1. El Panel de Control Web
Acceda a la URL de administración (`http://127.0.0.1:8000/admin` o la que corresponda a su servidor de red). Navegue a la pestaña **Envíos** del menú izquierdo. Allí encontrará un área diseñada específicamente para trastear con todos los mecanismos de validación sintáctica del Middleware.

## 2. La Tercera Pestaña: "Identidades y Series"
Para que las pruebas enviadas a la AEAT se rechacen lo menos posible por causas ajenas al esquema (por ejemplo, errores de validación cruzada por intentar enviar a "Juan Nadie" con un NIF falso), el Simulador cuenta con una tercera pestaña reservada para registrar **Identidades (Contrapartes)** reales o pseudo-reales.

En este apartado preliminar usted puede:
*   Definir identidades válidas (clientes/proveedores).
*   Definir **Series** de facturación, vinculándolas a tipos de factura y descripciones concretas.
De este modo, cuando llegue el momento de crear la factura de prueba, dispondrá de desplegables ágiles (combos) y herramientas pre-armadas en vez de tener que teclear la información de los sujetos una y otra vez.

## 3. Pestaña "Generación Masiva y Pruebas"
Esta pantalla está partida funcionalmente en dos mitades que interaccionan entre sí en tiempo real. **El concepto principal aquí es que el JSON de entrada es de sólo lectura**, usted no debe escribir su estructura a mano, sino utilizar los controles de la interfaz.

### Mitad Izquierda (La Composición Guiada)
Aquí dispone de una botonera de mandos y listas desplegables para generar al vuelo el `"factura.json"` estándar.
*   **Composición Asistida:** A través de los botones y combos, usted irá seleccionando las identidades, incrementando numeraciones, cambiando el tipo de factura, o recalculando cuotas e importes globales. El panel se encarga de reestructurar y equilibrar matemáticamente el JSON para usted.
*   **Generador Fluido:** Podrá forzar diferentes casuísticas, invocar fechas o alterar la composición del desglose impositivo a un solo clic, permitiendo que el visor asimile las reglas de negocio de una manera inmensamente cómoda.

### Mitad Derecha (El XML de Salida y la Prueba de Fuego)
Una vez que el JSON se ha compuesto automáticamente mediante la botonera:
1. El motor simula el proceso interno del *Worker*. 
2. Recolecta el certificado digital que usted tiene guardado en el Alta del Emisor en el propio servidor.
3. Confecciona el XML con el estándar exacto de Hacienda VeriFactu (`suministroLR`).
4. Lo incrusta en el nodo especial, calcula el Digest SHA-256 base64 y firma legalmente el archivo resultando en el complejo `XAdES-EN` puro.

Este XML crudo recién horneado **aparecerá pintado en la mitad derecha del monitor**. 
La conexión con la AEAT le devolverá la respuesta oficial de los sistemas de Validación Tributaria.

## 4. Utilidad Real del Simulador
La existencia de este módulo no es decorativa. Cumple con 3 funciones capitales que salvarán semanas de investigación al equipo informático:

*   **Ingeniería Inversa (Reverse Engineering):** Muchos programadores prefieren moldear una factura compleja (ej. dos recargos de equivalencia simultáneos, exenciones, IRPF o rectificaciones) **con los combos y botones del Simulador**. Una vez comprueban que la AEAT "se lo traga" correctamente y les devuelve un OK verde, simplemente **copian el texto de ese JSON autogenerado de sólo lectura** y lo clonan algorítmicamente en el código C#, PHP o Python de su propio ERP.
*   **Depuración de Certificados:** Verificar que el Emisor `.p12` se configuró bien. Si Hacienda contesta al test, la conexión TLS mutua de su servidor funciona.
*   **Tranquilidad Conceptual:** Si su comprobación funciona en este entorno, cuando automatice la llamada de carga por API postulará el mismo resultado con total garantía.
