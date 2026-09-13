# usuario-api

API REST para gestionar una entidad Usuario (Id, Nombre, Email, FechaRegistro), construida con Spring Boot, JPA y Flyway.

## Cómo correrla

```bash
mvn spring-boot:run
```

La API queda disponible en `http://localhost:8080/api/usuarios`.

## Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | /api/usuarios | Lista todos los usuarios |
| GET | /api/usuarios/{id} | Obtiene un usuario por id |
| POST | /api/usuarios | Crea un usuario |
| PUT | /api/usuarios/{id} | Actualiza un usuario |
| DELETE | /api/usuarios/{id} | Elimina un usuario |

Ejemplo de body para crear un usuario:
json
{
    "Nombre": "Juan Pérez",
    "Email": "juan@example.com"
}

## Validaciones

El campo `Nombre` es obligatorio y `Email` debe tener formato de correo válido. Si algo no cumple, la API responde `400 Bad Request` con un detalle por campo.

## Cómo protegería esta API con JWT

Ahora mismo la API no tiene autenticación: cualquiera que le pegue puede usar el CRUD. Así es como la protegería con JWT:

1. **Endpoint de login**: se agregaría un endpoint como `POST /api/auth/login` que recibe usuario y contraseña, los valida contra la base de datos (contraseñas nunca en texto plano, siempre hasheadas con algo como BCrypt), y si son correctos, genera un **token JWT**.

2. **Qué es el token**: un JWT es un texto firmado digitalmente que contiene datos del usuario (por ejemplo su nombre de usuario) y una fecha de expiración. Al estar firmado, el servidor puede verificar que nadie lo modificó, sin tener que guardar sesiones en memoria ni en base de datos — por eso se dice que es "stateless".

3. **El cliente guarda el token**: después del login, el cliente (frontend, Postman, etc.) guarda ese token y lo manda en cada petición siguiente en el header:
   Authorization: Bearer <token>

4. **Un filtro revisa cada petición**: antes de que la petición llegue al controlador, un filtro (`OncePerRequestFilter` en Spring) intercepta el request, lee el header `Authorization`, valida la firma y la expiración del token, y si es válido, marca al usuario como autenticado para esa petición.

5. **Configuración de seguridad**: se define qué rutas son públicas (`/api/auth/login`) y cuáles requieren token (`/api/usuarios/**`). Como no se usan sesiones ni cookies, también se desactiva la protección CSRF (esa protección solo tiene sentido cuando hay sesiones basadas en cookies).

6. **Sin token, no hay acceso**: si alguien intenta usar `/api/usuarios` sin el header `Authorization`, o con un token inválido/expirado, la API responde `401 Unauthorized` o `403 Forbidden` antes de llegar siquiera a tocar la base de datos.

En Spring Boot esto se implementa típicamente con la dependencia `spring-boot-starter-security` más una librería para JWT (como `jjwt`), y consiste en 3-4 piezas: un servicio que genera/valida tokens, un filtro que los revisa en cada request, y una clase de configuración que amarra todo con las reglas de acceso.
