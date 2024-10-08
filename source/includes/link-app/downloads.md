
# Descargas

Integra Dátil a tu sistema en minutos.

### Más reciente (6.4.1)

- [Windows 32 bits](https://linkapp-installers.s3.us-west-2.amazonaws.com/Datil+LinkApp_6.4.1_32bit.exe)
- [Windows 64 bits](https://linkapp-installers.s3.us-west-2.amazonaws.com/Datil+LinkApp_6.4.1_64bit.exe)

#### Notas de la versión

#### <strong>6.4.1</strong><br><small><strong>Enero 11, 2023</strong></small>

##### Corregido
Se agregaron nuevas entradas a la configuración de ats_retention.ini que permite emitir Comprobantes de Retención versión 2.0.0 de Exportaciones y otros Ingresos del Exterior:
<ul>
    <li>Tipo de Sujeto Retenido</li>
    <li>Tipo de Régimen Fiscal</li>
    <li>País</li>
    <li>Aplica Convenio de Doble Tributación</li>
    <li>Pago Exterior</li>
    <li>Pago Régimen Fiscal</li>
</ul>

#### 6.4.0

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.4.0_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.4.0_64bit.exe)

#### Notas de la versión

#### <strong>6.4.0</strong><br><small><strong>Noviembre 15, 2022</strong></small>

##### Nuevo
Emitir Comprobantes de Retención versión 2.0.0

#### 6.2.4

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.4_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.4_64bit.exe)

#### Notas de la versión

#### <strong>6.2.4</strong><br><small><strong>Enero 31, 2020</strong></small>

##### Nuevo
Emitir liquidaciones de compra a través de archivos XML

##### Corregido
Error al emitir documentos a través de archivos XML

### Versiones anteriores

#### 6.2.3

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.3_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.3_64bit.exe)

#### Notas de la versión

#### <strong>6.2.3</strong><br><small><strong>Enero 15, 2020</strong></small>

##### Corregido
Error al momento de emitir una liquidación de compra.


#### 6.2.2

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.2_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.2_64bit.exe)

#### Notas de la versión

#### <strong>6.2.2</strong><br><small><strong>Enero 3, 2020</strong></small>

##### Corregido
Mejora en información de pagos para compatibilidad con Oracle.


#### 6.2.1

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.1_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.2.1_64bit.exe)

#### Notas de la versión

#### <strong>6.2.1</strong><br><small><strong>Diciembre 24, 2019</strong></small>

##### Corregido
Error al momento de obtener información de una liquidación de compras sin que ésta tenga registrada una máquina fiscal.


#### 6.2.0

- Windows 32 bits
- Windows 64 bits

#### Notas de la versión

#### <strong>6.2.0</strong><br><small><strong>Octubre 30, 2019</strong></small>

##### Nuevo
Soporte para Microsoft Access. 


#### 6.1.0

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.1.0_32bit_generic.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_6.1.0_64bit_generic.exe)

#### Notas de la versión

#### <strong>6.1.0</strong><br><small><strong>Octubre 24, 2019</strong></small>

##### Nuevo
Soporte para poder liquidaciones de compra

##### Corregido
Corrige error de regresión en la tarea `sync_company_resources`

#### 5.0.1

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_5.0.1_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_5.0.1_64bit.exe)

#### Notas de la versión

#### <strong>5.0.1</strong><br><small><strong>Enero 5, 2018</strong></small>

##### Corregido
La longitud máxima para el nombre de una columna en versiones de ORACLE menores
a la 12 es de 30 caracteres. Por esto agregamos alias más cortos para parámetros
que exceden esta longitud, como `fecha_emision_documento_sustento -> fecha_emi_doc_sustento`,
`numero_autorizacion_documento_sustento -> num_aut_doc_sustento`

#### 5.0.0

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_5.0.0_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_5.0.0_64bit.exe)

#### Notas de la versión

#### <strong>5.0.0</strong><br><small><strong>Enero 5, 2018</strong></small>

##### Nuevo
Soporte para ORACLE.<br/>
Personalización del formato de fecha utilizado como literal de fecha utilizando
la nueva entrada de configuración `datetime_format`.<br/>

##### Removido
La tarea `sync_resources` fue removida completamente en favor de tarea más
reciente `resource_sync`.


#### 4.7.0

- [Windows 32 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_4.7.0_32bit.exe)
- [Windows 64 bits](https://s3-us-west-2.amazonaws.com/linkapp-installers/Datil+LinkApp_4.7.0_64bit.exe)

#### Notas de la versión

#### <strong>4.7.0</strong><br><small><strong>Octubre 19, 2017</strong></small>

##### Nuevo
Emisión de facturas de reembolso. Se agregaron tres nuevas entradas al archivo
de configuración `config/receipts/invoice.ini`: invoice_reimbursement,
invoice_reimbursement_document, invoice_reimbursement_document_tax

