# Configuración avanzada

__Tablas o Vistas__

Para que la aplicación **Link** se conecte a su ERP o sistema contable se deben configurar
los *queries* para extraer información directamente de las tablas  de su base de datos.

Si no desea que **Link** consulte directamente sus tablas puede crear vistas con la información
de los comprobantes a emitir.

__Formato__

Los queries se guardan siguiendo el formato de archivos `.ini`. en el directorio
*C:/Archivos de Programa/Datil/Link/config/*.

En cada archivo `.ini` cada _query_ está asignado a una variable.

Si aún no ha instalado __Link__, puede crear un archivo `.ini` de respaldo con los queries configurados.

[Ver ejemplos de archivos de configuración](#ejemplos-de-archivos-de-configuracion)

__Nombres de tablas y columnas__

Sea directamente o por medio de vistas, dependiendo de su sistema, deberá cambiar los nombres de las tablas  al configurar los _queries_.

Los nombres de las columnas en las tablas
de su sistema pueden ser distintos pero el resultado de cada _query_ debe tener los nombres establecidos en esta documentación.

__Queries opcionales__

Algunos queries son opcionales porque la información que extraen no es obligatoria para el SRI, en este caso la variable que se configura en el archivo `.ini` debe tener asignada el valor de `None`.

Ejemplo:

`invoice_additional_information = None`

`item_details = None`


<h2 id="facturas-advanced">Facturas</h2>

Los queries para la emisión electrónica de __facturas__ se guardan en el archivo de configuración `invoice.ini`.

[Ejemplo de archivo invoice.ini](/link-app#invoice-ini)

### Cabecera

Obtiene información de la cabecera de la factura.

```sql
headers = SELECT
  id_factura            id_local,
  secuencial,
  fecha_emision,
  guia_remision,
  moneda,
  clave_acceso,
  tipo_emision
  FROM
  DocElectronicoFactura.cabecera
  WHERE
  info.id_factura in (:sequence)
  ORDER BY id_factura :order
```


Campo |  Tipo | Descripción
--------- | -----------| ---------
id_local | int o string | Identifica de manera única la factura. __Requerido__
secuencial | string  | Número de secuencia de la factura. __Requerido__
fecha_emision | datetime  | Fecha de emisión   __Requerido__
guia_remision | string | Número de guía de remisión asociada a esta factura en formato 001-002-000000003 ([0-9]{3}-[0-9]{3}-[0-9]{9})
moneda | string | Código [ISO](https://en.wikipedia.org/wiki/ISO_4217) de la moneda. __Requerido__
clave_acceso | string | La clave de acceso representa un identificador único del comprobante. Si esta información no es provista, Dátil la generará.<br>¿Cómo [generar](#clave-de-acceso) la clave de acceso?
tipo_emision | integer | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__

### Emisor

Obtiene información del vendedor en la factura

```sql
invoice_seller  = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_emisor,
  codigo,
  punto_emision
  FROM
  DocElectronicoFactura.cabecera
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_emisor | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__


### Comprador

Obtiene información del comprador en la factura

```sql
invoice_buyer  = SELECT
  identificacion,
  tipo_identificacion,
  razon_social,
  direccion,
  email,
  telefono
  FROM
  FROM
  DocElectronicoFactura.cabecera
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
email | string | Correo electrónico. Máximo 300 caracteres. __Requerido__
telefono | string | Teléfono
direccion | string | Dirección


### Totales

Obtiene información de los valores totales de la facturas

```sql
invoice_totals  = SELECT
  total_sin_impuestos,
  importe_total,
  propina,
  descuento
  FROM
  DocElectronicoFactura.cabecera
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
total_sin_impuestos | float | Total antes de los impuestos. __Requerido__
importe_total       | float | Total incluyendo impuestos. __Requerido__
propina             | float | Propina total, llamado también servicio. __Requerido__
descuento           | float | Suma de los descuentos de cada ítem y del descuento adicional. __Requerido__

### Impuestos de totales

Obtiene información de los impuestos de los totales de la factura

```sql
invoice_totals_taxes  = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  valor
  FROM
  DocElectronicoFactura.totales_impuestos
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__


### Items

Obtiene todos los items de una factura

```sql
items  = SELECT
  id_detalle,
  codigo_principal,
  codigo_auxiliar,
  descripcion,
  cantidad,
  precio_unitario,
  descuento,
  precio_total_sin_impuestos,
  unidad_medida
  FROM
  DocElectronicoFactura.items
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
id_detalle | string | Identifica de manera única el ítem o detalle de la factura. Si no hay un solo campo que lo identifique de manera única se debe usar la concatenación de varios.__Requerido__
codigo_principal | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres. __Requerido__
codigo_auxiliar | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
descripcion | string | Descripción del ítem. __Requerido__
cantidad | float | Cantidad de items. __Requerido__
precio_unitario | float | Precio unitario. __Requerido__
descuento | float | El descuento es aplicado por cada producto. __Requerido__
precio_total_sin_impuestos | float | Precio antes de los impuestos. Se obtiene multiplicando la `cantidad` por el `precio_unitario` __Requerido__
unidad_medida | string | Unidad de medida. Ejemplo: Kilos. __Requerido para facturas de exportación__

### Impuestos de items

Obtiene los impuestos de un item. Este query es __opcional__

```sql
item_taxes  = SELECT
  base_imponible,
  valor,
  tarifa,
  codigo,
  codigo_porcentaje
  FROM
  DocElectronicoFactura.items_impuestos
  WHERE
  id_detalle = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__

### Detalles adicionales de items

Obtiene los detalles adicionales de un ítem. Este query es __opcional__.

Los detalles adicionales de un ítem se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Peso':'Kg'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con detalles adicionales del ítem:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Peso        |   KG
Color           |   Rojo
Caducidad                 |   10 días

```sql
item_details = SELECT
  columna_de_nombres    nombre,
  columna_de_valores   valor
  FROM
  DocElectronicoFactura.items_detalles_adicionales
  WHERE
  id_detalle = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
nombre | string | Nombre del detalle adicional del ítem
valor | string | Valor del detalle adicional del ítem

### Información adicional

Obtiene la información adicional de la factura. Este query es __opcional__

La información adicional de la factura se maneja de la forma 'Clave':'Valor'. Ejemplo: 'Tipo de pago':'Cheque'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con información adicional de la factura:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Tipo de servicio        |   Avanzado
Forma de pago           |   Cheque
Periodo                 |   3 meses

```sql
invoice_additional_information = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores     _valor_
  FROM
  DocElectronicoFactura.informacion_adicional
  WHERE
  id_factura = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la información adicional de la factura
`_valor_` | string | Valor de la información adicional de la factura

### Pagos

Obtiene la información de los pagos aplicables a la factura.

```sql
payment_methods  = SELECT
  id_pago,
  medio_pago medio,
  total_pago total
  FROM
  DocElectronicoFactura.pagos
  WHERE
  id_factura = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
id_pago              | --                  | Identificador único del pago, se usa para obtener las [propiedades
del pago](Propiedades de Pagos), si no hay un identificador único del pago o no hay propiedades de pagos, se debe devolver el *id_factura*.
medio              | string                  | Código del [tipo de forma de pago](#tipos-de-forma-de-pago). __Requerido__
total               | float                   | Total aplicable a la forma de pago especificada. __Requerido__

### Propiedades de Pagos

Obtiene la propiedades de los pagos de determinada factura. Este query es __opcional__

Las propiedades de los pagos se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Plazo':'5'

Se asume que en la tabla consultada una columna tiene los nombres y otra los valores.

Ejemplo de columnas con propiedades de los pagos:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
plazo        |   9
unidad_tiempo           |   dias

```sql
payment_method_properties = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores     _valor_
  FROM
  DocElectronicoFactura.propiedades_pagos
  WHERE
  id_pago = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la propiedad del pago
`_valor_` | string | Valor de la propiedad del pago


Para el Servicio de Rentas Internas de Ecuador (SRI), las únicas propiedades que se tomarán en cuenta son `plazo` (especifica el plazo del tipo de pago) y `unidad_tiempo` (especifica la unidad de tiempo en la cual se expresa el plazo).

Las demás propiedades que se especifiquen se registrarán en Dátil como parte del pago, pero no se reportarán al SRI.

### Crédito

Crédito otorgado en la venta.  Este query es __opcional__ , en caso de que todos los pagos se realizan en el momento de la venta, es decir no hay crédito.

Campo           | Tipo    | Descripción
------------------- | ------- | ----------
fecha_vencimiento   | string  | Fecha de vencimiento en formato AAAA-MM-DD, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
monto               | float   | Monto otorgado de crédito. __Requerido__

```sql
invoice_credit = SELECT
  monto,
  fecha_vencimiento
  FROM
  facturas.credito
  WHERE
  id_factura = ?
```

### Compensación solidaria

Descuento otorgado a las provincias de Manabí y Esmeraldas. Obligatorio solo para estas provincias.


Campo           | Tipo    | Descripción
------------------- | ------- | ----------
codigo   | int  | Código del porcentaje de IVA . __Requerido__
tarifa   | int   | Porcentaje de compensación. __Requerido__
valor   | float   | Valor de la compensación. __Requerido__

```sql
invoice_compensation = SELECT
  codigo,
  tarifa,
  valor
  FROM
  facturas.compensacion
  WHERE
  id_factura = ?
```

### Exportación

Obligatorio __solo__ para facturas de exportación


Campo           | Tipo    | Descripción
------------------- | ------- | ----------
incoterm_termino   | string  | Código de 3 letras correspondiente al [Incoterm](https://en.wikipedia.org/wiki/Incoterms#Rules_for_any_mode_of_transport) . __Requerido__
incoterm_lugar   | string  | Lugar Incoterm . __Requerido__
incoterm_total_sin_impuestos   | string  | Total sin impuestos pagado por el incoterm. __Requerido__
codigo_pais_origen   | string  | Código de dos letras del país origen según [ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)  . __Requerido__
codigo_pais_destino   | string  | Código de dos letras del país origen según [ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)
codigo_pais_adquisicion   | string  | Código de dos letras del país origen según[ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)
puerto_origen   | string  | Puerto de origen. __Requerido__
puerto_destino   | string  | Puerto de destino. __Requerido__
total_flete_internacional   | float  | Total del flete internacional
total_seguro_internacional   | float  | Total del seguro internacional
total_gastos_aduaneros   | float  | Total de los gastos aduaneros
total_otros_gastos_transporte   | float  | Total de otros gastos de transporte

```sql
invoice_export = SELECT
  incoterm_termino,
  incoterm_lugar,
  incoterm_total_sin_impuestos,
  codigo_pais_origen,
  codigo_pais_destino,
  codigo_pais_adquisicion,
  puerto_origen,
  puerto_destino,
  total_flete_internacional,
  total_seguro_internacional,
  total_gastos_aduaneros,
  total_otros_gastos_transporte,
  FROM
  facturas.exportacion
  WHERE
  id_factura = ?
```

### Tablas recomendadas

Estructura recomendada para las tablas o vistas con información de la factura.

Ejemplo en SQL Server:

```sql
CREATE SCHEMA facturas

DROP TABLE [facturas].[pago_propiedad]
DROP TABLE [facturas].[pago]
DROP TABLE [facturas].[item_impuesto]
DROP TABLE [facturas].[item_detalle_adicional]
DROP TABLE [facturas].[item]
DROP TABLE [facturas].[total_impuesto]
DROP TABLE [facturas].[informacion_adicional]
DROP TABLE [facturas].[factura]


CREATE TABLE [facturas].[factura](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [ambiente] [int] NOT NULL, -- (OPCIONAL) el ambiente se toma del archivo de configuración de Link App.
    [tipo_emision] [int] NOT NULL,
    [secuencial] [bigint] NOT NULL,
    [fecha_emision] [datetime] NULL,
    [moneda] [varchar](15) NOT NULL, -- USD para dólares
    [guia_remision] [varchar](17),
    [clave_acceso] [varchar](49) NULL,
    -- EMISOR
    [ruc] [varchar](13) NULL,
    [obligado_contabilidad] [varchar](2) NULL,
    [contribuyente_especial] [varchar](10) NULL,
    [nombre_comercial] [varchar](300) NULL,
    [razon_social] [varchar](300) NULL,
    [direccion_matriz] [varchar](300) NULL,
    [codigo_establecimiento] [varchar](3) NULL,
    [punto_emision] [varchar](3) NULL,
    [direccion_establecimiento] [varchar](300) NULL,
    -- COMPRADOR
    [email_comprador] [varchar](254) NULL,
    [identificacion_comprador] [varchar](20) NULL,
    [tipo_identificacion_comprador] [varchar](2) NULL,
    [razon_social_comprador] [varchar](200) NULL,
    [direccion_comprador] [varchar](200) NULL,
    [telefono_comprador] [varchar](200) NULL,
    -- TOTALES
    [total_sin_impuestos] [decimal](14,2) NULL,
    [importe_total] [decimal](14,2) NULL,
    [propina] [decimal](14,2) NULL,
    [descuento] [decimal](14,2) NULL,
    [descuento_adicional] [decimal](14,2) NULL,
    -- CREDITO
    [monto_credito] [decimal](14,2) NULL,
    [fecha_vencimiento_credito] [date] NULL,
    -- VALORES RETENIDOS
    [valor_retenido_iva] [decimal](14,2) NULL,
    [valor_retenido_renta] [decimal](14,2) NULL,
)

-- FACTURA: ITEMS
CREATE TABLE [facturas].[item](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
    [cantidad] [decimal](14,2)  NOT NULL,
    [codigo_principal] [varchar](50)  NULL,
    [codigo_auxiliar] [varchar](50)  NULL,
    [precio_unitario] [decimal](14,2)  NOT NULL,
    [descripcion] [varchar](300)  NOT NULL,
    [precio_total_sin_impuestos] [decimal](14,2)  NOT NULL,
    [descuento] [decimal](14,2)  NULL,
    [unidad_medida] [varchar](50)  NULL

)

-- FACTURA: IMPUESTOS DE ITEMS
CREATE TABLE [facturas].[item_impuesto](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [facturas].[item](id),
    [codigo] [varchar](2) NOT NULL,
    [codigo_porcentaje] [varchar](2) NOT NULL,
    [base_imponible] [decimal](14,2) NOT NULL,
    [valor] [decimal](14,2) NOT NULL,
    [tarifa] [decimal](14,2) NOT NULL, -- porcentaje
    CONSTRAINT PK_item_impuesto PRIMARY KEY (id_item, codigo, codigo_porcentaje)
)

-- FACTURA: DETALLES ADICIONALES DE ITEMS
CREATE TABLE [facturas].[item_detalle_adicional](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [facturas].[item](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT pk_items_detalles_adicionales PRIMARY KEY (id_item, nombre)
)

-- FACTURA: IMPUESTOS TOTALES
CREATE TABLE [facturas].[total_impuesto](
    [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
    [codigo] [varchar](2) NOT NULL,
    [codigo_porcentaje] [varchar](2) NOT NULL,
    [base_imponible] [decimal](14,2) NOT NULL,
    [valor] [decimal](14,2) NOT NULL
    CONSTRAINT PK_total_impuesto PRIMARY KEY (id_factura, codigo, codigo_porcentaje)
)

-- FACTURA: INFORMACION ADICIONAL DE LA FACTURA
CREATE TABLE [facturas].[informacion_adicional](
    [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT PK_informacion_adicional PRIMARY KEY (id_factura, nombre)
)

-- FACTURA: PAGOS
CREATE TABLE [facturas].[pago](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
    [fecha] [datetime] NOT NULL,
    [medio] [varchar](100) NOT NULL,
    [notas] [varchar](max) NOT NULL,
    [monto] [decimal](14, 2) NOT NULL
)

-- FACTURA: PROPIEDADES DE PAGOS
CREATE TABLE [facturas].[pago_propiedad](
  [id_pago] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[pago](id),
  [nombre] [varchar](100) NOT NULL,
  [valor] [varchar](100) NOT NULL,
  CONSTRAINT PK_pago_propiedad PRIMARY KEY (id_pago, nombre)
)

-- FACTURA: CREDITO
CREATE TABLE [facturas].[credito](
  [id] bigint IDENTITY(1,1) PRIMARY KEY,
  [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
  [monto] [decimal](14,2) NOT NULL,
  [fecha_vencimiento] [varchar](10) NOT NULL
)

-- FACTURA: COMPENSACION SOLIDARIA
CREATE TABLE [facturas].[compensacion](
  [id] bigint IDENTITY(1,1) PRIMARY KEY,
  [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
  [codigo] [int] NOT NULL,
  [tarifa] [int] NOT NULL,
  [valor] [decimal](14,2) NOT NULL
)

-- FACTURA: EXPORTACION
CREATE TABLE [facturas].[exportacion](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_factura] bigint NOT NULL FOREIGN KEY REFERENCES [facturas].[factura](id),
    [incoterm_termino] [varchar](10)  NULL,
    [incoterm_lugar] [varchar](300)  NULL,
    [incoterm_total_sin_impuestos] [varchar](10)  NOT NULL,
    [codigo_pais_origen] [varchar](2)  NULL,
    [codigo_pais_destino] [varchar](2)  NULL,
    [codigo_pais_adquisicion] [varchar](2)  NULL,
    [puerto_origen] [varchar](300)  NULL,
    [puerto_destino] [varchar](300)  NULL,
    [total_flete_internacional] [decimal](14,2)  NOT NULL,
    [total_seguro_internacional] [decimal](14,2)  NOT NULL,
    [total_gastos_aduaneros] [decimal](14,2)  NOT NULL,
    [total_otros_gastos_transporte] [decimal](14,2)  NOT NULL
)
```

<h2 id="retenciones-ats-advanced">Comprobantes de Retención ATS</h2>

Los queries para la emisión electrónica de __comprobantes de retención ats__ se guardan en el archivo de configuración
`ats_retention.ini`.

[Ejemplo de archivo ats_retention.ini](/link-app#credit_note-ini)

### Retención ATS

Obtiene información de la información principal de la retención ats.

```sql
headers = SELECT
  id id_local,
  tipo_emision,
  secuencial,
  fecha_emision,
  clave_acceso,
  periodo_fiscal
  FROM
  retenciones_ats.retencion_ats
  WHERE retenciones_ats.retencion_ats.id in (:sequence)
  ORDER BY fecha_emision :order

```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_local | integer | Identifica de manera única la retención ats. __Requerido__
secuencial | string  | Número de secuencia de la retención ats. __Requerido__
fecha_emision | datetime  | Fecha de emisión   __Requerido__
clave_acceso | string | La clave de acceso representa un identificador único del comprobante. Si esta información no es provista, Dátil la generará.<br>¿Cómo [generar](#clave-de-acceso) la clave de acceso?
tipo_emision | integer | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__
periodo_fiscal | string | Mes y año en el siguiente formato MM/AAAA. Ejm: 12/2015 __Requerido__

### Emisor

Obtiene la información del vendedor de la retención ats

```sql
issuer = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_matriz,
  codigo_establecimiento,
  punto_emision
  FROM
  retenciones_ats.retencion_ats
  WHERE
  id = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_matriz | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo_establecimiento | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__

### Sujeto

Obtiene información del sujeto retenido en la retención

```sql
retention_recipient  = SELECT
  identificacion_sujeto,
  tipo_identificacion,
  razon_social,
  direccion_sujeto,
  email_sujeto,
  telefono_sujeto
  FROM
  retenciones_ats.retencion_ats
  WHERE
  id = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
identificacion_sujeto | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
direccion_sujeto | string | Dirección
email_sujeto | string | Correo electrónico. Máximo 300 caracteres.
telefono_sujeto | string | Teléfono

### Información Adicional de la Retención ATS

Información adicional adjunta al documento. Es utilizada para especificar cualquier detalle
que no pueda ser descrito con los elementos que son parte del documento.

```sql
additional_information = SELECT
  nombre,
  valor
  FROM
  retenciones_ats.info_adicional
  WHERE
  id_retencion_ats = ?
```

Parámetro | Tipo | Descripción
--------- | ---- |-----------
nombre | string | Máximo 300 caracteres. __Requerido__
valor | string | Máximo 300 caracteres. __Requerido__

### Documentos Soporte

Obtiene toda la información de los documentos de soporte de una retención ats

```sql
support_documents = SELECT
  codigo_sustento,
  tipo_documento,
  numero,
  fecha_emision,
  numero_autorizacion,
  total_sin_impuestos,
  total
  FROM
  retenciones_ats.documentos_soporte
  WHERE
  id_retencion_ats = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
codigo_sustento | string | Ver (tabla)(#tipos-de-sustento-de-comprobantes) de tipos de sustento de los comprobantes. Máximo 2 caracteres. __Requerido__
tipo_documento | string | Ver códigos de [tipos de documentos](#tipos-de-documentos). Máximo 2 caracteres. __Requerido__
numero | string | Número completo del documento asociado a la retención ATS. Máximo 17 caracteres. __Requerido__
fecha_emision | string |  Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
numero_autorización | string | Número de autorización del comprobante de venta. Máximo 300 caracteres. __Requerido__
total_sin_impuestos | string | Total antes de los impuestos. __Requerido__
total | string | Total incluyendo impuestos. __Requerido__

### Impuestos de Documentos de Soporte

Obtiene la información de los impuestos de un documento de soporte. Este query es __opcional__

```sql
taxes = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  valor,
  tarifa
  FROM
  retenciones_ats.impuestos_documentos_soporte
  WHERE
  id_documento_soporte = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__

### Retenciones de Documentos de Soporte

Obtiene la información de los impuestos retenidos de un documento de soporte.

```sql
taxes_support_documents = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  tarifa,
  valor_retenido
  FROM
  retenciones_ats.retenciones_documentos_soporte
  WHERE
  id_documento_soporte = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
base_imponible | float | Suma de las bases imponibles de cada item para el tipo de impuesto y porcentaje. __Requerido__
valor_retenido | float (hasta 2 cifras decimales) | Valor del impuesto.  __Requerido__
tarifa | float (hasta 2 cifras decimales) | Porcentaje actual del impuesto.  __Requerido__
codigo | string | Código del [tipo de impuesto para la retención en la factura](#tipos-de-impuesto-para-la-retencion-en-la-factura).  __Requerido__
codigo_porcentaje | string | Código del [porcentaje del impuesto](#retencion-de-iva-presuntivo-y-renta). __Requerido__

### Dividendos de Documentos de Soporte

Obtiene la información de los dividendos asociados a un documento de soporte.

```sql
dividends = SELECT
  impuesto_renta,
  fecha_pago,
  annio_fiscal
  FROM
  retenciones_ats.dividendos_retenciones
  WHERE
  id_documento_soporte = ?
```

Campo | Tipo | Descripción
--------- | ---- |-----------
fecha_pago | string | Fecha de pago en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
impuesto_renta | string | Impuesto a la Renta pagado por la sociedad correspondiente al dividendo __Requerido__
annio_fiscal | integer | Año en que se generaron las utilidades atribuibles al dividendo. __Requerido__

### Reembolso de Documentos de Soportes

Información del reembolso de un documento de soporte

```sql
reimbursement = SELECT
  codigo,
  subtotal,
  total,
  total_impuestos
  FROM
  retenciones_ats.documentos_soporte
  WHERE
  id_retencion_ats = ?
```

Campo | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del tipo de documento de reembolso equivalente a 41. __Requerido__
subtotal | float | Sumatoria de los subtotales de los documentos. __Requerido__
total_impuestos | float | Sumatoria de los totales de impuestos de los documentos. __Requerido__
total | float | Subtotal más total de impuestos. __Requerido__


### Documentos de Reembolso

Información de los documentos de reembolso

```sql
reimbursement_documents = SELECT
  codigo_establecimiento,
  codigo_punto_emision,
  secuencia,
  fecha_emision,
  identificacion_proveedor,
  tipo_identificacion_proveedor,
  numero_autorizacion,
  pais_origen_proveedor,
  tipo,
  tipo_proveedor
  FROM
  retenciones_ats.reembolsos_documentos_soporte
  WHERE
  id_reembolso = ?
```

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo_establecimiento  | string |  Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: 001. __Requerido__
codigo_punto_emision  | string |  Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: 001.. __Requerido__
secuencia  | integer (min. 1 - max. 999999999 ) | Número de secuencia del documento. __Requerido__
fecha_emision  | string |  Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
identificacion_proveedor | string | Identificación del proveedor. De 5 a 20 caracteres. __Requerido__
tipo_identificacion_proveedor | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
numero_autorizacion  | string | Número de autorización del documento. 10, 37 o 49 caracteres. __Requerido__
pais_origen_proveedor | string | Código  de dos caracteres del país origen según [ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements). __Requerido__
tipo  | string |  Código de [tipos de documentos](#tipos-de-documentos). __Requerido__
tipo_proveedor  | string | Código de [tipo de proveedor](#tipo-de-proveedor) de reembolso. __Requerido__


### Impuestos de Reembolsos de los Documentos de Soporte

Información de los impuestos de los reembolsos de un documento de soporte.

```sql
reimbursement_taxes = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  valor,
  tarifa
  FROM
  retenciones_ats.impuestos_reembolsos
  WHERE
  id_reembolso_documentos = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__

### Pagos de Documentos de Soporte

Información de los pagos asociados a un documento de soporte.

```sql
support_documents_payments = SELECT
  tipo_pago,
  total
  FROM
  retenciones_ats.pagos_documentos_soporte
  WHERE
  id_documento_soporte = ?
```

<h2 id="notas-credito-advanced">Notas de crédito</h2>

Los queries para la emisión electrónica de __notas de crédito__ se guardan en el archivo de configuración `credit_note.ini`.

[Ejemplo de archivo credit_note.ini](/link-app#credit_note-ini)

### Cabecera

Obtiene información de la cabecera de la nota de crédito

```sql
headers = SELECT
  id_nota_credito             id_local,
  secuencial,
  fecha_emision,
  moneda,
  clave_acceso,
  tipo_emision,
  fecha_emision_documento_modificado,
  numero_documento_modificado,
  tipo_documento_modificado,
  motivo
  FROM
  DocElectronicoNotaCredito.cabecera
  WHERE
  id_nota_credito in (:sequence)
  ORDER BY id_nota_credito :order
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_local | int o string | Identifica de manera única la nota de crédito. __Requerido__
secuencial | string  | Número de secuencia de la nota de crédito. __Requerido__
fecha_emision | datetime  | Fecha de emisión   __Requerido__
moneda | string | Código [ISO](https://en.wikipedia.org/wiki/ISO_4217) de la moneda. __Requerido__
clave_acceso | string | La clave de acceso representa un identificador único del comprobante. Si esta información no es provista, Dátil la generará.<br>¿Cómo [generar](#clave-de-acceso) la clave de acceso?
tipo_emision | integer | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__
fecha_emision_documento_modificado | datetime | Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria del documento modificado, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6).  __Requerido__
numero_documento_modificado | string | Número completo del documento que se está afectando. Normalmente facturas. Ejm: 001-002-010023098 __Requerido__
tipo_documento_modificado | string | Códigos de [tipos de documentos](#tipos-de-documentos). __Requerido__
motivo | string | Motivo de la operación. Ejm: Devolución de producto. __Requerido__


### Vendedor

Obtiene información del vendedor en la nota de crédito

```sql
credit_note_seller  = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_emisor,
  codigo,
  punto_emision
  FROM
  DocElectronicoNotaCredito.cabecera
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_emisor | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__


### Comprador

Obtiene información del comprador en la nota de crédito

```sql
credit_note_buyer  = SELECT
  identificacion,
  tipo_identificacion,
  razon_social,
  direccion,
  email,
  telefono
  FROM
  DocElectronicoNotaCredito.cabecera
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
email | string | Correo electrónico. Máximo 300 caracteres.
telefono | string | Teléfono
direccion | string | Dirección


### Totales

Obtiene información de los valores totales de la nota de crédito

```sql
credit_note_totals  = SELECT
  total_sin_impuestos,
  importe_total
  FROM
  DocElectronicoNotaCredito.cabecera
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
total_sin_impuestos | float | Total antes de los impuestos. __Requerido__
importe_total       | float | Total incluyendo impuestos. __Requerido__

### Impuestos de totales

Obtiene información de los impuestos de los totales de la nota de crédito

```sql
credit_note_totals_taxes  = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  valor
  FROM
  DocElectronicoNotaCredito.totales_impuestos
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__


### Items

Obtiene todos los items de una nota de crédito

```sql
items  = SELECT
  id_detalle,
  codigo_principal,
  codigo_auxiliar,
  descripcion,
  cantidad,
  precio_unitario,
  descuento,
  precio_total_sin_impuestos
  FROM
  DocElectronicoNotaCredito.items
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
id_detalle | string | Identifica de manera única el ítem o detalle de la nota de crédito. Si no hay un solo campo que lo identifique de manera única se debe usar la concatenación de varios.__Requerido__
codigo_principal | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres. __Requerido__
codigo_auxiliar | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
descripcion | string | Descripción del ítem. __Requerido__
cantidad | float | Cantidad de items. __Requerido__
precio_unitario | float | Precio unitario. __Requerido__
descuento | float | El descuento es aplicado por cada producto. __Requerido__
precio_total_sin_impuestos | float | Precio antes de los impuestos. Se obtiene multiplicando la `cantidad` por el `precio_unitario` __Requerido__

### Impuestos de items

Obtiene los impuestos de un item

```sql
item_taxes  = SELECT
  base_imponible,
  valor,
  tarifa,
  codigo,
  codigo_porcentaje
  FROM
  DocElectronicoNotaCredito.items_impuestos
  WHERE
  id_detalle = ?
```


Campo | Tipo | Descripción
--------- | ------- | -----------
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__

### Detalles adicionales de items

Obtiene los detalles adicionales de un ítem. Este query es opcional.

Los detalles adicionales de un ítem se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Peso':'Kg'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con detalles adicionales del ítem:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Peso        |   KG
Color           |   Rojo
Caducidad                 |   10 días

```sql
item_details = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores   _valor_
  FROM
  DocElectronicoNotaCredito.items_detalles_adicionales
  WHERE
  id_detalle = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
nombre | string | Nombre del detalle adicional del ítem
valor | string | Valor del detalle adicional del ítem

### Información adicional

Obtiene la información adicional de la nota de crédito.

La información adicional de la nota de crédito se maneja de la forma 'Clave':'Valor'. Ejemplo: 'Tipo de pago':'Cheque'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con información adicional de la nota de crédito:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Tipo de servicio        |   Avanzado
Forma de pago           |   Cheque
Periodo                 |   3 meses

```sql
credit_note_additional_information = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores     _valor_
  FROM
  DocElectronicoNotaCredito.informacion_adicional
  WHERE
  id_nota_credito = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la información adicional de la nota de crédito
`_valor_` | string | Valor de la información adicional de la nota de crédito


### Detalles adicionales de items

Obtiene los detalles adicionales de un ítem. Este query es opcional.

Los detalles adicionales de un ítem se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Peso':'Kg'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con detalles adicionales del ítem:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Peso        |   KG
Color           |   Rojo
Caducidad                 |   10 días

```sql
item_details = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores   _valor_
  FROM
  DocElectronicoNotaCredito.items_detalles_adicionales
  WHERE
  id_detalle = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
nombre | string | Nombre del detalle adicional del ítem
valor | string | Valor del detalle adicional del ítem


### Tablas recomendadas

Estructura recomendada para las tablas o vistas con información de la nota de crédito.

Ejemplo en SQL Server:

```sql

CREATE SCHEMA notas_de_credito

DROP TABLE [notas_de_credito].[item_impuesto]
DROP TABLE [notas_de_credito].[item_detalle_adicional]
DROP TABLE [notas_de_credito].[item]
DROP TABLE [notas_de_credito].[total_impuesto]
DROP TABLE [notas_de_credito].[informacion_adicional]
DROP TABLE [notas_de_credito].[nota_de_credito]


CREATE TABLE [notas_de_credito].[nota_de_credito](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [ambiente] [int] NOT NULL,
    [tipo_emision] [int] NOT NULL,
    [secuencial] [bigint] NOT NULL,
    [fecha_emision] [datetime] NULL,
    [moneda] [varchar](15) NOT NULL,
    [clave_acceso] [varchar](49),
    -- DOCUMENTO MODIFICADO
    [fecha_emision_documento_modificado] [datetime] NULL,
    [numero_documento_sustento] [varchar](17) NULL,
    [tipo_documento_modificado] [varchar](2) NULL,
    [motivo] [varchar](300) NULL,
    -- EMISOR
    [ruc] [varchar](13) NULL,
    [obligado_contabilidad] [varchar](2) NULL,
    [contribuyente_especial] [varchar](10) NULL,
    [nombre_comercial] [varchar](300) NULL,
    [razon_social] [varchar](300) NULL,
    [direccion_matriz] [varchar](300) NULL,
    [codigo_establecimiento] [varchar](3) NULL,
    [punto_emision] [varchar](3) NULL,
    [direccion_establecimiento] [varchar](300) NULL,
    -- COMPRADOR
    [email_comprador] [varchar](254) NULL,
    [identificacion_comprador] [varchar](20) NULL,
    [tipo_identificacion_comprador] [varchar](2) NULL,
    [razon_social_comprador] [varchar](200) NULL,
    [direccion_comprador] [varchar](200) NULL,
    [telefono_comprador] [varchar](200) NULL,
    -- TOTALES
    [total_sin_impuestos] [decimal](14,2) NULL,
    [importe_total] [decimal](14,2) NULL,
)

-- NOTA DE CRÉDITO: ITEMS
CREATE TABLE [notas_de_credito].[item](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_nota_credito] bigint NOT NULL FOREIGN KEY REFERENCES [notas_de_credito].[nota_de_credito](id),
    [cantidad] [decimal](14,2)  NOT NULL,
    [codigo_principal] [varchar](50)  NULL,
    [codigo_auxiliar] [varchar](50)  NULL,
    [precio_unitario] [decimal](14,2)  NOT NULL,
    [descripcion] [varchar](300)  NOT NULL,
    [precio_total_sin_impuestos] [decimal](14,2)  NOT NULL,
    [descuento] [decimal](14,2)  NULL
)

-- NOTA DE CRÉDITO: IMPUESTOS DE ITEMS
CREATE TABLE [notas_de_credito].[item_impuesto](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [notas_de_credito].[item](id),
    [codigo] [varchar](2) NOT NULL,
    [codigo_porcentaje] [varchar](2) NOT NULL,
    [base_imponible] [decimal](14,2) NOT NULL,
    [valor] [decimal](14,2) NOT NULL,
    [tarifa] [decimal](14,2) NOT NULL, -- porcentaje
    CONSTRAINT PK_item_impuesto PRIMARY KEY (id_item, codigo, codigo_porcentaje)
)

-- NOTA DE CRÉDITO: DETALLES ADICIONALES DE ITEMS
CREATE TABLE [notas_de_credito].[item_detalle_adicional](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [notas_de_credito].[item](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT pk_items_detalles_adicionales PRIMARY KEY (id_item, nombre)
)

-- NOTA DE CRÉDITO: IMPUESTOS TOTALES
CREATE TABLE [notas_de_credito].[total_impuesto](
    [id_nota_credito] bigint NOT NULL FOREIGN KEY REFERENCES [notas_de_credito].[nota_de_credito](id),
    [codigo] [varchar](2) NOT NULL,
    [codigo_porcentaje] [varchar](2) NOT NULL,
    [base_imponible] [decimal](14,2) NOT NULL,
    [valor] [decimal](14,2) NOT NULL
    CONSTRAINT PK_total_impuesto PRIMARY KEY (id_nota_credito, codigo, codigo_porcentaje)
)

-- NOTA DE CRÉDITO: INFORMACION ADICIONAL DE LA NOTA DE CRÉDITO
CREATE TABLE [notas_de_credito].[informacion_adicional](
    [id_nota_credito] bigint NOT NULL FOREIGN KEY REFERENCES [notas_de_credito].[nota_de_credito](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT PK_informacion_adicional PRIMARY KEY (id_nota_credito, nombre)
)

```


<h2 id="retencion-advanced">Comprobantes de Retención</h2>

Los queries para la emisión electrónica de __retención__ se guardan en el archivo de configuración `retention.ini`.

[Ejemplo de archivo retention.ini](/link-app#retention-ini)

### Cabecera

Obtiene información de la cabecera de la retención

```sql
headers = SELECT
  id_nota_credito             id_local,
  secuencial,
  fecha_emision,
  clave_acceso,
  tipo_emision,
  periodo_fiscal
  FROM
  DocElectronicoRetencion.cabecera
  WHERE
  id_retencion in (:sequence)
  ORDER BY id_retencion :order
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_local | int o string | Identifica de manera única la retención. __Requerido__
secuencial | string  | Número de secuencia de la retención. __Requerido__
fecha_emision | datetime  | Fecha de emisión   __Requerido__
clave_acceso | string | La clave de acceso representa un identificador único del comprobante. Si esta información no es provista, Dátil la generará.<br>¿Cómo [generar](#clave-de-acceso) la clave de acceso?
tipo_emision | integer | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__
periodo_fiscal | string | Mes y año en el siguiente formato MM/AAAA. Ejm: 12/2015 __Requerido__


### Emisor

Obtiene información del vendedor en la retención

```sql
retention_seller  = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_emisor,
  codigo,
  punto_emision
  FROM
  DocElectronicoRetencion.cabecera
  WHERE
  id_retencion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_emisor | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__


### Sujeto retenido

Obtiene información del sujeto retenido en la retención

```sql
retention_recipient  = SELECT
  identificacion,
  tipo_identificacion,
  razon_social,
  direccion,
  email,
  telefono
  FROM
  DocElectronicoRetencion.cabecera
  WHERE
  id_retencion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
direccion | string | Dirección
email | string | Correo electrónico. Máximo 300 caracteres.
telefono | string | Teléfono


### Impuestos de la retención

Obtiene información de los impuestos de la retención

```sql
retention_taxes  = SELECT
    codigo,
    codigo_porcentaje,
    porcentaje,
    base_imponible,
    valor_retenido,
    tipo_documento_sustento,
    numero_documento_sustento,
    fecha_emision_documento_sustento
    FROM
    DocElectronicoRetencion.impuesto
    WHERE
    id_retencion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
codigo                           | string | Código de [tipo de impuesto](#tipos-de-impuesto-para-la-retencion). __Requerido__
codigo_porcentaje                | string | [Código del porcentaje](#retencion-de-iva) a aplicar dentro del tipo de impuesto __Requerido__
porcentaje | string | Porcentaje a retener __Requerido__
base_imponible | float | Base imponible. __Requerido__
valor_retenido | float | Valor retenido. __Requerido__
tipo_documento_sustento | string | Códigos de [tipos de documentos](#tipos-de-documentos). __Requerido__
numero_documento_sustento | string | Número completo del documento que se está afectando. Normalmente facturas. Ejm: 001-002-010023098 __Requerido__
fecha_emision_documento_sustento | datetime | Fecha de emisión del documento sustento de la retención__Requerido__

### Información adicional

Obtiene la información adicional de la retención.

La información adicional de la retención se maneja de la forma 'Clave':'Valor'. Ejemplo: 'Tipo de pago':'Cheque'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con información adicional de la retención:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Tipo de servicio        |   Avanzado
Forma de pago           |   Cheque
Periodo                 |   3 meses

```sql
retention_additional_information = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores     _valor_
  FROM
  DocElectronicoRetencion.informacion_adicional
  WHERE
  id_retencion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la información adicional de la rentención
`_valor_` | string | Valor de la información adicional de la retención

### Tablas recomendadas

Estructura recomendada para las tablas o vistas con información de la retención.

Ejemplo en SQL Server:


```sql

CREATE SCHEMA retenciones

DROP TABLE [retenciones].[item]
DROP TABLE [retenciones].[informacion_adicional]
DROP TABLE [retenciones].[retencion]


CREATE TABLE [retenciones].[retencion](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [ambiente] [int] NOT NULL,
    [tipo_emision] [int] NOT NULL,
    [secuencial] [bigint] NOT NULL,
    [fecha_emision] [datetime] NULL,
    [clave_acceso] [varchar](49) NULL,
    [periodo_fiscal] [varchar](7),
    -- EMISOR
    [ruc] [varchar](13) NULL,
    [obligado_contabilidad] [varchar](2) NULL,
    [contribuyente_especial] [varchar](10) NULL,
    [nombre_comercial] [varchar](300) NULL,
    [razon_social] [varchar](300) NULL,
    [direccion_matriz] [varchar](300) NULL,
    [codigo_establecimiento] [varchar](3) NULL,
    [punto_emision] [varchar](3) NULL,
    [direccion_establecimiento] [varchar](300) NULL,
    -- SUJETO RETENIDO
    [email_sujeto] [varchar](254) NULL,
    [identificacion_sujeto] [varchar](20) NULL,
    [tipo_identificacion_sujeto] [varchar](2) NULL,
    [razon_social_sujeto] [varchar](200) NULL,
    [direccion_sujeto] [varchar](200) NULL,
    [telefono_sujeto] [varchar](200) NULL,
)

-- RETENCION: ITEMS
CREATE TABLE [retenciones].[item](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_retencion] bigint NOT NULL FOREIGN KEY REFERENCES [retenciones].[retencion](id),
    [codigo] [varchar](2) NOT NULL,
    [codigo_porcentaje] [varchar](5) NOT NULL,
    [base_imponible] [decimal](14,2) NOT NULL,
    [fecha_emision_documento_sustento] [datetime] NULL,
    [numero_documento_sustento] [varchar](17) NULL,
    [tipo_documento_sustento] [varchar](2) NULL,
    [porcentaje] [decimal](14,2) NULL,
    [valor_retenido] [decimal](14,2) NULL,
)

-- RETENCION: INFORMACION ADICIONAL DE LA RETENCION
CREATE TABLE [retenciones].[informacion_adicional](
    [id_retencion] bigint NOT NULL FOREIGN KEY REFERENCES [retenciones].[retencion](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT PK_informacion_adicional PRIMARY KEY (id_retencion, nombre)
)

```

<h2 id="guias-remision-advanced">Guías de Remisión</h2>

Los queries para la emisión electrónica de __guías de remisión__ se guardan en el archivo de configuración `waybill.ini`.

[Ejemplo de archivo waybill.ini](/link-app#waybill-ini)

### Cabecera

Obtiene información de la cabecera de la guía de remisión

```sql
headers = SELECT
  id_guia_remision             id_local,
  secuencial,
  fecha_inicio_transporte,
  fecha_fin_transporte,
  direccion_partida,
  clave_acceso,
  tipo_emision
  FROM
  DocElectronicoGuiaRemision.cabecera
  WHERE
  id_guia_remision in (:sequence)
  ORDER BY id_guia_remision :order
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_local | int o string | Identifica de manera única la guía de remisión. __Requerido__
secuencial | string  | Número de secuencia de la retención. __Requerido__
fecha_inicio_transporte | datetime  | Fecha en la que inicia el transporte dada la guía de remisión __Requerido__
fecha_fin_transporte | datetime  | Fecha en la que termina el transporte dada la guía de remisión __Requerido__
direccion_partida | string | Dirección de partida
clave_acceso | string | La clave de acceso representa un identificador único del comprobante. Si esta información no es provista, Dátil la generará.<br>¿Cómo [generar](#clave-de-acceso) la clave de acceso?
tipo_emision | integer | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__

### Vendedor

Obtiene información del vendedor en la guía de remisión

```sql
waybill_seller  = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_emisor,
  codigo,
  punto_emision
  FROM
  DocElectronicoGuiaRemision.cabecera
  WHERE
  id_guia_remision = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string| Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_emisor | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__


### Transportista

Obtiene información del transportista en la guía de remisión

```sql
waybill_shipper  = SELECT
  identificacion,
  tipo_identificacion,
  razon_social,
  direccion,
  email,
  telefono
  FROM
  DocElectronicoGuiaRemision.cabecera
  WHERE
  id_guia_remision = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres. __Requerido__
direccion | string | Dirección
email | string | Correo electrónico. Máximo 300 caracteres.
telefono | string | Teléfono


### Destinatarios

Obtiene información de los destinatarios en la guía de remisión

```sql
waybill_receivers  = SELECT
    id_destinatario     receiver_id,
    razon_social,
    identificacion,
    tipo_identificacion,
    email,
    telefono,
    direccion,
    ruta,
    documento_aduanero_unico,
    codigo_establecimiento_destino,
    fecha_emision_documento_sustento,
    numero_documento_sustento,
    tipo_documento_sustento,
    motivo_traslado,
    numero_autorizacion_documento_sustento
    FROM
    DocElectronicoGuiaRemision.destinatario
    WHERE
    id_guia_remision = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
receiver_id | string | Identifica de manera única al destinatario en la guía de remisión. __Requerido__
razon_social | string | Razón social del destinatario. Máximo 300 caracteres __Requerido__
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
email | string | Correo electrónico del destinatario. Máximo 300 caracteres.
telefono | string | Teléfono del destinatario
direccion | string | Dirección del destinatario
ruta | string | Ruta de transporte. Máximo 300 caracteres.
documento_aduanero_unico | string |  Máximo 20 caracteres.
fecha_emision_documento_sustento | datetime  | Fecha de emisión del documento sustento de la guía de remisión, usualmente una factura.  __Requerido__
numero_documento_sustento | string | Número completo del documento que detalla la mercadería a transportar. Normalmente facturas. Ejm: 001-002-010023098
codigo_establecimiento_destino | string | Número establecimiento que recibe la entrega.  __Requerido__
numero_documento_sustento | string | Número completo del documento que detalla la mercadería a transportar. Normalmente facturas. Ejm: 001-002-010023098 __Requerido__
tipo_documento_sustento | string | tipo_documento_sustento | string | Códigos de [tipos de documentos](#tipos-de-documentos). __Requerido__
motivo_traslado | string | Motivo del traslado. Ejm: Entrega de mercadería. __Requerido__
numero_autorizacion_documento_sustento | string | Autorización del documento de sustento.

### Items del destinatario

Obtiene la información de los items que recibirá el destinatario

```sql
waybill_receiver_items = SELECT
    id_detalle,
    descripcion,
    codigo_principal,
    codigo_auxiliar,
    cantidad
    FROM
    DocElectronicoGuiaRemision.detalle
    WHERE
    id_destinatario = ?

```


Campo | Tipo | Descripción
--------- | ------- | -----------
id_detalle | string | Identifica de manera única el ítem o detalle de la factura. Si no hay un solo campo que lo identifique de manera única se debe usar la concatenación de varios.__Requerido__
descripcion | string | Descripción del ítem. __Requerido__
codigo_principal | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres. __Requerido__
codigo_auxiliar | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
cantidad | float | Cantidad de items. __Requerido__


### Detalles adicionales de items

Obtiene los detalles adicionales de un ítem. Este query es opcional.

Los detalles adicionales de un ítem se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Peso':'Kg'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con detalles adicionales del ítem:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Peso        |   KG
Color           |   Rojo
Caducidad                 |   10 días

```sql
item_details = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores   _valor_
  FROM
  DocElectronicoGuiaRemision.items_detalles_adicionales
  WHERE
  id_detalle = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
nombre | string | Nombre del detalle adicional del ítem
valor | string | Valor del detalle adicional del ítem


### Información adicional

Obtiene la información adicional de la guía de remisión.

La información adicional de la guía de remisión se maneja de la forma 'Clave':'Valor'. Ejemplo: 'Tipo de pago':'Cheque'

Se asume que en la tabla consultada
una columna tiene los nombres y otra los valores.

Ejemplo de columnas con información adicional de la guía de remisión:



columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Tipo de servicio        |   Avanzado
Forma de pago           |   Cheque
Periodo                 |   3 meses

```sql
waybill_additional_information = SELECT
  columna_de_nombres    _nombre_,
  columna_de_valores     _valor_
  FROM
  DocElectronicoRetencion.informacion_adicional
  WHERE
  id_retencion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la información adicional de la rentención
`_valor_` | string | Valor de la información adicional de la retención

### Tablas recomendadas

Estructura recomendada para las tablas o vistas con información de la guía de remisión.

Ejemplo en SQL Server:

```sql
CREATE SCHEMA guias_de_remision

DROP TABLE [guias_de_remision].[guia_remision]
DROP TABLE [guias_de_remision].[destinatario]
DROP TABLE [guias_de_remision].[item]
DROP TABLE [guias_de_remision].[item_detalle_adicional]
DROP TABLE [guias_de_remision].[informacion_adicional]

CREATE TABLE [guias_de_remision].[guia_remision](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [ambiente] [int] NOT NULL,
    [tipo_emision] [int] NOT NULL,
    [secuencial] [bigint] NOT NULL,
    [fecha_inicio_transporte] [datetime] NULL,
    [fecha_fin_transporte] [datetime] NULL,
    [direccion_partida] [varchar](200) NULL,
    [clave_acceso] [varchar](49),
    -- EMISOR
    [ruc] [varchar](13) NULL,
    [obligado_contabilidad] [varchar](2) NULL,
    [contribuyente_especial] [int] NULL,
    [nombre_comercial] [varchar](300) NULL,
    [razon_social] [varchar](300) NULL,
    [direccion] [varchar](300) NULL,
    [codigo_establecimiento] [varchar](3) NULL,
    [punto_emision] [varchar](3) NULL,
    [direccion_establecimiento] [varchar](300) NULL,
    -- TRANSPORTISTA
    [email_comprador] [varchar](254) NULL,
    [identificacion_comprador] [varchar](20) NULL,
    [tipo_identificacion_comprador] [varchar](2) NULL,
    [razon_social_comprador] [varchar](200) NULL,
    [direccion_comprador] [varchar](200) NULL,
    [telefono_comprador] [varchar](200) NULL,
    [placa] [varchar](200) NULL
)

-- GUIA DE REMISION: DESTINATARIOS
CREATE TABLE [guias_de_remision].[destinatario](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_destinatario] bigint NOT NULL FOREIGN KEY REFERENCES [guias_de_remision].[guia_remision](id),
    [email] [varchar](254) NULL,
    [identificacion] [varchar](20) NULL,
    [tipo_identificacion] [varchar](2) NULL,
    [razon_social] [varchar](200) NULL,
    [direccion] [varchar](200) NULL,
    [telefono] [varchar](200) NULL,
    [fecha_emision_documento_sustento] [datetime] NULL,
    [numero_documento_sustento] [varchar](17) NULL,
    [tipo_documento_sustento] [varchar](2) NULL,
    [numero_autorizacion_documento_sustento] [varchar](300) NULL,
    [ruta] [varchar](300) NULL,
    [motivo_traslado] [varchar](300) NULL,
    [documento_aduanero_unico] [varchar](300) NULL,
    [codigo_establecimiento_destino] [varchar](3) NULL
)

-- GUIA DE REMISION: ITEMS
CREATE TABLE [guias_de_remision].[item](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_destinatario] bigint NOT NULL FOREIGN KEY REFERENCES [guias_de_remision].[destinatario](id),
    [cantidad] [decimal](14,2)  NOT NULL,
    [codigo_principal] [varchar](50)  NULL,
    [codigo_auxiliar] [varchar](50)  NULL,
    [descripcion] [varchar](300)  NOT NULL
)

-- GUIA DE REMISION: DETALLES ADICIONALES DE ITEMS
CREATE TABLE [guias_de_remision].[item_detalle_adicional](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [guias_de_remision].[item](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT pk_items_detalles_adicionales PRIMARY KEY(id_item, nombre)
)

-- GUIA DE REMISION: INFORMACION ADICIONAL DE LA GUIA DE REMISION
CREATE TABLE [guias_de_remision].[informacion_adicional](
    [id_guia_remision] bigint NOT NULL FOREIGN KEY REFERENCES [guias_de_remision].[guia_remision](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT PK_informacion_adicional PRIMARY KEY (id_guia_remision, nombre)
)

```

<h2 id="liq-compra-advanced">Liquidaciones de compra</h2>

Los queries para la emisión electrónica de __liquidaciones de compra__ se guardan en el archivo de configuración `purchase_settlement.ini`.

[Ejemplo de archivo purchase_settlement.ini](/link-app#purchase_settlement-ini)

### Cabecera

Obtiene información de la cabecera de la liquidación de compra

```sql
headers = SELECT
  id                      id_local,
  secuencial,
  fecha_emision,
  moneda,
  tipo_emision
  FROM
  liquidaciones_compra.liquidacion
  WHERE
  id in (:sequence)
  ORDER BY id :order
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_local | int o string | Identifica de manera única la liquidación de compra. __Requerido__
secuencial | string  | Número de secuencia de la liquidación. __Requerido__
fecha_emision | datetime  | Fecha en la que se emite la liquidación.
moneda | string  | Código [ISO](https://en.wikipedia.org/wiki/ISO_4217) de la moneda. __Requerido__
tipo_emision | int | Emisión normal: `1`.<br>Emisión por indisponibilidad: `2`<br>__Requerido__

### Emisor

Obitiene información del emisor de la liquidación de compra

```sql
purchase_settlement_buyer  = SELECT
  ruc,
  obligado_contabilidad,
  contribuyente_especial,
  nombre_comercial,
  razon_social,
  direccion_establecimiento,
  direccion_matriz              direccion_emisor,
  codigo_establecimiento        codigo,
  punto_emision
  FROM
  liquidaciones_compra.liquidacion
  WHERE
  id = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
ruc | string | Número de RUC de 13 caracteres. __Requerido__
obligado_contabilidad | string | `'SI'` si está obligado a llevar contabilidad. `'NO'` si no lo está.
contribuyente_especial | string | Número de resolución. En blanco `''` si no es contribuyente especial.
nombre_comercial | string | Nombre comercial. Máximo 300 caracteres __Requerido__
razon_social | |
direccion_establecimiento | string | Dirección registrada en el SRI. Máximo 300 caracteres. __Requerido__
direccion_emisor | string | Dirección del punto de emisión. Máximo 300 caracteres. __Requerido__
codigo | string | Código numérico de 3 caracteres que representa al establecimiento. Ejemplo: `001` __Requerido__
punto_emision | string | Código numérico de 3 caracteres que representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__

### Proveedor

Obtiene información del proveedor de la liquidación de compra

```sql
purchase_settlement_provider = SELECT
  identificador_proveedor                   identificacion,
  tipo_identificador_proveedor              tipo_identificacion,
  razon_social_proveedor razon_social,
  direccion_proveedor direccion
  FROM
  liquidaciones_compra.liquidacion
  WHERE
  id = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
identificacion | string | De 5 a 20 caracteres. __Requerido__
tipo_identificacion | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
razon_social | string | Razón social. Máximo 300 caracteres __Requerido__
direccion | string | Dirección

### Totales

Obtiene información de los valores totales de las liquidaciones

```sql
purchase_settlement_totals  = SELECT
  total_sin_impuestos,
  importe_total,
  descuento
  FROM
  liquidaciones_compra.liquidacion
  WHERE
  id = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
total_sin_impuestos | float | Total antes de los impuestos. __Requerido__
importe_total | float | Total incluyendo impuestos. __Requerido__
descuento | float | Suma de los descuentos de cada ítem y del descuento adicional. __Requerido__

### Impuestos de totales

Obtiene información de los impuestos de los totales de la factura

```sql
purchase_settlement_totals_taxes = SELECT
  codigo,
  codigo_porcentaje,
  base_imponible,
  valor
  FROM
  liquidaciones_compra.total_impuesto
  WHERE
  id_liquidacion = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__

### Items

Obtiene todos los items de una factura

```sql
items  = SELECT
  id id_detalle,
  codigo_principal,
  codigo_auxiliar,
  descripcion,
  cantidad,
  unidad_medida,
  precio_unitario,
  descuento,
  precio_total_sin_impuestos
  FROM
  liquidaciones_compra.item
  WHERE
  id_liquidacion = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_detalle | int | Identifica de manera única el ítem o detalle de la factura. __Requerido__
codigo_principal | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres. __Requerido__
codigo_auxiliar | string | Código alfanumérico de uso del comercio. Máximo 25 caracteres.
descripcion | string | Descripción del ítem. __Requerido__
cantidad | float | Cantidad de items. __Requerido__
unidad_medida | string | Unidad de medida. Ejemplo: Kilos.
precio_unitario | float | Precio unitario. __Requerido__
descuento | float | El descuento es aplicado por cada producto.
precio_total_sin_impuestos | float | Precio antes de los impuestos. Se obtiene multiplicando la `cantidad` por el `precio_unitario` __Requerido__

### Impuestos de items

Obtiene los impuestos de un item.

```sql
item_taxes  = SELECT
  id_item id_detalle,
  base_imponible,
  valor,
  tarifa,
  codigo,
  codigo_porcentaje
  FROM
  liquidaciones_compra.item_impuesto
  WHERE
  id_item = ?
```

Campo |  Descripción | Valor de ejemplo
--------- | -----------| ---------
id_detalle | |
base_imponible | float | Base imponible. __Requerido__
valor | float | Valor del total. __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__

### Detalles adicionales de items

Obtiene los detalles adicionales de un ítem.

Los detalles adicionales de un ítem se manejan de la forma 'Clave':'Valor'. Ejemplo: 'Peso':'Kg'

Se asume que en la tabla consultada una columna tiene los nombres y otra los valores.

Ejemplo de columnas con detalles adicionales del ítem:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Peso        |   KG
Color       |   Rojo
Caducidad   |   10 días

```sql
item_details = SELECT
  nombre    nombre,
  valor   valor
  FROM
  liquidaciones_compra.item_detalle_adicional
  WHERE
  id_item = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
nombre | string | Nombre del detalle adicional del ítem
valor | string | Valor del detalle adicional del ítem

### Información adicional

Obtiene la información adicional de la factura. Este query es __opcional__

La información adicional de la factura se maneja de la forma 'Clave':'Valor'. Ejemplo: 'Tipo de pago':'Cheque'

Se asume que en la tabla consultada una columna tiene los nombres y otra los valores.

Ejemplo de columnas con información adicional de la liquidación:

columna_de_nombres  |  columna_de_valores
-------------------- | --------------
Tipo de servicio        |   Avanzado
Forma de pago           |   Cheque
Periodo                 |   3 meses

```sql
purchase_settlement_additional_information = SELECT
  nombre    _nombre_,
  valor     _valor_
  FROM
  liquidaciones_compra.info_adicional
  WHERE
  id_liquidacion = ?
```

Campo | Tipo | Descripción
--------- | ------- | -----------
`_nombre_` | string | Nombre de la información adicional de la factura
`_valor_` | string | Valor de la información adicional de la factura

### Pagos

Obtiene la información de los pagos aplicables a la liquidación.

```sql
payment_methods = SELECT
  forma_pago,
  total total,
  unidad_tiempo,
  plazo
  FROM
  liquidaciones_compra.pago
  WHERE
  id_liquidacion = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
forma_pago          | string                  | Código del [tipo de forma de pago](#tipos-de-forma-de-pago). __Requerido__
total               | float                   | Total aplicable a la forma de pago especificada. __Requerido__
unidad_tiempo       | string                  | Especifica la unidad de tiempo en la cual se expresa el plazo.
plazo               | int                     | Especifica el plazo del tipo de pago.

### Reembolsos

Obtiene el reembolso aplicable a una liquidación.

```sql
purchase_settlement_reimbursement = SELECT
  r.id_reembolso,
  l.codigo_documento_reembolso
  FROM
  liquidaciones_compra.liquidacion AS l JOIN
  liquidaciones_compra.reembolso AS r
  ON
  l.id = r.id_liquidacion
  WHERE
  id = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
r.id_reembolso      | int     | Identificador del reembolso. __Reembolso__
l.codigo_documento_reembolso  | Identificador del documento de reembolso. __Reembolso__

### Totales de reembolosos

Obtiene la información de los totales de un reembolso

```sql
purchase_settlement_reimbursement_totals = SELECT
  total_comprobante_reembolso            subtotal,
  total_base_imponible_reembolso         total,
  total_impuesto_reembolso               total_impuestos
  FROM
  liquidaciones_compra.liquidacion
  WHERE
  id = ?
```
Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
subtotal            | float                   | Total del reembolso incluyendo base imponible e impuestos. __Requerido__
total               | float                   | Total del reembolso incluyendo la base imponible. __Requerido__
total_impuestos     | float                   | Total del reembolso incluyendo los impuestos

### Documennto reembolso

Obtiene la información de un reembolso.

```sql
purchase_settlement_reimbursement_document = SELECT
  codigo_documento_reembolso            codigo,
  id_reembolso                          id_documento,
  id_proveedor_reembolso,
  tipo_id_proveedor_reembolso,
  codigo_pais_pago_proveedor_reembolso,
  tipo_proveedor_reembolso,
  secuencia_reembolso,
  punto_emision_reembolso,
  fecha_emision_reembolso,
  numero_autorizacion_reembolso,
  codigo_establecimiento_reembolso
  FROM
  liquidaciones_compra.reembolso
  WHERE
  id_reembolso = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
codigo | string | Código numérico de 2 caracteres que representa al documento de reembolso. Ejemplo: `01` __Requerido__
id_documento | int | Identifica de manera única al documento de reemoblso. __Requerido__
id_proveedor_reembolso | string | Identifica de manera única al proveedor en la liquidación. __Requerido__
tipo_id_proveedor_reembolso | string | Ver [tabla](#tipo-de-identificacion) de tipos de identificación __Requerido__
codigo_pais_pago_proveedor_reembolso | string | Código de dos letras del país del proveedor según [ISO_3166](https://en.wikipedia.org/wiki/ __Requerido__
tipo_proveedor_reembolso | string | Tipo de proveedor __Requerido__
secuencia_reembolso | int | Número de secuencia de la factura. __Requerido__
punto_emision_reembolso | string | Código numérico de 3 caracteres que  representa al punto de emisión, o punto de venta. Ejemplo: `001`. __Requerido__
fecha_emision_reembolso | datetime | Fecha de emisión __Requerido__
numero_autorizacion_reembolso | string  | Autorización del documento de reembolso. __Requerido__
codigo_establecimiento_reembolso | float | Número establecimiento que recibe la entrega. __Requerido__

### Impuestos de reembolsos

Obtiene información de los impuestos de los reembolsos

```sql
purchase_settlement_reimbursement_tax = SELECT
  codigo,
  codigo_porcentaje,
  tarifa,
  base_imponible,
  impuesto_reembolso
  FROM
  liquidaciones_compra.reembolso_tax
  WHERE
  id_reembolso = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
codigo | string | Código del [tipo de impuesto](#tipos-de-impuesto) __Requerido__
codigo_porcentaje | string | Código del [porcentaje](#codigo-de-porcentaje-de-iva). __Requerido__
tarifa | float | Porcentaje actual del impuesto expresado por un número entre 0.0 y 100.0 __Requerido__
base_imponible | float | Base imponible. __Requerido__
impuesto_reembolso | float | Valor del impuesto del reembolso


### Máquina fiscal

Obtiene información de la máquina fiscal con la que se emitió la liquidación

```sql
purchase_settlement_fiscal_machine = SELECT
  marca,
  modelo,
  serie
  FROM
  liquidaciones_compra.maquina_fiscal
  WHERE
  id_liquidacion = ?
```

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
marca | string | Marca de máximo 100 caracteres
modelo | string | Modelo de máximo 100 caracteres
serie | string | Serie de máximo 100 caracteres

### Tablas recomendadas

Estructura recomendada para las tablas o vistas con información de las liquidaciones de compra.

Ejemplo en SQL Server:

```sql
CREATE SCHEMA liquidaciones_compra;
CREATE TABLE [liquidaciones_compra].[liquidacion](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [ambiente] [int] NOT NULL,
    [tipo_emision] [int] NOT NULL,
    [secuencial] [bigint] NOT NULL,
    [clave_acceso] [VARCHAR](49) NULL,
    [fecha_emision] [DATETIME] NULL,
    [moneda] [VARCHAR](15) NOT NULL,
    [codigo_documento] [VARCHAR](2) NOT NULL,
    -- EMISOR
    [ruc] [VARCHAR](13) NULL,
    [obligado_contabilidad] [VARCHAR](10) NULL,
    [contribuyente_especial] [VARCHAR](13) NULL,
    [nombre_comercial] [VARCHAR](300) NULL,
    [razon_social] [VARCHAR](300) NULL,
    [direccion_matriz] [VARCHAR](300) NOT NULL,
    [codigo_establecimiento] [VARCHAR](3) NULL,
    [punto_emision] [VARCHAR](3) NULL,
    [direccion_establecimiento] [VARCHAR](300) NULL,
    -- PROVEEDOR
    [tipo_identificador_proveedor] [VARCHAR](2) NULL,
    [razon_social_proveedor] [VARCHAR](300) NOT NULL,
    [identificador_proveedor] [VARCHAR](20) NOT NULL,
    [direccion_proveedor] [vARCHAR](300) NULL,
    -- TOTALES
    [total_sin_impuestos] [DECIMAL](14, 2) NOT NULL,
    [descuento] [DECIMAL](14, 2) NOT NULL,
    [importe_total] [DECIMAL](14, 2) NOT NULL,
    -- REEMBOLSO
    [codigo_documento_reembolso] [VARCHAR](2) NOT NULL,
    [total_comprobante_reembolso] [DECIMAL](14, 2) NOT NULL,
    [total_base_imponible_reembolso] [DECIMAL](14, 2) NOT NULL,
    [total_impuesto_reembolso] [DECIMAL](14, 2) NOT NULL,
)

-- LIQUIDACION: ITEMS
CREATE TABLE [liquidaciones_compra].[item](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_liquidacion] bigint NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [codigo_principal] [VARCHAR](25) NOT NULL,
    [codigo_auxiliar] [VARCHAR](25)  NULL,
    [descripcion] [VARCHAR](300)  NOT NULL,
    [unidad_medida] [VARCHAR](50) NULL,
    [cantidad] [DECIMAL](14,6)  NOT NULL,
    [precio_unitario] [DECIMAL](18,6)  NOT NULL,
    [descuento] [DECIMAL](14,2)  NULL,
    [precio_total_sin_impuestos] [DECIMAL](14,2)  NOT NULL,
)

-- LIQUIDIACION: DETALLES ADICIONALES DE ITEMS
CREATE TABLE [liquidaciones_compra].[item_detalle_adicional](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[item](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar](100) NOT NULL
    CONSTRAINT pk_items_detalles_adicionales PRIMARY KEY (id_item, nombre)
)

-- LIQUIDACION: IMPUESTOS DE ITEMS
CREATE TABLE [liquidaciones_compra].[item_impuesto](
    [id_item] [bigint] NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[item](id),
    [codigo] [VARCHAR](2) NOT NULL,
    [codigo_porcentaje] [VARCHAR](2) NOT NULL,
    [base_imponible] [DECIMAL](14,2) NOT NULL,
    [valor] [DECIMAL](14,2) NOT NULL,
    [tarifa] [DECIMAL](5,2) NOT NULL, -- porcentaje
    CONSTRAINT PK_item_impuesto PRIMARY KEY (id_item, codigo, codigo_porcentaje)
)

-- LIQUIDACION: IMPUESTOS TOTALES
CREATE TABLE [liquidaciones_compra].[total_impuesto](
    [id_liquidacion] bigint NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [codigo] [VARCHAR](2) NOT NULL,
    [codigo_porcentaje] [VARCHAR](2) NOT NULL,
    [descuento_adicional] [DECIMAL](14, 2) NULL,
    [base_imponible] [DECIMAL](14,2) NOT NULL,
    [tarifa] [DECIMAL](14, 2) NOT NULL,
    [valor] [DECIMAL](14,2) NOT NULL
    CONSTRAINT PK_total_impuesto PRIMARY KEY (id_liquidacion, codigo, codigo_porcentaje)
)

-- LIQUIDIACION: PAGOS
CREATE TABLE [liquidaciones_compra].[pago](
    [id] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_liquidacion] bigint NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [forma_pago] [VARCHAR](2) NOT NULL,
    [total] [DECIMAL](14, 2) NOT NULL,
    [plazo] VARCHAR(50) NOT NULL,
    [unidad_tiempo] VARCHAR(50) NOT NULL
)

-- LIQUIDACION: REEMBOLSOS
CREATE TABLE [liquidaciones_compra].[reembolso](
    [id_reembolso] bigint IDENTITY(1,1) PRIMARY KEY,
    [id_liquidacion] bigint NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [tipo_id_proveedor_reembolso] [VARCHAR](2) NOT NULL,
    [id_proveedor_reembolso] [VARCHAR](20) NOT NULL,
    [codigo_pais_pago_proveedor_reembolso] [VARCHAR](2) NOT NULL,
    [tipo_proveedor_reembolso] [VARCHAR](2) NOT NULL,
    [codigo_documento_reembolso] [VARCHAR](3) NOT NULL,
    [codigo_establecimiento_reembolso] [DECIMAL](3) NOT NULL,
    [punto_emision_reembolso] [VARCHAR](3) NOT NULL,
    [secuencial_reembolso] [bigint] NOT NULL,
    [fecha_emision_reembolso] [DATETIME] NOT NULL,
    [numero_autorizacion_reembolso] [VARCHAR](49) NOT NULL
)

-- LIQUIDACION: REEMBOLSO IMPUESTOS
CREATE TABLE [liquidaciones_compra].[reembolso_tax](
    [id_reembolso] [bigint] NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[reembolso](id_reembolso),
    [codigo] [VARCHAR](2) NOT NULL,
    [codigo_porcentaje] [VARCHAR](2) NOT NULL,
    [tarifa] [DECIMAL](14, 2) NOT NULL,
    [base_imponible] [DECIMAL](14,2) NOT NULL,
    [impuesto_reembolso] [DECIMAL](14, 2) NOT NULL
)

-- LIQUIDACION: MAQUINA FISCAL
CREATE TABLE [liquidaciones_compra].[maquina_fiscal](
    [id_liquidacion] bigint NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [marca] [VARCHAR](100) NOT NULL,
    [modelo] [VARCHAR](100) NOT NULL,
    [serie] [VARCHAR](100) NOT NULL,
    CONSTRAINT pk_maquina_fiscal PRIMARY KEY (id, marca)
)

-- LIQUIDACION: INFO ADICIONAL
CREATE TABLE [liquidaciones_compra].[info_adicional](
    [id_liquidacion] [bigint] NOT NULL FOREIGN KEY REFERENCES [liquidaciones_compra].[liquidacion](id),
    [nombre] [varchar](100) NOT NULL,
    [valor] [varchar(100)] NOT NULL,
    CONSTRAINT pk_info_adicionales PRIMARY KEY (id_liquidacion, nombre)
)
```

## Consulta de documentos

La información que Link consulta cuando un documento ha sido emitido y el query `on_status_update` ha sido definido son los mismos detallados en la [consulta de documentos](https://datil.dev/#envio-sri). Se puede acceder y personalizar la sentencia SQL utilizando las etiquetas [Jinja2](http://jinja.pocoo.org/)

> #### Ejemplo para guardar autorizacion de facturas

```sql
on_status_update = {% if autorizacion is defined %}
INSERT INTO dbo.RESPUESTA_SRI_FACTURSA (
  id,
  numero,
  clave_acceso,
  codig_estado
) VALUES (
  {{ id_local }},
  {{ autorzacio.numero }},
  {% if autorizacion.numero == 'AUTORIZADO' %} 
    100
  {% else %}
    0
  {% endif %}
)
```

## Configuración de Ambiente
La configuración general de la aplicación __Link-App__ se guardan en el archivo de configuración `environment.ini`.
A continuación se describen las configuraciones de ambiente necesarias para el correcto funcionamiento de Link-app.

### General
La configuración general se encuentra en la sección `[General]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
timezone | string | Zona horaria
issue_receipts_from_database | string | Habilitar la emisión de documentos desde la base de datos
issue_receipts_from_xml | string | Habilitar la emisión de documentos desde archivos en formato xml
update_control_table | string | Habilitar el control de los documentos emitidos desde la tabla de Control

### Configuración de la base de datos
La configuración de la base de datos se encuentra en la sección<br>`[DatabaseSource]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
driver | string | Nombre del driver de la base de datos
server | string | Nombre del servidor
name | string | Nombre de la base de datos a la cuál se conectará la aplicación
user | string | Nombre del usuario con el que se autenticará la aplicación en la base de datos
password | string | Contraseña con el que se autenticará la aplicación en la base de datos
api | string | API para acceder a la base de datos.<br> APIS: `odbc` y `adodb`

### Registros de la aplicación
La configuración acerca de los registros generados por la aplicación se encuentra en la sección `[Log]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
level | string | Nivel de los registros (En el caso de que se esté configurando por primera vez la aplicación se recomienda mantener el nivel de `DEBUG`)<br>Valores posibles: `INFO`, `DEBUG` y `ERROR`. 
interval | integer | Intervalo de tiempo en horas que se genera un nuevo archivo de registros
backup_count | interger | Cantidad de archivos de registros guardados

### Sincronización General de documentos
La configuración de la sincronización de eventos se encuentra en la sección<br>`[Sync]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
enabled | string | Habilitar la sincronización de documentos
update_tables | string | Habilitar la actualización de tablas cuando se sincronicen los documentos
download_files | string | Habilitar la descarga de archivos cuando se sincronicen los documentos

### Código de Eventos
La configuración de los tipos de eventos se encuentra en la sección `[EventTypeCodes]`. Para esta configuración se debe especificar el nombre del documento con el evento y como valor se le asigna el código numérico del evento.<br>
Ejemplo:<br>
`invoice.issued = 21`<br>
`invoice.received = 11`

### Restricciones para la emisión
La configuración de las restricciones para emitir documentos se encuentra en la sección `[Constraints]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
first_receipt_date | date | Fecha inicial para la consulta de documentos a emitir
max_days_to_query | date | Cantidad de días atrás para la consulta de documentos a emitir

## Configuración de la Compañía
La configuración de la compañía de la aplicación __Link-App__ se guardan en el archivo de configuración `companies/company.ini`.
A continuación se describen las configuraciones de ambiente necesarias para el correcto funcionamiento de Link-app.

### General
La configuración general se encuentra en la sección `[General]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
enabled | string | Habilitar la emisión de documentos para la compañía
ruc | string | RUC de la compañía que emitirá los documentos

### API
La configuración de la API se encuentra en la sección `[API]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
xkey | string | API Key para emitir documentos. Esta información se encuentra en la configuración de la compañía en el portal web
xpassword | string | Contraseña del certificado de firma electrónica
environment | integer | Pruebas: `1`.<br>Producción `2`.

### Emisión de Documentos desde la base de datos
Habilitar la emisión de documentos desde la base de datos. Para esta configuración se debe especificar el nombre del documento y como valor se le asigna `yes` o  `no`.<br>
Ejemplo:
`invoice = yes`

### Emisión de Documentos desde XML
Habilitar la emisión de documentos desde documentos XML. Para esta configuración se debe especificar el nombre del documento y como valor se le asigna `yes` o  `no`.<br>
Ejemplo:
`credit_note = yes`

### Sincronización de documentos
La configuración de la sincronización de eventos se encuentra en la sección<br>`[Sync]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
enabled | string | Habilitar la sincronización de documentos
update_tables | string | Habilitar la actualización de tablas cuando se sincronicen los documentos
download_files | string | Habilitar la descarga de archivos cuando se sincronicen los documentos

Además se debe habilitar la sincronización por cada tipo de documento de la misma forma de la que se habilita la emisión de documentos desde la base de datos o desde documentos XML.<br>
Ejemplo: `waybill = yes`

### Configuración de la base de datos
La configuración de la base de datos se encuentra en la sección<br>`[DatabaseSource]`.

Parámetro           | Tipo                    | Descripción
------------------- | ----------------------- | ----------
driver | string | Nombre del driver de la base de datos
server | string | Nombre del servidor
name | string | Nombre de la base de datos a la cuál se conectará la aplicación
user | string | Nombre del usuario con el que se autenticará la aplicación en la base de datos
password | string | Contraseña con el que se autenticará la aplicación en la base de datos
api | string | API para acceder a la base de datos.<br> APIS: `odbc` y `adodb`

### Ruta de documentos XML
La configuración de las rutas dónde los documentos XML se almacenarán se encuentra en la sección `[XmlSource]`.
Para esta configuración se debe especificar el nombre del documento y como valor se le asigna la ruta en la que se encuentra el tipo de documento.<br>
Ejemplo: `invoice = C:\\Program Files\Facturas`

### Patrón de documentos XML
Para emitir documentos XML desde la ruta configurada del documento se necesita definir un patrón que será el prefijo de los nombres de los documentos XML.<br>
Ejemplo: `invoice = FA`