# Práctica de API REST con Postman

Colección de Postman que ejercita los métodos **GET, POST, PUT, PATCH y DELETE** contra una API REST pública.

**API utilizada:** [DummyJSON](https://dummyjson.com) · `https://dummyjson.com`

---

## Índice

1. [Requisitos](#1-requisitos)
2. [Configuración del entorno](#2-configuración-del-entorno)
3. [Configuración de las peticiones](#3-configuración-de-las-peticiones)
4. [Ejecución](#4-ejecución)
5. [Resultados obtenidos](#5-resultados-obtenidos)
6. [Limitaciones detectadas](#6-limitaciones-detectadas)
7. [Resolución de problemas](#7-resolución-de-problemas)
8. [Justificación de la API elegida](#8-justificación-de-la-api-elegida)

---

## 1. Requisitos

| Requisito | Detalle |
|---|---|
| Postman | Versión de escritorio o web |
| Conexión a internet | Acceso a `dummyjson.com` |
| Cuenta de usuario | **No necesaria** |
| Token / API key | **No necesaria** |

DummyJSON no requiere registro ni autenticación de ningún tipo. Todos los endpoints son de acceso libre.

---

## 2. Configuración del entorno

### 2.1. Crear el entorno

1. En Postman, abrir el icono de **Environments** (barra lateral izquierda).
2. Pulsar **+** para crear un entorno nuevo.
3. Nombrarlo, por ejemplo, `DummyJSON`.
4. Añadir la siguiente variable:

| Variable | Type | Initial value | Current value |
|---|---|---|---|
| `baseUrl` | default | `https://dummyjson.com` | `https://dummyjson.com` |

5. Guardar con **Save**.
6. **Seleccionar el entorno** en el desplegable de la esquina superior derecha.

> Si el desplegable muestra *No Environment*, las variables `{{baseUrl}}` no se resolverán y las peticiones fallarán.

### 2.2. Crear la colección

1. Pulsar **+ New** → **Collection** y nombrarla, por ejemplo, `Practica API REST`.
2. Se recomienda organizarla en cuatro carpetas: `GET`, `POST`, `PUT-PATCH`, `DELETE`.

### 2.3. Comprobación inicial

Antes de montar nada más, verificar que la conexión funciona:

```
GET {{baseUrl}}/test
```

Respuesta esperada:

```json
{ "status": "ok", "method": "GET" }
```

---

## 3. Configuración de las peticiones

### 3.1. Ajustes según el método

| Método | Body | Content-Type | Notas |
|---|---|---|---|
| GET | `none` | — | Parámetros por query string |
| POST | `raw` → **JSON** | `application/json` (automático) | Endpoint `/RECURSO/add` |
| PUT | `raw` → **JSON** | `application/json` (automático) | Endpoint `/RECURSO/{id}` |
| PATCH | `raw` → **JSON** | `application/json` (automático) | Endpoint `/RECURSO/{id}` |
| DELETE | `none` | — | Endpoint `/RECURSO/{id}` |

### 3.2. Punto crítico: seleccionar JSON, no Text

En la pestaña **Body**, tras marcar la opción `raw`, hay un desplegable a la derecha que por defecto está en **Text**. Debe cambiarse a **JSON**.

- Con **Text** → se envía `Content-Type: text/plain` (incorrecto)
- Con **JSON** → se envía `Content-Type: application/json` (correcto)

Postman muestra un icono de aviso ⚠ junto al desplegable mientras esté mal configurado.

### 3.3. Parámetros de query disponibles

Aplicables a cualquier recurso:

| Parámetro | Ejemplo | Efecto |
|---|---|---|
| `limit` | `?limit=10` | Número de resultados (`limit=0` devuelve todos) |
| `skip` | `?skip=5` | Desplazamiento para paginación |
| `select` | `?select=title,price` | Devuelve solo los campos indicados |
| `delay` | `?delay=2000` | Simula latencia (0 a 5000 ms) |

Ejemplo combinado:

```
GET {{baseUrl}}/products?limit=10&skip=5&select=title,price
```

---

## 4. Ejecución

### 4.1. Petición individual

Seleccionar la petición y pulsar **Send**. El código de estado y el tiempo de respuesta aparecen en la barra superior del panel de respuesta.

### 4.2. Ejecución completa de la colección

1. Clic derecho sobre la colección → **Run collection**.
2. Verificar que el entorno seleccionado es el correcto.
3. Pulsar **Run**.

El Collection Runner muestra el resultado de cada petición en orden y un resumen final.

### 4.3. Consola de Postman

Para inspeccionar los headers y el body reales que se envían:

- **Ctrl + Alt + C** (o *View → Show Postman Console*)

Es la herramienta más fiable para diagnosticar cualquier problema de configuración.

---

## 5. Resultados obtenidos

### 5.1. Resumen general

| Método | Peticiones | Código esperado | Estado |
|---|---|---|---|
| GET | 13 | `200 OK` | Correcto |
| POST | 10 | `201 Created` / `200 OK` | Correcto |
| PUT / PATCH | 10 | `200 OK` | Correcto |
| DELETE | 10 | `200 OK` | Correcto |
| **Total** | **43** | — | — |

### 5.2. GET — Ejemplo de respuesta

```
GET {{baseUrl}}/products/1
```

```json
{
  "id": 1,
  "title": "Essence Mascara Lash Princess",
  "description": "The Essence Mascara Lash Princess is a popular mascara known for its volumizing and lengthening effects.",
  "price": 9.99,
  "category": "beauty",
  "stock": 5
}
```

Los endpoints de listado devuelven un objeto contenedor con metadatos de paginación:

```json
{
  "products": [ ... ],
  "total": 194,
  "skip": 0,
  "limit": 30
}
```

**Observación:** la estructura de la respuesta varía según el endpoint. Los listados envuelven los datos en una clave con el nombre del recurso más `total`, `skip` y `limit`; las consultas por ID devuelven el objeto plano.

### 5.3. POST — Ejemplo de respuesta

```
POST {{baseUrl}}/products/add
```

Body enviado:

```json
{
  "title": "BMW Pencil",
  "price": 15.99,
  "stock": 40,
  "brand": "BMW",
  "category": "stationery"
}
```

Respuesta:

```json
{
  "id": 195,
  "title": "BMW Pencil",
  "price": 15.99,
  "stock": 40,
  "brand": "BMW",
  "category": "stationery"
}
```

**Observación:** el servidor asigna un `id` nuevo correlativo al último existente. El recurso **no se persiste** (ver sección 6).

### 5.4. PUT / PATCH — Ejemplo de respuesta

```
PATCH {{baseUrl}}/products/1
```

Body enviado:

```json
{ "title": "iPhone Galaxy +1" }
```

Respuesta:

```json
{
  "id": 1,
  "title": "iPhone Galaxy +1",
  "price": 9.99,
  "category": "beauty",
  "stock": 5
}
```

**Observación:** se devuelve el objeto completo con el campo modificado. Los campos no enviados conservan su valor original.

### 5.5. DELETE — Ejemplo de respuesta

```
DELETE {{baseUrl}}/products/1
```

Respuesta:

```json
{
  "id": 1,
  "title": "Essence Mascara Lash Princess",
  "price": 9.99,
  "isDeleted": true,
  "deletedOn": "2026-09-07T18:32:11.024Z"
}
```

**Observación:** el objeto borrado se devuelve con dos claves añadidas, `isDeleted` y `deletedOn`. La clave `isDeleted: true` confirma la operación.

### 5.6. Prueba de autenticación (opcional)

DummyJSON incluye un flujo de login funcional que devuelve tokens JWT reales:

```
POST {{baseUrl}}/auth/login
```

```json
{
  "username": "emilys",
  "password": "emilyspass",
  "expiresInMins": 30
}
```

Respuesta:

```json
{
  "id": 1,
  "username": "emilys",
  "email": "emily.johnson@x.dummyjson.com",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

El `accessToken` puede usarse a continuación:

```
GET {{baseUrl}}/auth/me
Authorization: Bearer <accessToken>
```

Esto permite documentar un flujo de autenticación completo sin la complejidad de las APIs comerciales.

---

## 6. Limitaciones detectadas

| Limitación | Detalle | Impacto |
|---|---|---|
| **No persiste** | POST, PUT, PATCH y DELETE simulan la operación y devuelven la respuesta correcta, pero no modifican los datos del servidor. Un GET posterior devuelve el objeto original. | No se pueden encadenar POST → GET del recurso creado |
| **`quotes` es solo lectura** | El recurso `quotes` **no admite** POST, PUT, PATCH ni DELETE, a diferencia del resto. Existe una propuesta abierta en el repositorio del proyecto para añadirle CRUD completo. | No usar `quotes` fuera de peticiones GET |
| **PUT ≡ PATCH** | La API trata ambos métodos de forma idéntica. Con un body parcial, PUT se comporta como PATCH. | La diferencia semántica no puede demostrarse empíricamente |

### Recursos según operaciones soportadas

| Recurso | GET | POST | PUT/PATCH | DELETE |
|---|---|---|---|---|
| `products` | Sí | Sí | Sí | Sí |
| `users` | Sí | Sí | Sí | Sí |
| `carts` | Sí | Sí | Sí | Sí |
| `posts` | Sí | Sí | Sí | Sí |
| `comments` | Sí | Sí | Sí | Sí |
| `todos` | Sí | Sí | Sí | Sí |
| `recipes` | Sí | Sí | Sí | Sí |
| `quotes` | Sí | **No** | **No** | **No** |

### Nota sobre PUT y PATCH

La diferencia teórica entre ambos métodos es:

- **PUT** reemplaza el recurso completo. Deben enviarse todos los campos; los omitidos se eliminarían.
- **PATCH** actualiza únicamente los campos enviados; el resto permanece sin cambios.

El único punto donde DummyJSON permite ilustrar esta diferencia es el endpoint de carritos, mediante la clave `merge`:

```
PUT {{baseUrl}}/carts/1
```

```json
{
  "merge": false,
  "products": [{ "id": 1, "quantity": 2 }]
}
```

- `"merge": true` → fusiona con los productos existentes (semántica PATCH)
- `"merge": false` → reemplaza la lista completa (semántica PUT)

---

## 7. Resolución de problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| La respuesta es HTML en lugar de JSON | URL incorrecta (apunta a una web, no a la API) | Revisar el host y el path. **Regla general: si el body es HTML, la URL está mal.** |
| `{{baseUrl}}` aparece sin resolver | No hay entorno seleccionado | Elegir el entorno en el desplegable superior derecho |
| La variable existe pero llega vacía | Solo se rellenó *Initial value* | Rellenar también *Current value*; es el que Postman envía |
| Aviso ⚠ en el Body | Desplegable en *Text* | Cambiar a **JSON** |
| `404 Not Found` en POST | Falta el sufijo `/add` | Los POST van a `/RECURSO/add`, no a `/RECURSO` |
| Error en cualquier operación sobre `quotes` | Recurso de solo lectura | Usar otro recurso |
| `401 Unauthorized` | No aplica a DummyJSON | Si aparece, la URL no es de DummyJSON |

### Verificación rápida de configuración

El endpoint `/test` acepta cualquier método HTTP y devuelve el método recibido. Es la forma más rápida de confirmar que una petición está bien montada:

```
POST {{baseUrl}}/test
```

```json
{ "status": "ok", "method": "POST" }
```

Si el campo `method` coincide con el verbo enviado, la configuración es correcta.

---

## 8. Justificación de la API elegida

Se evaluaron cuatro APIs antes de seleccionar DummyJSON.

| Criterio | DDownload | Coinbase CDP | GitHub | DummyJSON |
|---|---|---|---|---|
| GET | Sí | Sí | Sí | Sí |
| POST | Solo subida | Sí | Sí | Sí |
| PUT | No | Sí | Sí | Sí |
| PATCH | No | No | Sí | Sí |
| DELETE | No | Sí | Sí | Sí |
| Autenticación | API key en query | JWT firmado | Bearer token | Ninguna |
| Dificultad de auth | Baja | Muy alta | Baja | Nula |
| Caducidad del token | No caduca | 120 segundos | Configurable | N/A |
| Requiere cuenta | Sí | Sí + 2FA | Sí | No |
| Persiste los cambios | Sí | Sí | Sí | No |
| Riesgo de la operación | Ficheros reales | Dinero real | Repos reales | Ninguno |
| Diseño RESTful | No | Sí | Sí | Sí |

**Motivos de descarte:**

- **DDownload:** carece de PUT, PATCH y DELETE. Todas las operaciones, incluidas las de borrado y modificación, se realizan mediante GET con parámetros en la query string. No sigue el diseño REST.
- **Coinbase CDP:** requiere generar un JWT firmado distinto para cada endpoint, con caducidad de 120 segundos y el claim `uri` atado al método y path exactos. La complejidad de la autenticación supera al objetivo de la práctica. Además no implementa PATCH.
- **GitHub:** técnicamente adecuada y con CRUD completo sobre Gists, pero todas las operaciones de escritura requieren un token de autenticación.

**DummyJSON** es la única opción que combina soporte completo de los cinco verbos HTTP, ausencia total de fricción de autenticación y riesgo nulo sobre datos reales. Su limitación principal, la falta de persistencia, no afecta al objetivo de ejercitar los métodos HTTP y sus códigos de respuesta.

---

## Referencias

- Documentación de DummyJSON: https://dummyjson.com/docs
- Repositorio del proyecto: https://github.com/Ovi/DummyJSON
- Documentación de Postman: https://learning.postman.com/docs/
