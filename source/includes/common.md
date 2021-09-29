# Objetos comunes

## Clave de acceso

El sistema se encarga de generar automáticamente la clave de acceso de cada
comprobante y luego retornarla como parte de la respuesta de emisión del mismo.
Pero es posible también proveer la clave de acceso si se requiere tener control
en la generación de esta clave. La siguiente documentación explica como debe
estar estructurada la clave de acceso.

Las claves de acceso estarán compuestas de 49 caracteres numéricos, la herramienta o aplicativo a utilizar por el sujeto pasivo deberá generar de manera automática la clave de acceso, que constituirá un requisito que dará el carácter de único a cada uno de los comprobantes, y la misma servirá para que el SRI indique si el comprobante es autorizado o no; se describe a continuación su conformación:

Campo | Formato | Longitud
--------- | ---- |-----------
Fecha de Emisión| DDMMAAAA| 8
Tipo de Comprobante| Tabla 4| 2
Número de RUC| 1234567890001| 13
Tipo de Ambiente| Tabla 5| 1
Serie| 001001| 6
Número del Comprobante (secuencial)| 000000001| 9
Código Numérico| Numérico| 8
Tipo de Emisión| Tabla 2| 1
Dígito Verificador (módulo 11 )| Numérico| 1

**Nota:** Todos los campos deben completarse conforme a la longitud indicada, es decir si en el número secuencial no completa los 9 dígitos, la clave de acceso estará mal conformada y será motivo de rechazo de la autorización en línea.

El dígito verificador será aplicado sobre toda la clave de acceso (48 dígitos) y deberá ser incorporado por el contribuyente a través del método denominado Módulo 11, con un factor de chequeo ponderado (2), este mecanismo de detección de errores, será verificado al momento de la recepción del comprobante. Cuando el resultado del dígito verificador obtenido sea igual a once (11), el digito verificador será el cero (0) y cuando el resultado del dígito verificador obtenido sea igual a diez 10, el digito verificador será el uno (1).

El código numérico constituye un mecanismo para brindar seguridad al emisor en cada comprobante emitido, el algoritmo numérico para conformar este código es potestad absoluta del contribuyente emisor.

Ver [aquí](https://es.wikipedia.org/wiki/C%C3%B3digo_de_control) ejemplo de verificación utilizando algoritmo de módulo 11.


## Emisor

Información del emisor de un comprobante.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
ruc | string | Número de RUC de 13 caracteres
razon_social | string | Razón social. Máximo 300 caracteres
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres
direccion | string | Dirección registrada en el SRI. Máximo 300 caracteres.
contribuyente_especial | string | Número de resolución. En blanco si no es contribuyente especial.
obligado_contabilidad | boolean | `true` si está obligado a llevar contabilidad.
establecimiento | [establecimiento](#establecimiento) | Establecimiento que emite la factura.

## Establecimiento

Representa un establecimiento del comercio.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001`
direccion | string | Dirección registrada en el SRI. Máximo 300 caracteres
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`

## Máquina fiscal

Datos de una máquina fiscal.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
marca | string | Máximo 300 caracteres. __Requerido__
modelo | string | Máximo 300 caracteres. __Requerido__
serie | string | Máximo 300 caracteres. __Requerido__

## Reembolso

Datos de una factura o liquidación de compras de reembolso.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del tipo de documento de reembolso equivalente a 41. __Requerido__
documentos | listado de objeto [Documento](#documento) | Lista de documentos. __Requerido__
subtotal | float | Sumatoria de los subtotales de los documentos. __Requerido__
total_impuestos | float | Sumatoria de los totales de impuestos de los documentos. __Requerido__
total | float | Subtotal más total de impuestos. __Requerido__

## Documento
Datos de un documento.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo_establecimiento  | string |  3 caracteres. __Requerido__
codigo_punto_emision  | string |  3 caracteres. __Requerido__
secuencia  | integer (min. 1 - max. 999999999 ) | Número de secuencia del documento. __Requerido__
fecha_emision  | string |  Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
identificacion_proveedor | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion_proveedor | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
impuestos | listado de objeto [impuesto](#impuesto-item) | Impuestos totales del documento. __Requerido__
numero_autorizacion  | string | Número de autorización del documento. 10, 37 o 49 caracteres. __Requerido__
pais_origen_proveedor | string | Código  de dos caracteres del país origen según [ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements). __Requerido__
tipo  | string |  Código de [tipos de documentos](#tipos-de-documentos). __Requerido__
tipo_proveedor  | string | Código de [tipo de proveedor](#tipo-de-proveedor) de reembolso. __Requerido__


## Info adicional

Información adicional adjunta al documento. Es utilizada para especificar cualquier detalle
que no pueda ser descrito con los elementos que son parte del documento.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
nombre | string | Máximo 300 caracteres. __Requerido__
valor | string | Máximo 300 caracteres. __Requerido__

## Persona

Datos de una persona. Utilizado como __comprador__ en facturas y notas de crédito, como __sujeto__ en retenciones

Parámetro | Tipo | Descripción
--------- | ---- |-----------
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
email | string | Correo electrónico. Máximo 300 caracteres.
telefono | string | Teléfono.
direccion | string | Dirección


## Receptor

Datos del receptor de un documento.

Parámetro | Tipo | Descripción
----------|------|------------
identificacion | string | De 5 a 20 caracteres.
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación
telefono | string | Teléfono.
email | string | Correo electrónico. Máximo 300 caracteres.
direccion | string | Dirección


## Total Impuesto

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto)
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva).
base_imponible | float | Suma de las bases imponibles de cada item para el tipo de impuesto y porcentaje.
descuento_adicional | float | Descuento global aplicado en la factura, expresado en valor monetario. Solo aplica a impuestos de tipo IVA.
valor | float | Resultado de aplicar el impuesto a la (`base_imponible` - `descuento_adicional`)

## Impuesto Item

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto)
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva).
base_imponible | float (hasta 2 cifras decimales) | Corresponde al valor de la `cantidad` multiplicado por el `precio_unitario` menos el `descuento`
valor | float (hasta 2 cifras decimales) | Valor del total.
tarifa | float (hasta 2 cifras decimales) | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0


## Envío SRI

Contiene la información y el estado de la fase de envío al SRI

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- |-----------
mensajes | listado de objeto [mensaje SRI](#mensajes-de-respuesta-sri) | Listado de mensajes.
estado   | string | Posibles valores: `RECIBIDA`, `DEVUELTA`
fecha    | string | Fecha en la que se realizó el envío en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6).


## Autorización SRI

Contiene la información y el estado de la fase de autorización del comprobante.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- |-----------
mensajes | listado de objeto [mensaje SRI](#mensajes-de-respuesta-sri) | Listado de mensajes.
estado   | string | Posibles valores: `AUTORIZADO`, `NO AUTORIZADO`, `PRE-AUTORIZADO`
numero   | string | Número de autorización.
fecha    | string | Fecha en la que se otorgó la autorización en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6).

## Mensajes de respuesta SRI

Parámetro             | Tipo   | Descripción
--------------------- | ------ | -----------
identificador         | string | Número con el que el SRI identifica al mensaje.
mensaje               | string | Descripción del error, información o advertencia.
tipo                  | string | Posibles valores: `INFORMATIVO`, `ADVERTENCIA`, `ERROR`
informacion_adicional | string | Información adicional del mensaje.

<h2 id="item-de-factura">Item de factura, nota de crédito y liquidación de compras</h2>

Representa un producto o servicio del comercio.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
descripcion | string (máximo 300 caracteres) | Descripción del ítem. __Requerido__
codigo_principal | string (máximo 25 caracteres) | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
codigo_auxiliar | string (máximo 25 caracteres) | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
cantidad | float (hasta 6 cifras decimales) | Cantidad de items. __Requerido__
precio_unitario | float (hasta 6 cifras decimales) | Precio unitario. __Requerido__
descuento | float (hasta 2 cifras decimales) | El descuento es aplicado por cada producto, expresado en valor monetario. __Requerido__
precio_total_sin_impuestos | float (hasta 2 cifras decimales) | Precio antes de los impuestos. Se obtiene multiplicando la `cantidad` por el `precio_unitario`
unidad_medida | string | Unidad de medida __Requerido para facturas de exportación__
impuestos | listado de objetos tipo [impuesto item](#impuesto-item) | Impuestos grabados sobre el producto. __Requerido__
detalles_adicionales | object | Diccionario de datos de carácter adicional. Ejemplo:<br><code>{"marca": "Ferrari", "chasis": "UANEI832-NAU101"}</code>

<h2 id="retencion-de-factura">Retención en factura </h2>

Caso específico de Retenciones en la Comercializadores / Distribuidores de derivados del Petróleo y Retención presuntiva de IVA a los Editores, Distribuidores y Voceadores que participan en la comercialización de periódicos y/o revistas.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del [tipo de impuesto para la retención en la factura](#tipos-de-impuesto-para-la-retencion-en-la-factura).  __Requerido__
codigo_porcentaje | string | Código del [porcentaje del impuesto](#retencion-de-iva-presuntivo-y-renta). __Requerido__
tarifa | float (hasta 2 cifras decimales) | Porcentaje actual del impuesto.  __Requerido__
valor | float (hasta 2 cifras decimales) | Valor del impuesto.  __Requerido__

# Catálogo

## Tipo de identificación

### Operación

`GET /catalog/id-types`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/id-types\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
      {
      "count": 6,
      "results": [
        {
          "nombre": "CEDULA",
          "codigo": "05"
        },
        {
          "nombre": "RUC",
          "codigo": "04"
        },
        {
          "nombre": "PASAPORTE",
          "codigo": "06"
        },
        {
          "nombre": "PLACA",
          "codigo": "09"
        },
        {
          "nombre": "IDENTIFICACION DEL EXTERIOR",
          "codigo": "08"
        },
        {
          "nombre": "CONSUMIDOR FINAL",
          "codigo": "07"
        }
      ]
    }
```

Consulta todos los tipos de identificaciones registrados en Datil que sean validos
por el Servicios de Rentas Internas.

Tipo de identificación      | Código
--------------------------- | ------
RUC                         | `04`
CEDULA                      | `05`
PASAPORTE                   | `06`
VENTA A CONSUMIDOR FINAL*   | `07`
IDENTIFICACION DELEXTERIOR* | `08`
PLACA                       | `09`


## Tipos de impuesto

### Operación

`GET /catalog/taxes/sales`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/taxes/sales\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
  {
  "count": 3,
  "results": [
    {
      "impuesto": "ICE",
      "codigo": "3"
    },
    {
      "impuesto": "IRBPNR",
      "codigo": "5"
    },
    {
      "impuesto": "IVA",
      "codigo": "2"
    }
  ]
}
```



Consulta todos los impuestos validos en la emision de un comprobante de ventas
acorde a los estándares que demanda el Servicio de Rentas Internas.

Impuesto | Código
-------- | ------
IVA      | 2
ICE      | 3
IRBPNR   | 5

## Código de Porcentaje de IVA

### Operación

`GET /catalog/tax-rates?code=2`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/tax-rates?code=2\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
      {
      "count": 4,
      "results": [
        {
          "nombre": "IVA 0%",
          "tarifa": "0.00",
          "codigo_porcentaje": "0"
        },
        {
          "nombre": "IVA 12%",
          "tarifa": "12.00",
          "codigo_porcentaje": "2"
        },
        {
          "nombre": "IVA NO OBJETO DE IMPUESTO",
          "tarifa": "0.00",
          "codigo_porcentaje": "6"
        },
        {
          "nombre": "IVA EXENTO DE IVA",
          "tarifa": "0.00",
          "codigo_porcentaje": "7"
        }
      ]
    }
```

Consulta los porcentajes y tarifas del IVA acorde al Servicios de Rentas Internas.


Porcentaje de IVA | Código | Tarifa
-------- | ------ | ------
0%     | 0 |  0
12%      | 2 | 12
No Objeto de Impuesto   | 6 | -
Exento de IVA   | 7 | -

## Tipos de impuesto para la retención

### Operación

`GET /catalog/taxes/withholdings`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/taxes/withholdings\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
    {
    "count": 4,
    "results": [
        {
          "nombre": "RENTA",
          "codigo": "1"
        },
        {
          "nombre": "ISD",
          "codigo": "6"
        },
        {
          "nombre": "IVA PRESUNTIVO Y RENTA",
          "codigo": "4"
        },
        {
          "nombre": "IVA",
          "codigo": "2"
        }
    ]
}
```

Consulta todos los impuestos a retener acorde a la ficha técnica del Servicio
de Rentas Internas.  


Impuesto | Código
-------- | ------
RENTA    | 1
IVA      | 2
IVA PRESUNTIVO Y RENTA | 4
ISD      | 6

## Tipos de impuesto para la retención en la factura

Caso específico de Retenciones en la Comercializadores / Distribuidores de derivados del
Petróleo y Retención presuntiva de IVA a los Editores, Distribuidores y Voceadores que
participan en la comercialización de periódicos y/o revistas.

Impuesto                  | Código
------------------------- | ------
IVA PRESUNTIVO Y RENTA    | 4



## Retención de IVA

### Operación

`GET /catalog/withholdings-tax-rates?code=2`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/withholdings-tax-rates?code=2\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
    {
    "count": 7,
    "results": [
        {
          "nombre": "RETENCION DE IVA 30%",
          "codigo": "1",
          "porcentaje": "30.00"
        },
        {
          "nombre": "RETENCION DE IVA 70%",
          "codigo": "2",
          "porcentaje": "70.00"
        },
        {
          "nombre": "RETENCION DE IVA 100%",
          "codigo": "3",
          "porcentaje": "100.00"
        },
        {
          "nombre": "RETENCION EN 0%",
          "codigo": "7",
          "porcentaje": "0.00"
        },
        {
          "nombre": "RETENCION DE IVA 10%",
          "codigo": "9",
          "porcentaje": "10.00"
        },
        {
          "nombre": "RETENCION DE IVA 20%",
          "codigo": "10",
          "porcentaje": "20.00"
        },
        {
          "nombre": "NO PROCEDE RETENCION",
          "codigo": "8",
          "porcentaje": "0.00"
        }
      ]
    }
```

Consulta los porcentajes de retencion del IVA. El codigo enviado en la ruta de
la peticion corresponde al codigo del impuesto.

Porcentaje IVA | Código
-------------- | ------
10%            | 9
20%            | 10
30%            | 1
70%            | 2
100%           | 3

__Retención en cero__

Porcentaje IVA | Código
-------------- | ------
0%             | 7

__No procede retención__

Porcentaje IVA | Código
-------------- | ------
0%             | 8

## Retención ISD

### Operación

`GET /catalog/withholdings-tax-rates?code=6`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/withholdings-tax-rates?code=6\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json

  {
  "count": 1,
  "results": [
      {
        "nombre": "ISD",
        "codigo": "4580",
        "porcentaje": "5.00"
      }
    ]
  }
```

Consulta los porcentajes de retencion del ISD. El codigo enviado en la ruta de
la peticion corresponde al codigo del impuesto.

Porcentaje IVA | Código
-------------- | ------
5%             | 4580


## Retención a la Fuente (Renta)

### Operación

`GET /catalog/withholdings-tax-rates?code=1`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/withholdings-tax-rates?code=1\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta
Actualmente tenemos registrado 323 retenciones por impuesto a la Renta registrados
acorde al SRI, por efecto de este ejemplo solo se muestra 8 de ellos.

```json

{
  "count": 8,
  "results": [
      {
        "nombre": "HONORARIOS PROFESIONALES Y DEMAS PAGOS RELACIONADOS CON EL T\u00cdTULO PROFESIONAL",
        "codigo": "303",
        "porcentaje": "10.00"
      },
      {
        "nombre": "SERVICIOS PREDOMINA EL INTELECTO NO RELACIONADOS CON EL TITULO PROFESIONAL",
        "codigo": "304",
        "porcentaje": "8.00"
      },
      {
        "nombre": "SERVICIOS PREDOMINA LA MANO DE OBRA",
        "codigo": "307",
        "porcentaje": "2.00"
      },
      {
        "nombre": "SERVICIOS PRESTADOS POR MEDIOS DE COMUNICACION Y AGENCIAS DE PUBLICIDAD",
        "codigo": "309",
        "porcentaje": "1.00"
      },
      {
        "nombre": "SERVICIO TRANSPORTE PRIVADO DE PASAJEROS O SERVICIO PUBLICO O PRIVADO DE CARGA",
        "codigo": "310",
        "porcentaje": "1.00"
      },
      {
        "nombre": "TRANSFERENCIA DE BIENES MUEBLES DE NATURALEZA CORPORAL",
        "codigo": "312",
        "porcentaje": "1.00"
      },
      {
        "nombre": "CUOTAS POR ARRENDAMIENTO MERCANTIL, INCLUSIVE LA DE OPCION DE COMPRA ",
        "codigo": "319",
        "porcentaje": "1.00"
      },
      {
        "nombre": "ARRENDAMIENTO BIENES INMUEBLES",
        "codigo": "320",
        "porcentaje": "8.00"
      }
    ]
  }
```

Consulta los porcentajes de retencion del impuesto a la RENTA. El codigo enviado en la ruta de
la peticion corresponde al codigo del impuesto.

O tambien puedes realizar la consulta en el catálogo vigente de códigos en:

[https://www.sri.gob.ec/web/guest/retenciones-en-la-fuente](https://www.sri.gob.ec/web/guest/retenciones-en-la-fuente)

<aside class="notice">
Utiliza los códigos indicados en la columna <em>"Código del Anexo"</em>
</aside>

## Retención de IVA Presuntivo y Renta

Caso específico de Retenciones en la Comercializadores / Distribuidores de derivados del
Petróleo y Retención presuntiva de IVA a los Editores, Distribuidores y Voceadores que
participan en la comercialización de periódicos y/o revistas.

__Retención IVA__

Porcentaje IVA presuntivo                               | Código
--------------------------------------------------------| ------
100%                                                    | 3
12%  (Editores a Margen de Comercialización Voceadores) | 4
100% (Venta periódicos y/o Revistas a Distribuidores)   | 5
100% (Venta periódicos y/o Revistas a Voceadores)       | 6

__Retención Renta__

Porcentaje Renta | Código
---------------- | ------
0.2%             | 327
0.3%             | 328


## Tipos de documentos

### Operación

`GET /catalog/document-types`

> #### Ejemplo del requerimiento

```shell
  curl -v https://link.datil.co/catalog/document-types\
  -H "Content-Type: application/json" \
  -H "X-Key: <clave-del-api>"
```

> #### Ejemplo de la respuesta

```json
    {
      "count": 17,
      "results": [
        {
          "nombre": "FACTURA",
          "codigo": "01"
        },
        {
          "nombre": "NOTA DEBITO",
          "codigo": "05"
        },
        {
          "nombre": "LIQ. COMPRAS",
          "codigo": "03"
        },
        {
          "nombre": "COMPROBANTE DE PAGO APORTES",
          "codigo": "19"
        },
        {
          "nombre": "NOTA DE VENTA",
          "codigo": "02"
        },
        {
          "nombre": "NOTA DE CREDITO",
          "codigo": "04"
        },
        {
          "nombre": "RETENCION POR COMPRAS",
          "codigo": "07"
        },
        {
          "nombre": "BOLETOS O ENTRADAS A ESPECTACULOS PUBLICOS",
          "codigo": "08"
        },
        {
          "nombre": "TIQUETES O VALES EMITIDOS POR MAQ. REGISTR.",
          "codigo": "09"
        },
        {
          "nombre": "PASAJES EMITIDOS POR EMPRESAS DE AVIACION",
          "codigo": "11"
        },
        {
          "nombre": "DOCUMENTOS EMITIDOS POR INSTITUCIONES FINANCIERAS",
          "codigo": "12"
        },
        {
          "nombre": "COMPROBANTE DE VENTA EMITIDO EN EL EXTERIOR",
          "codigo": "15"
        },
        {
          "nombre": "FORMULARIO UNICO DE IMPORTACION",
          "codigo": "17"
        },
        {
          "nombre": "DOC. SERV. ADM. EMITIDOS POR INSTIT. DEL ESTADO",
          "codigo": "20"
        },
        {
          "nombre": "CARTA DE PORTE AEREO",
          "codigo": "21"
        },
        {
          "nombre": "RECAP",
          "codigo": "22"
        },
        {
          "nombre": "LIQUIDACION PARA EXPLOT. Y EXPLOR. HIDROCARBUROS",
          "codigo": "43"
        }
      ]
    }
```
Consulta los tipos de documentos registrados en Datil y validados
por el Servicios de Rentas Internas

Documento                | Código
------------------------ | ------
Factura                  | 01
Liquidación de Compra    | 03
Nota de Crédito          | 04
Nota de Débito           | 05
Guía de Remisión         | 06
Comprobante de Retención | 07

## Tipos de forma de pago

Forma de pago            | Código
------------------------ | ------
Efectivo | efectivo
Cheque | cheque
Débito de cuenta bancaria | debito_cuenta_bancaria
Transferencia bancaria | transferencia
Depósito en cuenta (Corriente / Ahorros) | deposito_cuenta_bancaria
Tarjeta de débito | tarjeta_debito
Dinero electrónico Ecuador | dinero_electronico_ec
Tarjeta prepago | tarjeta_prepago
Tarjeta de crédito | tarjeta_credito
Otros | otros
Endoso de títulos | endoso_titulos

## Tipos de forma de pago del SRI

Forma de pago            | Código
------------------------ | ------
Sin utilización del sistema financiero | 01
Compensación de deudas | 15
Tarjeta de débito | 16
Dinero electrónico | 17
Tarjeta prepago | 18
Tarjeta de crédito | 19
Otros con utilización del sistema financiero | 20
Endoso de títulos | 21

## Equivalencia entre formas de pago Dátil y formas de pago del SRI

Forma de pago Dátil      | Código | Forma de pago SRI      | Código
------------------------ | ------ | -----------------------|-------
Efectivo | efectivo | Sin utilización del sistema financiero | 01
Cheque | cheque | Otros con utilización del sistema financiero | 20
Débito bancario | debito_cuenta_bancaria | Otros con utilización del sistema financiero| 20
Transferencia bancaria | transferencia | Otros con utilización del sistema financiero | 20
Tarjeta de crédito | tarjeta_credito | Tarjeta de crédito nacional | 19
Depósito en cuenta (Corriente / Ahorros) | deposito_cuenta_bancaria | Otros con utilización del sistema financiero | 20
Tarjeta de débito | tarjeta_debito | Tarjeta de débito | 16
Dinero electrónico Ecuador | dinero_electronico_ec | Dinero electrónico | 17
Tarjeta prepago | tarjeta_prepago | Tarjeta prepago | 18
Otros | otros | Otros con utilización del sistema financiero | 20
Endoso de títulos | endoso_titulos | Endoso de títulos | 21

Debido a que el Servicio de Rentas Internas exige incluir información del pago,
las facturas a crédito se enviarán al SRI con forma de pago "Otros con utilización del sistema financiero".

## Tipo de proveedor

Tipo | Código
---- |-----------
Personal natural |  01
Sociedad |  02
