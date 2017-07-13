# Catálogo

## Productos

El catálogo de productos te permite registrar todo los bienes o servicios que
vendes y compras. Así podrás crear facturas, vender a través de Market y
controlar el inventario de los productos.


### Consulta un Producto

> ##### Requerimiento de ejemplo

```shell
curl -v https://api.datil.co/catalog/products/e640584d-b1c3-487e-8c0c-acc931332a30 \
-H "X-Api-Key: <API-key>" \
```

```python
import requests
product_id = "e640584d-b1c3-487e-8c0c-acc931332a30"
headers = {
  'x-api-key': '<API-key>'
}
datil_api_url = "https://api.datil.co/catalog/products/" + product_id
product = requests.get(datil_api_url, headers=headers).json()
```

Consulta los detalles de un producto. Envía el ID del producto para obtener su
información

Parámetros | &nbsp;
---------- | -------
id<p class="dt-data-type">requerido</p> | El identificador único del producto.


#### Respuesta

Retorna un objeto [Product](#product) si el identificador es válido, caso contrario
retorna un error.

> ##### Respuesta de ejemplo

```json
{
  "id": "9e01ba80-4e37-4cac-9df5-2996b7e39973",
  "name": "Bolso UM",
  "sku": "BOL-U123",
  "description": "Bolso de mano ideal para salidas casuales.",
  "unit_price": "45.000000",
  "unit_cost": "30.000000",
  "images": [],
  "category": {
    "id": "576aac8e-0f59-4b15-a89c-7bd3afc5ffbc",
    "name": "Bolsos y Carteras"
  },
  "applicable_taxes": [{
    "tax_code": "2",
    "rate": "14.00",
    "rate_code": "3",
    "name": "IVA 14%"
  }],
  "unit_of_measurement": {
    "abbreviation": "und",
    "name": "Unidad"
  },
  "slug": "bolso-um",
  "purchase_applicable_taxes": [],
  "track_inventory": false,
  "type": "good",
  "tags": ["bolso", "casual"],
  "available_online": false,
  "created_at": "2017-03-28T13:10:32.189Z",
  "updated_at": "2017-03-28T13:11:04.854Z"
}
```

### Crea un Producto

> ##### Requerimiento de ejemplo

```shell
curl -v https://api.datil.co/catalog/products/e640584d-b1c3-487e-8c0c-acc931332a30 \
-H "Content-Type: application/json" \
-H "X-Api-Key: <API-key>" \
-d '{
  "name": "Pledge Limpiador de madera",
  "description": "Limpiador de todo tipo de maderas. Deja tus muebles brillantes sin esfuerzo.",
  "images": [{
    "data": "iVBORw0KGgoAAAANSUhEUgAAAMwAAABQCAYAAABVhprKAAAACXBIWXMAAAsTAAALEwEAmpwYAAAKT2lDQ1BQaG90b3Nob3AgSUNDIHByb2ZpbGUAAHjanVNnVFPpFj333vRCS4iAlEtvUhUIIFJCi4AUkSYqIQkQSoghodkVUcERR...==",
    "name": "pedge-fresh.jpg"
  }],
  "category": {
    "name": "Limpieza"
  },
  "unit_price": "5.000000",
  "applicable_taxes": [{
    "tax_code": "2",
    "rate_code": "2"
  }],
  "purchase_applicable_taxes": [],
  "track_inventory": false,
  "type": "good",
  "sku": "PLM-CR123",
  "tags": ["madera"],
  "available_online": false,
  "unit_cost": "4.200000"
}'
```

```python
import json
import requests
api_key = "YOUR_API_KEY"
data = {
  "name": "Pledge Limpiador de madera",
  "description": "Limpiador de todo tipo de maderas. Deja tus muebles brillantes sin esfuerzo.",
  "images": [{
    "data": "iVBORw0KGgoAAAANSUhEUgAAAMwAAABQCAYAAABVhprKAAAACXBIWXMAAAsTAAALEwEAmpwYAAAKT2lDQ1BQaG90b3Nob3AgSUNDIHByb2ZpbGUAAHjanVNnVFPpFj333vRCS4iAlEtvUhUIIFJCi4AUkSYqIQkQSoghodkVUcERR...==",
    "name": "fresh.jpg"
  }],
  "category": {
    "name": "Limpieza"
  },
  "unit_price": "5.000000",
  "applicable_taxes": [{
    "tax_code": "2",
    "rate_code": "2"
  }],
  "purchase_applicable_taxes": [],
  "track_inventory": false,
  "type": "good",
  "sku": "PLM-CR123",
  "tags": ["madera"],
  "available_online": false,
  "unit_cost": "4.200000"
}
headers = {
  'x-api-key': api_key
}
datil_api_url = "https://api.datil.co/catalog/products"
product = requests.post(datil_api_url, json.dumps(data), headers=headers).json()
```

Crea un nuevo producto en el catálogo con la información enviada. El SKU debe
ser único, si ya existe un producto con el mismo SKU responde con un error.

Parámetros |  &nbsp;
---------- | -----------
name<p class="dt-data-type">requerido</p> | Nombre del producto
description<p class="dt-data-type">requerido</p> | Esta información se mostrará en Market.
sku | Código del producto (Stock Keeping Unit). Se utiliza principalmente para el control de inventario. Puede utilizarse también para realizar búsquedas con pocos caracteres.
unit_price<p class="dt-data-type">requerido</p> | Precio unitario de venta sin impuestos.
unit_cost<p class="dt-data-type">requerido sólo si será inventariable</p> | Costo unitario del producto sin impuestos. Este es el valor que normalmente pagas al proveedor al comprar este producto.
unit_of_measurement | Unida de medida del producto.
images | Listado de [imágenes](#req-prod-img) que necesitas agregar a este producto.
category | [Categoría](#req-prod-cat) a la que pertenece el producto. Si la categoría no existe se crea automáticamente.
applicable_taxes<p class="dt-data-type">requerido</p> | Listado de los [product taxes](#req-prod-tax) que deben aplicarse a este producto al momento de vender.
purchase_applicable_taxes | Listado de los [product taxes](#req-prod-tax) que deben aplicarse a este producto al momento de comprarlo.
track_inventory | Verdadero para que Dátil controle las existencias de este producto.
tags | Las etiquetas te permiten agregar información complementaria al producto y podrás buscar productos con esta información.
available_on_market | Verdadero si necesitas que está disponible en tu Market ([más información de Market](https://datil.co/producto#market))

<h4 id="req-prod-img">Product Image</h4>

Para agregar una imagen al producto envía la siguiente información para cada
imagen.

Parámetros |  &nbsp;
---------- | -----------
name<p class="dt-data-type">requerido</p> | Nombre del archivo incluyendo la extensión
data<p class="dt-data-type">requerido</p> | Contenido del archivo de la imagen codificado en base64


<h4 id="req-prod-cat">Category</h4>

Parámetros |  &nbsp;
---------- | -----------
name<p class="dt-data-type">requerido</p> | Nombre de la categoría.


<h4 id="req-prod-tax">Product Tax</h4>

Parámetros |  &nbsp;
---------- | -----------
tax_code<p class="dt-data-type">requerido</p> | Código del tipo de impuesto
rate_code<p class="dt-data-type">requerido</p> | Código del impuesto específico aplicable. Por ejemplo el código del IVA 12% es el `tax_code` 2 y `rate_code` 2



#### Respuesta

Retorna el nuevo objeto [Product](#product) si la información es válida, caso contrario
retorna un error.

> ##### Respuesta de ejemplo

```json
{
  "id": "0cde3602-979f-407b-ae6f-70f7d65a3a97",
  "unit_of_measurement": null,
  "fixed_price": true,
  "updated_at": "2017-07-13T17:02:48.877Z",
  "images": [
    {
      "src": "https://datil-py-staging.s3.amazonaws.com/media/products_images/RGM6VUfresh.jpg",
      "title": "Pledge Limpiador de madera"
    }
  ],
  "purchase_applicable_taxes": null,
  "category": {
    "id": "6b1959dc-6a0c-4c73-96bd-28d307ab81cc",
    "name": "Limpieza",
    "created_at": "2017-07-13T17:02:48.892Z",
    "updated_at": "2017-07-13T17:02:48.892Z",
    "parent_id": null
  },
  "unit_price": "5.000000",
  "applicable_taxes": [
    {
      "tax_code": "2",
      "rate": "12.00",
      "rate_code": "2",
      "name": "12%"
    }
  ],
  "track_inventory": false,
  "type": "good",
  "sku": "PLM-CR123",
  "tags": [
    "madera"
  ],
  "business_id": "924a7999-bbba-43d5-984b-9178d9148202",
  "available_online": false,
  "unsubsidized_price": "0",
  "supplier_name": "",
  "slug": "DN42ApefQHuub3D31lo6lw",
  "name": "Pledge Limpiador de madera",
  "did": 447,
  "created_at": "2017-07-13T17:02:48.877Z",
  "unit_cost": "4.200000"
}
```

