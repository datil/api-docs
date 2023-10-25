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
obligado_contabilidad | boolean | `true` si está obligado a llevar contabilidad. __Requerido__
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

Datos de una factura, liquidación de compras o retención ats de reembolso.

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

## Documentos Soporte

Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo_sustento | string | Ver [tabla](#tipos-de-sustento-de-comprobantes) de tipos de sustento de los comprobantes. __Requerido__
tipo_documento | string | Ver códigos de [tipos de documentos](#tipos-de-documentos). __Requerido__
numero | string | Número completo del documento asociado a la retención ATS. __Requerido__
fecha_emision | string |  Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
fecha_registro_contable | string | Fecha de registro contable del comprobante de venta en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6).
numero_autorización | string | Número de autorización del comprobante de venta. __Requerido__
tipo_pago | string | Ver códigos de [tipos de pagos](#tipos-de-pago). __Requerido__
total_sin_impuestos | string | Total antes de los impuestos. __Requerido__
total | string | Total incluyendo impuestos. __Requerido__
tipo_regimen_fiscal | string | Ver [tabla](#tipo-de-regimen-fiscal) de tipos de régimen fiscal (Requerido si la identificación del sujeto retenido es Identificación del exterior)
pais | string | Código de dos caracteres del país según [ISO_3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements) (Requerido si la identificación del sujeto retenido es Identificación del exterior)
aplica_convenio | boolean | `true` si el pago está sujeto a algún convenio de doble tributación (Requerido si la identificación del sujeto retenido es Identificación del exterior)
pago_exterior | boolean | `true` si el pago realizado al exterior aplica retención (Requerido si la identificación del sujeto retenido es Identificación del exterior)
pago_regimen_fiscal | boolean | `true` si el pago realizado a un no residente se encuentra en un régimen fiscal preferente o de menor imposición (Requerido si la identificación del sujeto retenido es Identificación del exterior)
impuestos | listado de objetos [impuesto](#impuesto-item) | Impuestos totales del documento. __Requerido__
retenciones | listado de objetos [retenciones ats](#impuestos-retenidos-en-retencion-ats) | Listado de impuestos retenidos __Requerido__
reembolso | objeto tipo [reembolso](#reembolso) | Información de reembolso.
pagos | listado de objetos tipo [pagos](#pagos-de-documentos-de-soporte) | Información de los pagos __Requerido__

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

<h2 id="documento-sustento">Documento de Sustento</h2>

Información de la factura asociada a las notas de créditos.

Parámetro | Tipo | Descripción
--------- | ---- |-----------
numero_documento | string | Número completo de la factura asociada a la nota de crédito.
tipo_documento | string | Ver códigos de [tipos de documentos](#tipos-de-documentos).
fecha_documento | string | Fecha de emisión en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6).
# Catálogo

## Tipo de identificación

Tipo de identificación      | Código
--------------------------- | ------
RUC                         | `04`
CEDULA                      | `05`
PASAPORTE                   | `06`
VENTA A CONSUMIDOR FINAL*   | `07`
IDENTIFICACION DELEXTERIOR* | `08`
PLACA                       | `09`

## Tipo de régimen fiscal

Código | Tipo de régimen fiscal
-------- | ---------------------
01 | Régimen general 
02 | Paraíso fiscal
03 | Regimen fiscal preferente o jurisdicción de menor imposición

## Tipo de sujeto retenido

Código | Tipo de sujeto
-------- | ---------------------
01 | Persona natural
02 | Sociedad

## Tipos de impuesto

Impuesto | Código
-------- | ------
IVA      | 2
ICE      | 3
IRBPNR   | 5

## Tipos de Sustento de Comprobantes

Código | Tipo de Sustento
-------- | ----------
01 | Crédito Tributario para declaración de IVA (servicios y bienes distintos de inventarios y activos fijos)
02 | Costo o Gasto para declaración de IR (servicios y bienes distintos de inventarios y activos fijos)
03 | Activo Fijo - Crédito Tributario para declaración de IVA
04 | Activo Fijo - Costo o Gasto para declaración de IR
05 | Liquidación Gastos de Viaje, hospedaje y alimentación Gastos IR (a nombre de empleados y no de la empresa)
06 | Inventario - Crédito Tributario para declaración de IVA
07 | Inventario - Costo o Gasto para declaración de IR
08 | Valor pagado para solicitar Reembolso de Gasto (intermediario)
09 | Reembolso por Siniestros
10 | Distribución de Dividendos, Beneficios o Utilidades
11 | Convenios de débito o recaudación para IFI ́s
12 | Impuestos y retenciones presuntivos
13 | Valores reconocidos por entidades del sector público a favor de sujetos pasivos
14 | Valores facturados por socios a operadoras de transporte (que no constituyen gasto de dicha operadora)
15 | Pagos efectuados por consumos propios y de terceros de servicios digitales
00 | Casos especiales cuyo sustento no aplica en las opciones anteriores

## Tipos de Pago
Código | Pago residente o no residente
------- | -------------------
01 | Pago a residente / Establecimiento permanente
02 | Pago a no residente

## Impuestos Retenidos en Retención ATS
Parámetro | Tipo | Descripción
--------- | ---- |-----------
codigo | string | Código del [tipo de impuesto para la retención en la factura](#tipos-de-impuesto-para-la-retencion-en-la-factura).  __Requerido__
codigo_porcentaje | string | Código del [porcentaje del impuesto](#retencion-de-iva-presuntivo-y-renta). __Requerido__
base_imponible | float | Suma de las bases imponibles de cada item para el tipo de impuesto y porcentaje. __Requerido__
tarifa | float (hasta 2 cifras decimales) | Porcentaje actual del impuesto.  __Requerido__
valor_retenido | float (hasta 2 cifras decimales) | Valor del impuesto.  __Requerido__
dividendos | Listado de objetos [dividendo](#dividendo) | Participaciones en utilidades, excedentes, beneficios o similares que se obtienen en razón de los derechos representativos de capital que el beneficiario mantiene, de manera directa o indirecta.

## Dividendo
Parámetro | Tipo | Descripción
--------- | ---- |-----------
fecha_pago | string | Fecha de pago en formato AAAA-MM-DDHoraZonaHoraria, definido en el estándar [ISO8601](http://tools.ietf.org/html/rfc3339#section-5.6). __Requerido__
impuesto_renta | string | Impuesto a la Renta pagado por la sociedad correspondiente al dividendo __Requerido__
annio_fiscal | integer | Año en que se generaron las utilidades atribuibles al dividendo. __Requerido__

## Pagos de Documentos de Soporte
Parámetro | Tipo | Descripción
---------- | ----- | --------------
tipo_pago | string | Código de [forma de pago](#tipos-de-forma-de-pago-del-sri) del SRI. __Requerido__
total | string | Total del pago. __Requerido__

## Código de Porcentaje de IVA

Porcentaje de IVA | Código | Tarifa
-------- | ------ | ------
0%     | 0 |  0
12%      | 2 | 12
14%   | 3 | 14
No Objeto de Impuesto   | 6 | -
Exento de IVA   | 7 | -

## Tipos de impuesto para la retención

Impuesto | Código
-------- | ------
RENTA    | 1
IVA      | 2
ISD      | 6

## Tipos de impuesto para la retención en la factura

Caso específico de Retenciones en la Comercializadores / Distribuidores de derivados del
Petróleo y Retención presuntiva de IVA a los Editores, Distribuidores y Voceadores que
participan en la comercialización de periódicos y/o revistas.

Impuesto                  | Código
------------------------- | ------
IVA PRESUNTIVO Y RENTA    | 4



## Retención de IVA

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

Porcentaje IVA | Código
-------------- | ------
5%             | 4580


## Retención a la Fuente (Renta)

Consulta el catálogo vigente de códigos en:

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

Documento                | Código
------------------------ | ------
Factura                  | 01
Liquidación de Compra    | 03
Nota de Crédito          | 04
Nota de Débito           | 05
Guía de Remisión         | 06
Comprobante de Retención | 07
Boletos o entradas a espectáculos públicos | 08
Tiquetes o vales emitidos por máquinas registradoras | 09
Pasajes expedidos por empresas de aviación | 11
Documentos emitidos por instituciones financieras | 12
Comprobante de venta emitido en el Exterior | 15
Formulario Único de Exportación (FUE) o Declaración Aduanera Única (DAU) o Declaración Andina de Valor (DAV) | 16
Documentos autorizados utilizados en ventas excepto N/C N/D | 18
Comprobantes de Pago de Cuotas o Aportes | 19
Documentos por Servicios Administrativos emitidos por Inst. del Estado | 20
Carta de Porte Aéreo | 21
RECAP | 22
Nota de Crédito TC | 23
Nota de Débito TC | 24
Comprobante de venta emitido por reembolso | 41
Documento retención presuntiva y retención emitida por propio vendedor o por intermediario | 42
Liquidación para Explotación y Exploración de Hidrocarburos | 43
Comprobante de Contribuciones y Aportes | 44
Liquidación por reclamos de aseguradoras | 45
Nota de Crédito por Reembolso Emitida por Intermediario | 47
Nota de Débito por Reembolso Emitida por Intermediario | 48
Proveedor Directo de Exportador Bajo Régimen Especial | 49
A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta | 50
N/C A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta | 51
N/D A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta | 52
Liquidación de compra de Bienes Muebles Usados | 294
Liquidación de compra de vehículos usados | 344
Acta Entrega-Recepción PET | 364
Factura operadora transporte / socio | 370
Comprobante socio a operadora de transporte | 371
Nota de crédito operadora transporte / socio | 372
Nota de débito operadora transporte / socio | 373
Nota de débito operadora transporte / socio | 374
Liquidación de compra RISE de bienes o prestación de servicios | 375

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
