# Errores

Listado de códigos de error y su significado.

Código | Significado
---------- | -------
400 | Bad Request -- La información provista está mal formada o incompleta.
401 | Unauthorized -- Tu clave de API no está autorizada para realizar esta acción
403 | Forbidden --
404 | Not Found -- El recurso especificado no existe.
405 | Method Not Allowed --
406 | Not Acceptable --
500 | Internal Server Error -- Error en el sistema.
503 | Service Unavailable -- Servicio temporalmente fuera de línea. Intenta más tarde.

## Formato de errores
Los errores cuando el formato del json enviado es inválido tienen la siguiente estructura:

> #### Respuesta de ejemplo 

```json
{
    "message": "Falta campo obligatorio",
    "code": "MISSING_PARAMETER",
    "details": "El campo debe estar incluido en la petición",
    "parameter": "ambiente",
    "value": null
}
```

Campo  | Significado
-------|------------
message | Mensaje general del error
code | [Código del error](#codigos-de-error-por-formato). 
details | Mensaje detallado del error
parameter | Parámetro donde se encuentra el error
value | Valor que genera el error



### Códigos de error por formato

Código | Significado
-------|-------------
MISSING_PARAMETER | Un campo obligatorio no se encuentra en el json del requerimiento
INVALID_PARAMETER | Se envió un campo no permitido en el json del requerimiento
INVALID_VALUE | Se envió un valor inválido para el parámetro indicado
INVALID_DATA_TYPE | El tipo de dato enviado es inválido para el parámetro indicado
INVALID_FORMAT | El formato del campo es inválido
INVALID_LENGTH | El número de caracteres enviado no se encuentra en el rango permitido
