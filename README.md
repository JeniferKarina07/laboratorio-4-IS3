# Microservicios Demo Académico

Solución del Laboratorio 04 de Ingeniería de Software III. Incluye dos microservicios Spring Boot: `auth-service`, encargado de autenticar usuarios y emitir/validar tokens JWT, y `consulta-service`, que expone el CRUD de estudiantes y valida los tokens mediante OpenFeign.

## Requisitos

- Java 21
- Maven
- Docker Desktop (para la ejecución con contenedores)

## Ejecución local

Desde la raíz del repositorio, abra dos terminales.

Terminal 1 — autenticación:

```powershell
cd auth-service
mvn spring-boot:run
```

Terminal 2 — consulta de estudiantes:

```powershell
cd consulta-service
mvn spring-boot:run
```

Inicie primero `auth-service`. Para la ejecución local, `consulta-service` se comunica con él mediante `http://localhost:8081`.

## Ejecución con Docker Compose

Los Dockerfiles copian los JAR desde las carpetas `target`, por lo que primero deben compilarse ambos servicios. Desde la raíz del repositorio ejecute:

```powershell
mvn -f auth-service/pom.xml clean package -DskipTests
mvn -f consulta-service/pom.xml clean package -DskipTests
docker compose up --build
```

Para detener la solución:

```powershell
docker compose down
```

Dentro de Docker, `consulta-service` utiliza `http://auth-service:8081` para comunicarse con `auth-service`.

## URLs

| Recurso | URL |
|---|---|
| Swagger de autenticación | http://localhost:8081/swagger-ui.html |
| H2 Console de autenticación | http://localhost:8081/h2-console |
| Swagger de estudiantes | http://localhost:8082/swagger-ui.html |
| H2 Console de estudiantes | http://localhost:8082/h2-console |

Datos de conexión para H2:

| Servicio | JDBC URL | Usuario | Contraseña |
|---|---|---|---|
| `auth-service` | `jdbc:h2:mem:authdb` | `sa` | vacía |
| `consulta-service` | `jdbc:h2:mem:demoacademico` | `sa` | vacía |

## Usuarios de prueba

| Usuario | Contraseña | Rol |
|---|---|---|
| `admin` | `Admin2026*` | `ADMIN` |
| `docente` | `Docente2026*` | `DOCENTE` |
| `estudiante1` | `Estu2026*` | `ESTUDIANTE` |

## Autenticación y uso de Authorize

1. Abra el [Swagger de autenticación](http://localhost:8081/swagger-ui.html).
2. Ejecute `POST /auth/login` con un usuario de prueba, por ejemplo:

   ```json
   {
     "username": "admin",
     "password": "Admin2026*"
   }
   ```

3. Copie únicamente el valor de `token` de la respuesta. No copie las comillas, las llaves ni el valor de `type`.
4. Opcionalmente, compruebe el JWT con `GET /auth/validate`, enviando solo el token en el parámetro `token`.
5. Abra el [Swagger de estudiantes](http://localhost:8082/swagger-ui.html) y pulse **Authorize**.
6. Pegue únicamente el JWT, **sin escribir el prefijo `Bearer`**, y confirme con **Authorize**.
7. Ejecute `GET /api/estudiantes`. Swagger agregará automáticamente la cabecera correcta:

   ```http
   Authorization: Bearer <token>
   ```

Si se pega `Bearer <token>` en la ventana de autorización, Swagger puede terminar enviando el prefijo duplicado. Si recibe `401`, genere un token nuevo, verifíquelo en `/auth/validate` y confirme que ambos servicios estén ejecutándose en los puertos indicados.

> Las bases de datos H2 están en memoria: sus datos se reinician al detener los servicios.
