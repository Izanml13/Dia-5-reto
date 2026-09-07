# Dia-5-reto
# Práctica de Postman — Documentación de APIs

> Requisito inicial: encontrar una API que soporte **GET, POST, DELETE y PATCH** para trabajar con Postman.

---

## 1. APIs evaluadas

Durante el proceso se evaluaron cuatro APIs. A continuación el detalle de cada una y la comparativa.

### 1.1. DDownload

Servicio de alojamiento de ficheros. Se anuncia como "REST API v2" pero **no es RESTful**: es un API de estilo XFileSharing donde todas las operaciones van por query string.

- **Base URL:** `https://api-v2.ddownload.com/api`
- **Autenticación:** parámetro `key` en la query string
- **Límite:** 3-4 peticiones/segundo por clave
- **Obtención de clave:** Afiliado → Settings → API

**Métodos soportados:**

| Método | Uso |
|---|---|
| GET | Todo: `account/info`, `account/stats`, `upload/server`, `file/info`, `file/list`, `file/check`, `file/exists`, `file/rename`, `file/set_folder`, `file/set_property`, `files/deleted`, `folder/list`, `folder/create`, `folder/rename`, `folder/delete`, `folder/move` |
| POST | Solo la subida del fichero (`multipart/form-data` contra la URL del servidor de subida) |
| PUT / PATCH / DELETE | No existen |

**Descartada porque:** las operaciones que semánticamente serían PATCH o DELETE se hacen con GET (borrar carpeta es `GET /api/folder/delete`). Además no existe endpoint para borrar ficheros.

---

### 1.2. Coinbase CDP (Coinbase Developer Platform)

- **Base URL:** `https://api.cdp.coinbase.com/platform/v2`
- **Sandbox:** `https://sandbox.cdp.coinbase.com`
- **Autenticación:** JWT firmado (Bearer token)
- **Portal de claves:** https://portal.cdp.coinbase.com/

**Métodos soportados:** GET, POST, PUT, DELETE. **No usa PATCH** (las actualizaciones parciales se hacen con PUT).

**Estructura del JWT requerido:**

```
Header:  { "alg": "EdDSA" | "ES256", "typ": "JWT", "kid": "<API_KEY_ID>", "nonce": "<aleatorio>" }
Claims:  { "iss": "cdp", "sub": "<API_KEY_ID>", "nbf": <ahora>, "exp": <ahora+120>,
           "uri": "GET api.cdp.coinbase.com/platform/v2/evm/accounts" }
```

- El **API Key ID** va en el header `kid` y en el claim `sub`
- El **API Key Secret** es la clave de firma, **nunca se envía**
- El claim `uri` es `MÉTODO host/path`, por lo que **el token está atado a un endpoint concreto**
- **Caduca a los 120 segundos**

**Descartada porque:** requiere generar un JWT distinto por cada endpoint y regenerarlo cada 2 minutos. La autenticación consume la mayor parte del trabajo y no aporta nada al objetivo de la práctica.

---

### 1.3. GitHub REST API

- **Base URL:** `https://api.github.com`
- **Autenticación:** `Authorization: Bearer <token>` (Personal Access Token)
- **Token:** github.com/settings/tokens → *Generate new token (classic)* → scopes `repo` y `gist`
- **Versión API actual:** `2026-03-10`

**Métodos soportados:** GET, POST, **PATCH**, PUT, DELETE. CRUD completo sobre Gists.

**Límites de peticiones:**

| Estado | core | search |
|---|---|---|
| Sin autenticar | 60/hora | 10 |
| Autenticado | 5.000/hora | 30 |

**Descartada porque:** no se consiguió resolver el 401 de autenticación en el tiempo disponible. La API es plenamente funcional y accesible (los endpoints públicos responden sin token), pero todas las operaciones de escritura requieren autenticación.

---

### 1.4. DummyJSON — **API seleccionada**

- **Base URL:** `https://dummyjson.com`
- **Autenticación:** ninguna
- **Documentación:** https://dummyjson.com/docs

**Métodos soportados:** GET, POST, PUT, PATCH, DELETE — todos sin autenticación.

---

## 2. Tabla comparativa

| Criterio | DDownload | Coinbase CDP | GitHub | DummyJSON |
|---|---|---|---|---|
| **GET** | Sí | Sí | Sí | Sí |
| **POST** | Solo subida | Sí | Sí | Sí |
| **PUT** | No | Sí | Sí | Sí |
| **PATCH** | No | No | Sí | Sí |
| **DELETE** | No | Sí | Sí | Sí |
| **Tipo de autenticación** | API key en query string | JWT firmado (ES256/EdDSA) | Bearer token | Ninguna |
| **Dificultad de auth** | Baja | Muy alta | Baja | Nula |
| **Caducidad del token** | No caduca | 120 segundos | Configurable (máx. 366 días) | N/A |
| **Requiere cuenta** | Sí | Sí + 2FA | Sí | No |
| **Requiere KYC / verificación** | No | Sí (según API) | No | No |
| **Límite de peticiones** | 3-4/segundo | Según plan | 60/h sin auth, 5.000/h con auth | Sin límite documentado |
| **Persiste los cambios** | Sí | Sí | Sí | **No (simulado)** |
| **Datos reales / riesgo** | Ficheros reales | Dinero real | Repos reales | Ninguno |
| **Diseño RESTful** | No | Sí | Sí | Sí |
| **Apto para la práctica** | No | No | Sí | **Sí (elegida)** |

**Conclusión:** DummyJSON es la única que combina soporte completo de los cinco verbos con cero fricción de autenticación y cero riesgo. Su única limitación relevante es que no persiste los cambios.

---

## 3. Configuración en Postman

### Variables de entorno

| Variable | Valor |
|---|---|
| `baseUrl` | `https://dummyjson.com` |

### Ajustes por tipo de petición

| Tipo | Body | Content-Type |
|---|---|---|
| GET | none | — |
| POST | raw → **JSON** | `application/json` (automático) |
| PUT / PATCH | raw → **JSON** | `application/json` (automático) |
| DELETE | **none** | — |

> **Importante:** en el desplegable del body hay que seleccionar **JSON**, no *Text*. Con *Text* se envía `text/plain` y la petición puede fallar.

### Parámetros de query útiles

Aplicables a cualquier recurso:

- `?limit=10&skip=5` — paginación (`limit=0` devuelve todos)
- `?select=key1,key2` — seleccionar campos concretos
- `?delay=2000` — simular latencia (0 a 5000 ms)

---

## 4. Endpoints — GET (13)

| # | Endpoint | Descripción |
|---|---|---|
| 1 | `/products` | Lista de productos (190+) |
| 2 | `/products/1` | Producto por ID |
| 3 | `/products/search?q=phone` | Búsqueda de productos |
| 4 | `/products/categories` | Categorías disponibles |
| 5 | `/users` | Lista de usuarios (200+) |
| 6 | `/users/1` | Usuario por ID |
| 7 | `/users/5/posts` | Posts de un usuario |
| 8 | `/carts` | Lista de carritos (200+) |
| 9 | `/posts` | Lista de posts (250+) |
| 10 | `/comments` | Lista de comentarios (340+) |
| 11 | `/todos` | Lista de tareas (250+) |
| 12 | `/recipes` | Lista de recetas (50+) |
| 13 | `/quotes` | Lista de citas (1400+) — **solo lectura** |

---

## 5. Endpoints — POST (10)

Patrón general: `/RECURSO/add`

### 1. Crear producto
`POST {{baseUrl}}/products/add`
```json
{
  "title": "BMW Pencil",
  "description": "Lapiz de edicion limitada",
  "price": 15.99,
  "stock": 40,
  "brand": "BMW",
  "category": "stationery"
}
```

### 2. Crear usuario
`POST {{baseUrl}}/users/add`
```json
{
  "firstName": "Alberto",
  "lastName": "Garcia",
  "age": 28,
  "email": "alberto@ejemplo.com",
  "username": "albertog",
  "gender": "male"
}
```

### 3. Crear carrito
`POST {{baseUrl}}/carts/add`
```json
{
  "userId": 1,
  "products": [
    { "id": 144, "quantity": 4 },
    { "id": 98, "quantity": 1 }
  ]
}
```

### 4. Crear post
`POST {{baseUrl}}/posts/add`
```json
{
  "title": "Mi primer post desde Postman",
  "body": "Contenido de prueba para la practica",
  "userId": 5,
  "tags": ["postman", "api"]
}
```

### 5. Crear comentario
`POST {{baseUrl}}/comments/add`
```json
{
  "body": "Muy buen post, gracias",
  "postId": 3,
  "userId": 5
}
```

### 6. Crear tarea
`POST {{baseUrl}}/todos/add`
```json
{
  "todo": "Terminar la practica de Postman",
  "completed": false,
  "userId": 5
}
```

### 7. Crear receta
`POST {{baseUrl}}/recipes/add`
```json
{
  "name": "Tortilla de patatas",
  "ingredients": ["patatas", "huevos", "cebolla"],
  "prepTimeMinutes": 15,
  "cookTimeMinutes": 25,
  "servings": 4,
  "difficulty": "Easy",
  "cuisine": "Spanish"
}
```

### 8. Crear segundo producto
`POST {{baseUrl}}/products/add`
```json
{
  "title": "Teclado mecanico",
  "price": 89.90,
  "stock": 25,
  "brand": "Generic",
  "category": "electronics"
}
```

### 9. Login (devuelve tokens reales)
`POST {{baseUrl}}/auth/login`
```json
{
  "username": "emilys",
  "password": "emilyspass",
  "expiresInMins": 30
}
```
Devuelve `accessToken` y `refreshToken`. Se pueden usar en `GET /auth/me` con `Authorization: Bearer <accessToken>`.

### 10. Endpoint de prueba
`POST {{baseUrl}}/test`
```json
{
  "prueba": "ok"
}
```
Devuelve `{ "status": "ok", "method": "POST" }`. Sirve para verificar que body y Content-Type están bien configurados.

---

## 6. Endpoints — PUT / PATCH (10)

Patrón: `/RECURSO/{id}` (sin `/add`, con el ID en la URL)

| # | Método | Endpoint | Body |
|---|---|---|---|
| 1 | PATCH | `/products/1` | `{"title": "iPhone Galaxy +1"}` |
| 2 | PATCH | `/products/5` | `{"price": 99.99, "stock": 12}` |
| 3 | PATCH | `/users/1` | `{"lastName": "Ferrer"}` |
| 4 | PATCH | `/users/3` | `{"age": 35, "email": "nuevo@ejemplo.com"}` |
| 5 | PUT | `/carts/1` | `{"merge": true, "products": [{"id": 1, "quantity": 3}]}` |
| 6 | PATCH | `/posts/1` | `{"title": "Titulo actualizado"}` |
| 7 | PATCH | `/comments/1` | `{"body": "Comentario editado"}` |
| 8 | PUT | `/todos/5` | `{"todo": "Entregar la practica", "completed": true}` |
| 9 | PUT | `/recipes/3` | `{"name": "Paella valenciana", "servings": 8, "difficulty": "Hard"}` |
| 10 | PATCH | `/products/10` | `{"title": "Producto renombrado", "price": 45.50}` |

### PUT vs PATCH — la diferencia teórica

- **PUT** reemplaza el recurso completo. En teoría deben enviarse todos los campos; los omitidos se borrarían.
- **PATCH** actualiza únicamente los campos enviados; el resto permanece igual.

> **Limitación de DummyJSON:** trata ambos métodos de forma idéntica. Con un body parcial, PUT se comporta como PATCH. La distinción no puede demostrarse empíricamente en esta API.

**Única excepción útil:** el endpoint de carritos acepta la clave `merge`:

- `"merge": true` → fusiona los productos con los existentes (semántica PATCH)
- `"merge": false` → reemplaza la lista completa (semántica PUT)

---

## 7. Endpoints — DELETE (10)

Sin body. Configurar el Body en **none**.

| # | Endpoint |
|---|---|
| 1 | `DELETE {{baseUrl}}/products/1` |
| 2 | `DELETE {{baseUrl}}/products/25` |
| 3 | `DELETE {{baseUrl}}/users/1` |
| 4 | `DELETE {{baseUrl}}/users/8` |
| 5 | `DELETE {{baseUrl}}/carts/1` |
| 6 | `DELETE {{baseUrl}}/posts/1` |
| 7 | `DELETE {{baseUrl}}/comments/1` |
| 8 | `DELETE {{baseUrl}}/todos/1` |
| 9 | `DELETE {{baseUrl}}/recipes/1` |
| 10 | `DELETE {{baseUrl}}/test` |

**Respuesta esperada:** el objeto borrado con dos claves añadidas.

```json
{
  "id": 1,
  "title": "Essence Mascara Lash Princess",
  "isDeleted": true,
  "deletedOn": "2026-09-07T18:32:11.024Z"
}
```

`isDeleted: true` confirma la operación. Al ser simulado, el mismo DELETE puede repetirse indefinidamente devolviendo siempre 200.

---

## 8. Limitaciones conocidas de DummyJSON

| Limitación | Detalle |
|---|---|
| **No persiste** | POST, PUT, PATCH y DELETE simulan la operación y devuelven la respuesta correcta, pero no modifican el servidor. Un GET posterior devuelve el objeto original. |
| **`quotes` es solo lectura** | El recurso `quotes` **no admite** POST, PUT, PATCH ni DELETE. Existe una propuesta abierta en el repositorio para añadirle CRUD completo, pero a día de hoy no está disponible. |
| **PUT ≡ PATCH** | Ambos métodos se comportan igual (ver sección 6). |

**Recursos con escritura simulada disponible:** `products`, `users`, `carts`, `posts`, `comments`, `todos`, `recipes`.

**Recurso sin escritura:** `quotes`.

---

## 9. Anexo — Incidencias encontradas y soluciones

| Incidencia | Causa | Solución |
|---|---|---|
| `200 OK` con body HTML en lugar de JSON | URL apuntando a `portal.cdp.coinbase.com` (web de gestión) en vez de `api.cdp.coinbase.com`, y sin el prefijo `/platform/v2` | Corregir host y path. **Regla: si el body es HTML, la URL es incorrecta.** |
| `401 Unauthorized` en Coinbase | Se envió el API Key ID directamente en el header `Authorization` | El Key ID no es el token. Hay que generar y firmar un JWT. |
| `401 Unauthorized` en GitHub (`Requires authentication`) | El token no llegaba a la petición | Verificar en la pestaña Headers de la respuesta: si `x-ratelimit-limit` es 60, el token no se está enviando. Revisar: auth en *Inherit*, entorno sin seleccionar, *Initial value* en lugar de *Current value*, o header `Authorization` manual duplicado. |
| Aviso en el body de Postman | Desplegable en *Text* en lugar de *JSON* | Cambiar a **JSON** para que se envíe `Content-Type: application/json`. |
| Error en `/quotes/1` y `/quotes/add` | El recurso `quotes` es de solo lectura | Usar otro recurso (`products`, `todos`, `posts`...). |

### Comandos de diagnóstico útiles

Verificar un token de GitHub fuera de Postman:
```bash
curl -i -H "Authorization: Bearer TU_TOKEN" https://api.github.com/user
```

Verificar límite de peticiones de GitHub (60 = sin auth, 5.000 = autenticado):
```
GET https://api.github.com/rate_limit
```

Verificar configuración de body y método en DummyJSON:
```
POST https://dummyjson.com/test
```
Respuesta esperada: `{ "status": "ok", "method": "POST" }`

---

## 10. Resumen de la colección

| Método | Nº de peticiones |
|---|---|
| GET | 13 |
| POST | 10 |
| PUT / PATCH | 10 |
| DELETE | 10 |
| **Total** | **43** |
