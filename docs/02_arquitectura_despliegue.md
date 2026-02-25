# 2. Arquitectura de Despliegue y API

Una de las grandes ventajas de **VeriFactu systemsFGH** es su flexibilidad arquitectónica. Al estar diseñado como un microservicio autónomo (Middleware), puede instalarse e invocarse en distintos escenarios topológicos dependiendo de las necesidades de su red o de sus clientes.

A continuación, detallamos las tres modalidades principales en las que la API puede recibir las peticiones de facturación desde su ERP.

## 1. Despliegue en Localhost (Máquina única)
El escenario más común para TPVs autónomos, pequeños comercios o durante la fase de desarrollo e integración inicial.

* **Cómo funciona:** El Middleware y la Base de Datos se instalan en el mismo ordenador físico que ejecuta el software de facturación del cliente (ERP o TPV).
* **URL de Invocación Típica:** `http://127.0.0.1:8000/ingesta/` o `http://localhost:8000/ingesta/`
* **Ventajas:** Cero latencia de red. Si hay caídas en el router de la oficina, el ERP puede seguir delegando la facturación al Middleware; las facturas quedarán encoladas localmente y se autotransmitirán a Hacienda en background silenciosamente una vez regrese la conexión.
* **Seguridad:** Totalmente aislado del exterior, la API sólo escucha internamente a las peticiones del propio equipo.

## 2. Despliegue en Red Local / Intranet (Servidor LAN)
Ideal para pymes o restaurantes/supermercados con múltiples puestos de caja o terminales simultáneos, que apuntan a un único servidor central dentro de la misma estructura de la empresa.

* **Cómo funciona:** El Middleware se instala únicamente en la máquina principal ("Servidor Local"). Todos los terminales de cajero de la red disparan sus facturas JSON a la IP privada de dicho servidor.
* **URL de Invocación Típica:** `http://192.168.1.100:8000/ingesta/` *(Suponiendo que terminación .100 sea la IP asignada por el router a la máquina servidor)*.
* **Ventajas:** Una sola instalación y licencia de VeriFactu systemsFGH sirve a todos los empleados o puestos. Unifica las series, centraliza el monitor web en un sólo lugar y estandariza las peticiones usando el mismo almacén de certificados digitales sin que se deban repartir en cada terminal clíente.
* **Seguridad:** Protegido detrás de la infraestructura (firewall/router) de la empresa, evitando exposición directa a la web.

## 3. Despliegue en Servidor Remoto o Nube (VPS / Datacenter)
Pensado para proveedores de software de gestión contable en la nube (SaaS), facturación íntegramente web o macro-empresas con docenas de tiendas dispersas geográficamente.

* **Cómo funciona:** El Middleware se despliega en un entorno Cloud externo y virtualizado bajo su total control del motor operativo (AWS, Azure, o su propio servidor de Data Center).
* **URL de Invocación Típica:** `https://api.suservidorcloud.com/ingesta/` o `http://82.114.155.30:8000/ingesta/`
* **Ventajas:** Control centralizado y definitivo sobre todos sus clientes integradores. El comercio o cliente final en su sede únicamente necesita un PC básico y no aloja absolutamente nada técnico. Las métricas o cambios técnicos se consolidan todos directamente sobre este super-nodo maestro en la nube.
* **Seguridad:** En este modelo es aconsejable y necesario complementar el servidor con proxies reversos tradicionales que fortifiquen la conexión (ej. mapear el puerto 8000 mediante `Nginx` y asignarle tráfico cifrado `HTTPS` con certificados Let's Encrypt para proteger la transmisión entre sus sucursales y la nube, así como habilitar validación de Tokens para su API de Verifactu).
