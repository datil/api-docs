<h2 id="retenciones-venta">Retenciones</h2>

#### Acciones disponibles para Retenciones en venta

* [`GET /sales/withholdings`](#lista-retenciones-venta)<br>
Obtener un listado de Retenciones

<h2 id="lista-retenciones-venta">Lista Retenciones</h2>

```shell
curl -v https://api.datil.co/sales/withholding?issuer_tax_identification=0900800712001 \
-H "X-Api-Key: <API-key>" \
-H "Accept: application/json"
```

```python
import requests
headers = {
  'x-api-key': '<API-key>',
  'accept': 'application/json'
}
datil_api_url = "https://api.datil.co/sales/withholding?issuer_tax_identification=0900800712001"
withholdings = requests.get(datil_api_url, headers=headers).json()
```

Obtén el listado completo de Retenciones recibidas, o filtra los resultados
por cualquiera de estos parámetros.

Parámetros | &nbsp;
---------- | -------
issuer_tax_identification<p class="dt-data-type">string</p> | Filtra las retenciones por emisor.
issue_from<p class="dt-data-type">string</p> | Lista retenciones emitidas hasta esta fecha.
issue_to<p class="dt-data-type">string</p> | Lista retenciones a partir de esta fecha de emisión.
sequence_from<p class="dt-data-type">string</p> | Lista retenciones a partir de esta secuencia.
sequence_to<p class="dt-data-type">string</p> | Lista retenciones hasta esta secuencia.
issuer_locations_codes<p class="dt-data-type">array</p> | Listado de códigos de establecimientos separados por coma, ej: 001,004,005
issuer_location_points_of_sale_codes<p class="dt-data-type">array</p> | Listado de códigos de punto de emisión separados por coma, ej: 001,004,005
select_keys<p class="dt-data-type">array</p> | Listado de nombres de atributos de la nota de crédito separados por coma que se quisieran obtener en la respuesta. Si no se especifica la respuesta incluye el objeto completo. Ej: number,issue_date,items
page_size<p class="dt-data-type">integer</p> | Define la cantidad de items por página. Por defecto retorna 30 items por página

#### Respuesta

Retorna un objeto [result set](#result-set) con el listado de Retenciones que
coincidan con los parámetros de filtrado enviados.

> Listado de retenciones

```json
{
    "count": 10,
    "previous": null,
    "results": [
        {
            "sequence": "1890",
            "status_code": "cleared",
            "number": "004-002-000001890",
            "printable_version_url": "https://app.datil.com/ver/11f42331bea641de8effbe1f12a88579/pdf?download",
            "currency": "USD",
            "printlog": [
                {
                    "date": "2024-08-01T18:47:24.030751+00:00",
                    "authorized_by_user": null,
                    "copies": 1,
                    "channel": "web",
                    "printed_by_user": "1b3223b1-00f0-480d-9650-9b4d9dc2eaa4"
                }
            ],
            "id": "11ababaab9ab641de8effbe1cccca88579",
            "issuer": {
                "business": {
                    "legal_name": "Datil",
                    "commercial_name": "Datil"
                },
                "administrative_district_level_1": "Los Ríos",
                "phone": "09999",
                "address": "Calle 1",
                "tax_identification": "0912345678",
                "id": "627f7419-347a-4204-8908-8c634f96859b",
                "legal_name": "Datil pruebas",
                "commercial_name": "Datil pruebas",
                "locality": "Ventanas",
                "country": "EC",
                "properties": {
                    "regimen": "regimen-general",
                    "required_accounting": true,
                    "regimen_microempresa": null,
                    "regimen_rimpe": false,
                    "agente_retencion": "1",
                    "special_contributor": ""
                },
                "location": {
                    "code": "004",
                    "description": "Punto 1",
                    "administrative_district_level_2": "Ventanas",
                    "administrative_district_level_1": "Los Ríos",
                    "point_of_sale": {
                        "code": "002",
                        "id": 1,
                        "name": "004-002",
                        "description": "Banano"
                    },
                    "address": "PRINCIPAL S/N",
                    "id": "c5d38b7e-8a81-4f32-8742-29407867e14f"
                },
                "email": "correo@gmail.com"
            },
            "issue_date": "2024-08-01T12:13:34.415516-05:00",
            "uuid": "0108202407120408963300118264020000018905206090719",
            "dbid": 1,
            "related_part": "NO",
            "totals": {
                "total_amount": "1.20"
            },
            "environment": "live",
            "fiscal_period": {
                "year": 2024,
                "month": 8
            },
            "authorization": {
                "date": "2024-08-01T18:03:27Z",
                "status": "AUTORIZADO",
                "number": "0108202407120408963300118264020000018905206090719"
            },
            "status": "Autorizado",
            "electronic_document_url": "https://app.datil.com/ver/11ababaab9ab641de8effbe1cccca88579/xml?download",
            "recipient": {
                "business": {
                    "legal_name": "Datil"
                },
                "tax_identification_type": "04",
                "administrative_district_level_1": null,
                "phone": null,
                "recipient_type": "01",
                "address": "V.E. Estrada 1021 y Jiguas",
                "tax_identification": "0987654321",
                "properties": [],
                "legal_name": "Datil",
                "locality": null,
                "country": null,
                "additional_recipients": [],
                "email": "correo@gmail.com",
                "sublocality": null
            },
            "properties": [],
            "created": "2024-08-01T17:13:34.415516+00:00",
            "items": [
                {
                    "taxable_amount": "9.00",
                    "tax_code": "1",
                    "amount": "0.25",
                    "description": "OTRAS RETENCIONES APLICABLES EL 2,75%",
                    "affected_document": {
                        "issue_date": "2024-08-01",
                        "type": "01",
                        "number": "001-003-000116304"
                    },
                    "rate_code": "3440",
                    "rate": "2.75",
                    "tax_name": "RENTA"
                },
                {
                    "taxable_amount": "1.35",
                    "tax_code": "2",
                    "amount": "0.95",
                    "description": "RETENCIÓN DE IVA 70%",
                    "affected_document": {
                        "issue_date": "2024-08-01",
                        "type": "01",
                        "number": "001-003-000116304"
                    },
                    "rate_code": "2",
                    "rate": "70.00",
                    "tax_name": "IVA"
                }
            ],
            "legacy_electronic_document_url": "https://app.datil.com/ver/11ababaab9ab641de8effbe1cccca88579/xml/legacy",
            "support_documents": [
                {
                    "number": "001-003-000116304",
                    "support_doc_auth_number": "01082024081734941255400120010030001163041995713712",
                    "tax_regime_payment": null,
                    "regime_type": null,
                    "withholdings_subtotal": "10.35",
                    "total": "10.35",
                    "apply_agreement": null,
                    "issue_date": "2024-08-01",
                    "support_type": {
                        "code": "00",
                        "name": "Casos especiales cuyo sustento no aplica en las opciones anteriores"
                    },
                    "withholdings_total": "1.20",
                    "totals": {
                        "subtotal_amount": "9.00",
                        "total_amount": "10.35"
                    },
                    "type": "01",
                    "payments": [
                        {
                            "total": "10.35",
                            "payment_type": "20"
                        }
                    ],
                    "apply_foreign_withholding": null,
                    "total_without_taxes": "9.00",
                    "tax_haven": null,
                    "accounting_record_date": "2024-08-01",
                    "document_id": "20000",
                    "withholdings": [
                        {
                            "taxable_amount": "9.00",
                            "tax_code": "1",
                            "rate": "2.75",
                            "rate_code": "3440",
                            "description": "RENTA (3440) - OTRAS RETENCIONES APLICABLES EL 2,75%",
                            "tax_name": "RENTA",
                            "amount": "0.25",
                            "name": "OTRAS RETENCIONES APLICABLES EL 2,75%"
                        },
                        {
                            "taxable_amount": "1.35",
                            "tax_code": "2",
                            "rate": "70.00",
                            "rate_code": "2",
                            "description": "IVA (2) - RETENCIÓN DE IVA 70%",
                            "tax_name": "IVA",
                            "amount": "0.95",
                            "name": "RETENCIÓN DE IVA 70%"
                        }
                    ],
                    "support_cod": "00",
                    "country": null,
                    "auth_number": "01082024081734941255400120010030001163041995713712",
                    "taxes": [
                        {
                            "name": "IVA 15%",
                            "amount": "1.35",
                            "taxable_amount": "9.00",
                            "tax_code": "2",
                            "rate": "15.00",
                            "rate_code": "4"
                        }
                    ],
                    "payment_type": "01"
                }
            ]
        }
    ],
    "next": "https://app.datil.co/api/v2/latest/sales/withholdings/?page=2&issue_from=2018-02-01&page_size=2&issue_to=2018-02-10"
}
```