# Documentación del Middleware VeriFactu (B2B)

<div align="center">
  <br />
  <h3>🌐 Visita nuestra web oficial para más información sobre el Middleware, planes y soporte:</h3>
  <h2><a href="https://systemsfgh.com/">https://systemsfgh.com/</a></h2>
  <br />
</div>

Bienvenido al repositorio oficial de documentación de la API VeriFactu. Este repositorio contiene las guías técnicas, ejemplos de integración y la arquitectura de nuestro Middleware diseñado para facilitar el cumplimiento normativo con el entorno VeriFactu (Agencia Tributaria).

## 📚 Estructura de la Documentación

La documentación está dividida en las siguientes secciones (disponibles en la carpeta `docs/`):

1. **[Visión General del Middleware](docs/01_vision_general.md)** - Conceptos básicos y propósito del sistema.
2. **[Arquitectura de Componentes](docs/02_arquitectura_de_componentes.md)** - Diagrama general del Frontend, Backend y BD.
3. **[Conceptos y Flujo de Trabajo (Workflows)](docs/03_conceptos_flujo_de_trabajo.md)** - Cómo funciona la ingesta de facturas y los estados.
4. **[El Entorno de Simulación](docs/04_entorno_simulacion.md)** - Explicación para desarrolladores de cómo validar datos sin enviarlos a la AEAT real.
5. **[Integración de la API (REST)](docs/05_integracion_api.md)** - Referencia técnica de los _endpoints_ (Ingesta, Ack,...).
6. **[Diccionario de Datos (API y BD)](docs/06_diccionario_datos.md)** - Definición JSON de `IngestaRequest` y otros payload.
7. **[Rutas y Estructura de Proyecto](docs/07_rutas_y_estructura.md)** - Información sobre cómo está estructurado el código fuente.
8. **[Monitorización y Registro (Logging)](docs/08_monitorizacion_y_logs.md)** - Información operativa del sistema.

## 🛠️ Guías de Integración por Lenguaje (SDKs)

En la carpeta `docs/sdk_integration_guides/` encontrarás guías en formato HTML listas para visualizar desde tu navegador con ejemplos concretos de conexión a nuestra API para distintos entornos de desarrollo:

*   **Node.js**: `NODEJS_INTEGRATION_GUIDE.html`
*   **Python**: `PYTHON_INTEGRATION_GUIDE.html`
*   **C# (.NET)**: `CSHARP_INTEGRATION_GUIDE.html`
*   **PHP**: `PHP_INTEGRATION_GUIDE.html`
*   **Delphi 7 / 10+**: `DELPHI7_INTEGRATION_GUIDE.html` y `DELPHI10_INTEGRATION_GUIDE.html`

## 🚀 Empezar

Si eres nuevo en la plataforma, te recomendamos leer primero la **[Visión General](docs/01_vision_general.md)** y posteriormente revisar la guía de integración del lenguaje de programación que vayas a emplear.
