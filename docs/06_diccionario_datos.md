# 6. Diccionario de Datos (Payload JSON)

Este documento detalla exhaustivamente todos los campos admitidos por el motor de validación (Pydantic) dentro del JSON de Ingesta. Conocer las longitudes, los valores permitidos y las exigencias lógicas de estos elementos es vital para garantizar el `200 OK` en la inyección de facturas hacia la AEAT y sortear los rechazos por `422 Unprocessable Entity`.

*Aclaración sobre Tipos de Datos:* En aras de la consistencia aritmética y para impedir colisiones por punto flotante entre arquitecturas remotas, **todos los valores (incluidos importes y porcentajes) deben inyectarse obligatoriamente encapsulados en cadenas de texto (`string`)**.

---

## 1. Bloque `cabecera`
Este objeto engloba todos los identificadores troncales, las lógicas de rectificación y los totales sumarizados del ticket o factura.

| Campo JSON | Exigencia | Descripción y Valores Admitidos |
| :--- | :---: | :--- |
| **`emisor`** | **Obligatorio** | NIF/CIF válido del emisor de la factura. Actúa como identificador clave en sistemas *Multi-Tenant* (RBAC) para cotejar permisos de inserción en el servidor. |
| **`numfactura`** | **Obligatorio** | El número de factura emitido por el ERP. (Min. 1 caracter). |
| **`serie`** | **Obligatorio** | La serie de la factura. Su longitud no puede exceder los **20 caracteres**. *(Nota: Evite espacios).* |
| **`fecha`** | **Obligatorio** | Fecha de expedición activa de la factura. Se propone el formato `DD/MM/YYYY` como el más seguro. La aplicación intentará descifrar otros formatos legibles, pero al estamparse en el XML hacia la AEAT, siempre se formateará como `DD-MM-YYYY`. |
| **`tipodoc`** | **Obligatorio** | Tipo de documento facturado según esquema AEAT. Frecuentes: `"01"` (Factura oficial o F1) y `"02"` (Ticket/Factura Simplificada o F2). |
| **`descripcionoperacion`** | Opcional | Texto narrativo del concepto global de la factura. Se recomienda evitar caracteres especiales o retornos de carro. |
| **`pais`** | **Obligatorio** | Código ISO de dos letras. Típicamente `"ES"` (España). |
| **`nif`** | Opcional | NIF / CIF de la contraparte comercial (Cliente/Receptor). |
| **`nombre`** | Opcional | Nombre, apellidos o Razón Social del receptor. |

### 1.1. Banderas de Rectificación, Sustitución y Anulación
Estas banderas habilitan las obligaciones legales que varían el esquema hacia las vías de alteración tributaria. 

| Campo JSON | Exigencia | Descripción y Valores Admitidos |
| :--- | :---: | :--- |
| **`eliminacion`** | Opcional | Actúa como **Baja**. Valores: `"S"`, `"N"` o vacío `""`. Si toma `"S"`, el sistema enviará a la AEAT la matriz de anulación de la factura (omitiendo importes). |
| **`rectificativa`** | Opcional / Def. `"N"`| Indica si la inyección es una factura Rectificativa de anteriores. Valores aceptados por el validador: `"S"`, `"N"`. |
| **`tipofacturarectificativa`** | Condicional | Exigido solo si el anterior es `"S"`. Acota si fue por *"Errores fundados"*, etc. Patrones: `"R1"`, `"R2"`, `"R3"`, `"R4"`, `"R5"`. |
| **`facturarectificada`** | Condicional | Número y Serie de la factura original que se desea reparar. (Obligatorio si `rectificativa = "S"`). |
| **`fecharectificada`** | Condicional | Fecha de expedición del documento original enmendado. Obligatorio si es rectificativa. Se propone el formato `DD/MM/YYYY` como el más seguro. |
| **`facturaf3`** | Opcional / Def. `""`| Factura de Sustitución a un Ticket previo. Valores aceptados: `"S"`, `"N"`, `""`. |
| **`numseriesustituidaf3`** | Condicional | Obligatorio si `facturaf3 = "S"`. |
| **`fechafactsustituidaf3`** | Condicional | Obligatorio si `facturaf3 = "S"`. Se propone el formato `DD/MM/YYYY` como el más seguro. |

### 1.2. Módulo Aritmético (Bases y Totales Generales)
Este bloque encapsula los importes. Las restricciones de redondeo (hacia arriba `HALF_UP` o el bancario par `HALF_EVEN`) se toleran en su nivel máximo de céntimos.

| Campo JSON | Exigencia | Descripción y Valores Permitidos |
| :--- | :---: | :--- |
| **`base`** | **Obligatorio** | Suma total de TODAS las Bases Imponibles *(Max 2 decimales, obligatoriamente separados por punto `.`)*. |
| **`totaliva`** | **Obligatorio** | Suma total de TODAS las Cuotas de IVA devengadas en la factura. *(Max 2 decimales, separados por punto `.`)*. |
| **`totalrecargo`**| Opcional | Si existiera recargo de equivalencia. *(Max 2 decimales, separados por punto `.`)*. |

### 1.3 Módulo Aritmético (Totales Desglosados en Cabecera)
Son 4 *Slots* posibles que exponen una radiografía matemática sobre cuál fue la composición final de la factura.  *(Nota: Deben concordar milimétricamente con los sumatorios generales declarados en el punto anterior, y con los vectores declarados más adelante en la tabla `Detalle`)*.

*(Deje vacíos `""` o con valor `"0.00"` aquellos Slots que no aplique, por ejemplo: `base2`, `iva2`)*

*   **`base1`**, **`base2`**, **`base3`**, **`base4`** `(String)`: Las Bases del bloque impositivo respectivo.
*   **`piva1`**, **`piva2`**, **`piva3`**, **`piva4`** `(String)`: Porcentaje del IVA. (Ej. `"21.00"` o `"10.0"` o `"4.00"`).
*   **`iva1`**, **`iva2`**, **`iva3`**, **`iva4`** `(String)`: Cuota del IVA calculado para esa base y el susodicho porcentaje. 
*   **`precargo1..4`**, **`recargo1..4`** `(String)`: Porcentaje y cuota del tributo de recargo de equivalencia respectivo en el Slot.

---

## 2. Bloque `detalle`
El bloque de Detalle funciona como una colección de vectores que desglosa las líneas y perfiles de base imponibles.
Es cardinal en la validación algorítmica y su regla vital es incorruptible: **Todos los parámetros dentro del detalle deben ser Array de exactamente 4 dimensiones (Listas de Strings), independientemente de que se encuentren vacíos**.

| Matriz (Array JSON) | Dimensiones | Ejemplo Legal de inyección (Pydantic valid) |
| :--- | :---: | :--- |
| **`base`** | `List[str]` tamaño 4 | `["100.00", "", "", ""]` |
| **`piva`** | `List[str]` tamaño 4 | `["21.00", "", "", ""]` |
| **`iva`** | `List[str]` tamaño 4 | `["21.00", "", "", ""]` |
| **`precargo`** | `List[str]` tamaño 4 | `["", "", "", ""]` ó `["5.20", "1.40", "", ""]` |
| **`recargo`** | `List[str]` tamaño 4 | `["", "", "", ""]` |

*Excepción Algorítmica:*  En el supuesto de que envíe una Baja explícita (bloque Cabecera `eliminacion = "S"`), el Middleware sortea las validaciones estrictas y acepta importes y listas a cero, ya que el único propósito de esa petición será tumbar el histórico del documento sin declarar finanzas a los servidores de Hacienda.
