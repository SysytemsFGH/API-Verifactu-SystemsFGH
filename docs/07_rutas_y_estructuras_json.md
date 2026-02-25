# 7. Rutas API y Estructuras JSON (El Flujo Asíncrono)

Para comprender las "tripas" del sistema de comunicación, este documento expone los canales JSON de **ida y de vuelta** exactos del Middleware. 

El modelo de Cero Fricción (`Zero Friction`) funciona inyectando las facturas por una vía rápida (`POST /ingesta`) y preguntando periódicamente por la vía paralela (`GET /pendientes`) hasta que el *Worker* de fondo devuelve los tickets con el OK de Hacienda.

> *Nota Técnica sobre las Rutas (Prefijos):* Notará que la Ingesta utiliza el prefijo `/v1/` mientras que las rutas de lectura asíncrona utilizan `/verifactu/`. Esto ocurre porque `/v1/ingesta` se programó en la arquitectura inicial de la API para soportar posibles versiones futuras del marco global de entrada, mientras que las respuestas y confirmaciones (`/verifactu/pendientes` y `/verifactu/ack`) se consolidaron bajo un enrutador semántico específico de la Agencia Tributaria. Utilice **literalmente** las rutas aquí descritas para evitar errores "HTTP 404 (Not Found)" en sus conexiones.

---

## 1. La Ingesta (El "Dispara y Olvida")

### `POST /v1/ingesta`
Ruta principal para enviar la factura desde su ERP.

*   **Request (Lo que usted envía):** El JSON masivo de Ingesta (detallado extensamente en el *Capítulo 6 - Diccionario de Datos*).
*   **Response OK (HTTP 200):** Si Pydantic da el visto bueno, el sistema abraza el JSON, lo baja a base de datos y le devuelve un localizador único o ID.

```json
{
  "status": "ok",
  "mensaje": "Grabado correctamente",
  "id": "TK2049",
  "id_formateado": "TK2049",
  "mask_raw": "",
  "tracking": {
    "emisor": "05616281A",
    "serie": "TK",
    "numfactura": "2049",
    "cab_num_secuencia": 18452,
    "det_linea": 4124,
    "timestamp": "2026-02-25T13:30:15.123456+01:00"
  },
  "operacion_realizada": "I"
}
```

*   **Response Error (HTTP 422):** Si falta un campo o la suma de las bases de IVA no cuadra matemáticamente, el motor escupe el rechazo instantáneo en formato JSON estandarizado, parando en seco la comunicación para no enviar basura a HACIENDA.

```json
{
  "status": "error",
  "codigo": "INGESTA_CALCULO_INCONSISTENTE",
  "mensaje": "Inconsistencia en totales",
  "detalles": [
    {
      "regla": "iva1 == base1 * piva1 / 100 (half_up|half_even)",
      "base1": "100.00",
      "piva1": "21.00",
      "iva1": "10.00",
      "calc_half_up": "21.00"
    }
  ]
}
```

---

## 2. El Polling (Preguntar por las pendientes)

### `GET /verifactu/pendientes`
Ruta que su ERP llamará a intervalos regulares (ej: cada 2 segundos) para preguntar: *"¿Qué hay de lo mío?"*

*   **Parámetros GET Obligatorios:** `?nif_emisor=B12345678`
*   **Parámetros GET Opcionales:** `&n_ultimos=50` (Limita la barrida de la cola).
*   **Response (HTTP 200):** Una matriz (Array JSON) con todas las facturas que el *Worker* ya ha firmado, enviado a Hacienda, y de las cuales ha obtenido un **resultado final** (sea Éxito o Fracaso). Si no hay novedades, la lista vendrá vacía `[]`.

**Ejemplo de Estructura de Vuelta (Matriz de Resultados):**
```javascript
[
  {
    "nif_emisor_factura": "05616281A", // NIF legal titular emisor de la factura comunicada
    "indice_log": 240, // Identificador interno auto-secuencial del registro en el middleware
    "id_envio": 15, // Identificador de lote/bloque (o Batch) asociado al envío múltiple
    "orden_envio": 1, // Índice o posición de esta factura concreta dentro de dicho lote
    "modo": 10, // Etiqueta técnica numérica de operación (10=A, 20=M, 30=B...)
    "serie_numero": "TK-02048", // Identificador oficial concatenado de Serie y Número
    "operacion": "A", // Flujo reportado: (A)Alta, (M)Modificación, (B)Baja
    "fecha_hora_envio": "2026-02-24T18:30:15", // Marca temporal exacta (Timestamp) de la acción del servidor
    "fecha_oficial_factura": "2026-02-24", // Fecha oficial de facturación consolidada por la lógica general
    "fecha_expedicion_factura": "2026-02-24", // Fecha bruta natural remitida por la pre-configuración o ERP
    "importe": 1500.50, // Importe total exacto acumulado declarado en esta factura
    "status": 0, // Estado del sistema: 0=Aceptada(OK), 3=Rechazo, etc.
    "csv": "Z3Z5...QQ==", // Código Seguro de Verificación asignado si se ha procesado con AEAT
    "url_qr_verifactu": "https://www1.agenciatributaria.gob.es/wlpl/TIES-tike/...", // Enlace público exacto inyectado en la imagen del QR impreso
    "qr_verifactu": "iVBORw0KGgoAAAANSUhEUgAA... (Base64 PNG)", // Imagen gráfica del QR codificada directamente en Base64

    // 👇 CAMPOS TÉCNICOS BLINDADOS (Se abren decodificando de Base64 a su tipo Binario original) 👇
    "json_original_det": "eyJpZCI...", // JSON crudo originario sin adulterar tal cual lo mandó el ERP
    "xml_log": "PFJlZ0ZhY3R1...", // El XML bruto fabricado según esquemas AEAT por este motor
    "xml_log_firmado": "PD94bWwgdmVyc..." // 🛡️ EL COMPROBANTE OFICIAL DEFINITIVO. FIRMADO Y SELLADO. 🛡️
  },
  // ... (Siguientes registros)
]
```

### Diccionario de Respuesta de `pendientes`
| Campo Devuelto | Tipo | Descripción |
| :--- | :---: | :--- |
| **`indice_log`** | Entero | ID único de este registro en la tabla de Pendientes. **Vital guardarlo** para el paso 3 (El Acuse de Recibo). |
| **`status`** | Entero | Estado numérico final devuelto. Clásicos son `0` (Aceptada / OK), `1` (Aceptada con Errores), o `3` (Rechazada). |
| **`csv`** | String | Código Seguro de Verificación. El comprobante legal devuelto por la AEAT de que usted presentó y rubricó la factura. |
| **`url_qr_verifactu`** | String | URL absoluta pre-construida para generar hipervínculos hacia el cotejo en la web de la AEAT. |
| **`qr_verifactu`** | String | Archivo binario de tipo imagen transparente PNG encapsulado en *Base64* (`iVBORw0K...`), listo para imprimir en ticket. |
| **`xml_log_firmado`** | String | Blob *Base64* protector de la firma digital. Puede usarlo para recuperar el archivo electrónico encriptado real como comprobante para el cliente. |
| **`codigo_error_verifactu`** | String | Si `status` es Rechazada, contendrá la numeración del fallo (ej. `3000`). |

---

## 3. El Acuse de Recibo (ACK)

### `POST /verifactu/ack`
Cuando su ERP recibe la lista JSON de `/pendientes`, lee la URL del QR, y se la graba *a fuego* en su propia base de datos; **su ERP debe confirmar (hacer ACK)** al Middleware que ya tiene la respuesta. Si su ERP no hace el ACK, el Middleware deducirá que su programa se colgó, la red se cayó, y se la volverá a reenviar en el siguiente bucle para evitar pérdida de recibos.

*   **Request (Lo que usted envía):** Un pequeño JSON indicándole a la API qué `indice_log` acaba de procesar satisfactoriamente.

```json
{
  "nif_emisor": "B12345678",
  "indice_log": 84192
}
```

*   **Response (HTTP 200):** El Middleware ocultará esa línea de la Cola de Pendientes para siempre y responderá con el cierre de la transacción:

```json
{
  "status": "ok",
  "message": "ACK received"
}
```
*(A partir de este momento, el ciclo de facturación se da por cerrado técnica e informáticamente).*
