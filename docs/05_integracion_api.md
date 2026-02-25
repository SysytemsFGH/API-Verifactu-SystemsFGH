# 5. Guía de Integración API Rest

Una vez comprendido el simulador y la lógica asíncrona, este capítulo aborda exclusivamente cómo conectar su ERP o TPV programáticamente mediante nuestra API.  El objetivo es realizar llamadas `HTTP POST` con el JSON de la factura para que el Middleware inteligente trabaje de manera autónoma.

## 1. El Endpoint Principal

Todo el ecosistema de facturación converge en un único y simple endpoint de ingesta:

*   **URL:** `http://[IP_SERVIDOR]:8000/ingesta/`
*   **Método HTTP:** `POST`
*   **Content-Type:** `application/json`

*(Nota: Sustituya `[IP_SERVIDOR]` por `127.0.0.1` en caso de despliegue local o la IP pertinente de su LAN o VPC en la nube).*

## 2. El Esquema (Payload) JSON

La API se alimenta de un objeto JSON estructurado en tres grandes bloques jerárquicos: `metadata`, `cabecera` y `detalle`.

A continuación se muestra un ejemplo íntegro de **Alta de Factura** enviado mediante API:

```json
{
  "cabecera": {
    "emisor": "12345678Z",
    "numfactura": "2049",
    "serie": "TK",
    "rectificativa": "N",
    "tipodoc": "01",
    "pais": "ES",
    "nif": "B12345678",
    "nombre": "EMPRESA DE PRUEBAS VERIFACTU S.L.",
    "fecha": "20/02/2026",
    "totaliva": "104.31",
    "base": "667.97",
    "base1": "340.99",
    "piva1": "21.00",
    "iva1": "71.61",
    "base2": "326.98",
    "piva2": "10.00",
    "iva2": "32.70"
  },
  "detalle": {
    "base": ["340.99", "326.98", "", ""],
    "piva": ["21.00", "10.00", "", ""],
    "iva": ["71.61", "32.70", "", ""],
    "precargo": ["", "", "", ""],
    "recargo": ["", "", "", ""]
  }
}
```

### 2.1. Bloque "cabecera"
Este bloque hospeda los axiomas de la factura: El `emisor` (nuestro multi-tenant, clave primordial), serie, enumeración de totales y el desglose de los 4 tramos posibles de IVA y sus bases respectivas (`base1..base4`). También es el lugar donde recaen los marcadores del Tipo de documento (`tipodoc`), fechas y contrapartes (`nif` cliente, `pais`, `nombre`). Si la factura es de rectificación o sustitución, aquí se declararán los NIF y Fechas rectificadas (`rectificativa`: "S").

### 2.2. Bloque "detalle" (El array de 4 slots de impuestos)
Cierra la estructura. El Middleware Inteligente espera 4 posiciones fijas para asentar el sumatorio de impuestos indirectos, bases imponibles y potenciales recargos de equivalencia (`precargo`, `recargo`). Si su ERP emite facturas con un solo tipo de tributación (ej. IVA general), usted informará únicamente el índice 0 de los arreglos y dejará los índices del 1 al 3 como cadenas de caracteres vacías `""`.

*   **Regla de Oro de Pydantic:** Todas las listas (`base`, `piva`, `iva`...) deben ostentar **exactamente** tamaño 4 (con valores o strings vacíos `""`). 

## 3. Ejemplos de Subida desde Código

Aunque la API es 100% estándar REST, disponemos de **Adaptadores Nativos (SDKs)** en lenguajes como C#, Delphi, PHP, Node.js y Python (disponibles en sus respectivas guías de integración estáticas). El mayor beneficio de estos motores es que **permiten saltarse la gestión explícita de la capa de transporte HTTP** y la gestión manual del ciclo de esperas, dejando que el desarrollador trabaje únicamente enviando y recibiendo objetos nativos.

### 3.1. Usando llamada HTTP Pura (Python `requests`)
Si opta un entorno sin SDK, la integración manual (Raw HTTP) es sencilla y requiere gestionar el endpoint por su cuenta:

```python
import requests
import json

payload = {
    # su diccionario JSON ...
}

try:
    response = requests.post(
        "http://127.0.0.1:8000/ingesta/",
        json=payload,
        timeout=10 # Buena práctica para ingestas masivas
    )
    if response.status_code == 200:
        print("Encolado correctamente. ID de seguimiento", response.json()["id"])
    elif response.status_code == 422:
        print("Error de validación sintáctica:", response.json())
except requests.exceptions.RequestException as e:
    print("Fallo de red hacia el Middleware:", e)
```

### 3.2. Usando los Adaptadores SDK (Saltando la capa de transporte)
Si usted usa nuestros adaptadores (por ejemplo la clase `VFEngine` disponible para **Delphi** o **C# .NET**), el código del ERP se sintetiza dramáticamente. La librería orquesta por debajo la Ingesta, enmascara el transporte TCP, gestiona la espera (Polling) contra el `Worker` y lanza la confirmación final de recepción.

**Ejemplo en Delphi (Función Síncrona "Todo en uno"):**
```pascal
procedure TForm1.EmitirFacturaTodoEnUno;
var
  Engine: TVFEngine;
  Res: TVFIngestaAckResult;
begin
  Engine := TVFEngine.Create(MiConfiguracion);
  try
    // La función IngestaYConfirmacion oculta toda la capa de transporte HTTP y URL
    Res := Engine.IngestaYConfirmacion(MiJsonText);
    
    if Res.AckHecho then
      // Si todo fue bien, obtenemos los datos QR directos
      ImprimirTicket(Res.Pendiente.UrlQrVerifactu)
    else
      // O capturamos el rechazo de forma unificada
      ShowMessage('Error AEAT: ' + Res.ErrorMsg);
  finally
    Engine.Free;
  end;
end;
```

**Ejemplo paralelo en C# (.NET):**
```csharp
public async Task EmitirAltaSincrona(string nif, string serie, string num)
{
    // MiFacturaJson ya está mapeado. Se delega toda la inyección de red.
    var resIngesta = await _engine.IngestaJsonAsync(miFacturaJson);
    if (!resIngesta["ok"].GetValue<bool>()) return;

    // El adaptador se encarga de preguntar asíncronamente a los endpoints secundarios
    // hasta que el Worker conteste con el QR, sin que usted gestione llamadas HTTP.
    var statusState = await _engine.CheckStatusAsync(nif, serie, num);

    if (statusState?["status"]?.ToString() == "OK")
    {
        await _engine.AckIndiceAsync(nif, (int)statusState["indice_log"]);
        Console.WriteLine("QR Recuperado correctamente y confirmado al Servidor");
    }
}
```

## 4. Respuestas HTTP del Middleware
Las respuestas son inmediatas porque la factoría de Hacienda opera en las sobras (asíncrona). 

*   **`200 OK`**: La Ingesta es correcta matemática y jerárquicamente y descansa en la Cola Interna. Retorna su localizador y estatus.
*   **`422 Unprocessable Entity`**: Hay descarrilamiento. O bien falta un campo exigido inexcusablemente o los axiomas no convergen (ej. El sumatorio de `base1` + `base2` no da matemáticamente igual al valor global `base` consignado en la cabecera truncado a 2 decimales). La respuesta `422` especificará exactamente qué sumatorio o validación Pydantic reventó.
