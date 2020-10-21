# Consulta de autorización

Consulta la información de autorización de cualquier tipo de comprobante electrónico.
El ID del documento `id-doc` es el ID que obtienes después de emitir un documento.

<h3 id="consulta-comprobante-op">Operación</h3>

`GET /edocs/<id-doc>`

<h3 id="consulta-comprobante-requerimiento">Requerimiento</h3>

> #### Requerimiento de ejemplo

```shell
curl -v https://link.datil.co/edocs/<id-doc> \
-H "Accept: application/json" \
-H "X-Key: <clave-del-api>"
```

```python
import requests
cabeceras = {'x-key': '<clave-del-api>'}
respuesta = requests.get(
    'https://link.datil.co/edocs/<id-factura>',
    headers = cabeceras)
```

```csharp
using RestSharp;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace DatilClient {
  class InvoicingServiceClient {
    static void Main(string[] args) {

      var client = new RestClient("https://link.datil.co/");
      var idDocumento = "<id-doc>";
      var request = new RestRequest("edocs/" + idDocumento, Method.GET);
      request.AddHeader("X-Key", "<clave-del-api>");

      IRestResponse response = client.Execute(request);

      Console.WriteLine(response.Content);
    }
  }
}
```

<h3 id="consulta-comprobante-respuesta">Respuesta</h3>

Parámetro    | Tipo    | Descripción
------------ | ------- | -----------
id           | string  | El id del documento consultado
tipo         | string  | Código que representa el tipo de documento. Revisa [aquí](#tipos-de-documentos) el código que corresponde a cada tipo de documento
estado       | string  | Estado de autorización del comprobante. Posibles valores: `AUTORIZADO`, `NO AUTORIZADO`, `ENVIADO`, `DEVUELTO`, `RECIBIDO`, `ERROR`
clave_acceso | string  | Clave de acceso del documento.
url_formato_impresion | url | Esta URL te permite acceder de manera directa al formato de impresión (RIDE) del comprobante
url_documento_electronico | url | Esta URL te permite acceder de manera directa al documento electrónico (XML)
autorizacion | Objeto de tipo [autorización SRI](#autorizacion-sri)

> #### Respuesta de ejemplo

```json
{
    "url_documento_electronico": "https://app.datil.co/ver/be69b7bc64b643718a643caa9a8c3569/xml",
    "autorizacion": {
        "fecha": "2020-01-23T15:41:51Z",
        "estado": "AUTORIZADO",
        "mensajes": [],
        "numero": "2301202035679285132400120010020000287082794874518"
    },
    "tipo": "01",
    "url_formato_impresion": "https://app.datil.co/ver/be69b7bc64b643718a643caa9a8c3569/pdf",
    "clave_acceso": "2301202035679285132400120010020000287082794874518",
    "estado": "AUTORIZADO",
    "id": "be69b7bc64b643718a643caa9a8c3569",
    "ambiente": "2"
}
```

# Descarga de comprobantes

## Consulta de RIDE

Consulta de la representación impresa del documento electrónico (RIDE). El ID del comprobante `id-doc` es el ID que se obtiene después de emitir un comprobante.

### Operación

`GET app.datil.co/ver/<id-doc>/pdf`

## Consulta de XML

Consulta de representación XML de los comprobantes. El ID del comprobante `id-doc` es el ID que se obtiene después de emitir un comprobante.

### Operación

`GET app.datil.co/ver/<id-doc>/xml`

# Envío por correo

Envíe cualquier tipo de comprobantes por correo electrónico hacia la persona que va dirigada el comprobante o una lista de destinatarios.

## Envío simple

Envíe el comprobante al correo electrónico que está definido en el comprobante

#### Operación

`POST edocs/send-email/<id-doc>`

#### Requerimiento

Reemplaza en la ruta `<id-doc>` por el `id` del comprobante a enviar por correo.  

```shell
curl -v https://link.datil.co/edocs/send-email/<id-doc> \
-H "Content-Type: application/json" \
-H "X-Key: <clave-del-api>"

```

```python
import requests
cabeceras = {
  'x-key': '<clave-del-api>',
  'content-type': 'application/json'}
respuesta = requests.post(
    'https://link.datil.co/edocs/send-email/<id-factura>',
    headers = cabeceras)
```

```csharp
using RestSharp;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace DatilClient {
  class InvoicingServiceClient {
    static void Main(string[] args) {

      var client = new RestClient("https://link.datil.co/");
      var idDocumento = "<id-doc>";
      var request = new RestRequest("edocs/send-email/" + idDocumento, Method.POST);
      request.AddHeader("X-Key", "<clave-del-api>");
      request.AddHeader("Content-Type", "application/json");

      IRestResponse response = client.Execute(request);

      Console.WriteLine(response.Content);
    }
  }
}
```

### Respuesta

Parámetro    | Tipo    | Descripción
------------ | ------- | -----------
id           | string  | El id del documento al cual se envío el correo.
result       | string  | Resultado del requerimiento.

## Envio a multiples receptores

Envié cualquier tipo de comprobante a multiples receptores especificando los correos electrónicos de estos en el cuerpo del requerimiento en formato JSON.

#### Operación

`POST edocs/send-email/<id-doc>`

#### Requerimiento

Reemplaza en la ruta `<id-doc>` por el `id` del comprobante a enviar por correo. El cuerpo del requerimiento debe de tener una lista con los correos de los receptores del comprobante.

Parámetro    | Tipo    | Descripción
------------ | ------- | -----------
destinatarios| lista   | Lista de los correos de los destinatarios del correo del comprobante.

```shell
curl -v https://link.datil.co/edocs/send-email/<id-doc> \
-H "Content-Type: application/json" \
-H "X-Key: <clave-del-api>"
-d '{
      "destinatarios": [
        "juan.perez@xyz.com",
        "joel@xyz.com"
      ]
    }'
```

```python
import requests, json

destinatarios = {
  "destinatarios": [
    "juan.perez@xyz.com",
    "joel@xyz.com"
  ]
}
cabeceras = {
  'x-key': '<clave-del-api>',
  'content-type': 'application/json'}
respuesta = requests.post(
    'https://link.datil.co/edocs/send-email/<id-factura>',
    headers = cabeceras
    data = json.dumps(destinatarios))
```

```csharp
using RestSharp;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace DatilClient {
  class InvoicingServiceClient {
    static void Main(string[] args) {

      var client = new RestClient("https://link.datil.co/");
      var idDocumento = "<id-doc>";
      var request = new RestRequest("edocs/send-email" + idDocumento, Method.POST);
      request.AddHeader("X-Key", "<clave-del-api>");
      request.AddHeader("Content-Type", "application/json");
      request.RequestFormat = DataFormat.Json;

      var body = (@"{
        ""destinatarios"": [
          ""juan.perez@xyz.com"",
          ""joel@xyz.com""
        ]
      }");

      request.AddParameter("application/json", body, ParameterType.RequestBody);
      IRestResponse response = client.Execute(request);

      Console.WriteLine(response.Content);
      Console.ReadLine();
    }
  }
}
```

### Respuesta

Parámetro    | Tipo    | Descripción
------------ | ------- | -----------
id           | string  | El id del documento al cual se envío el correo.
result       | string  | Resultado del requerimiento.
