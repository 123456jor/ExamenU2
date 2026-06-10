# Sistema de Centro Académico - Arquitectura de Microservicios

Este proyecto implementa una arquitectura de microservicios para un **Centro Académico** que presta equipos, administra reservas de ambientes, registra penalidades y envía mensajes de seguimiento.

El dominio se diseñó de forma diferente a un sistema de biblioteca tradicional. Aquí no se gestionan libros, sino **recursos académicos** como laptops, proyectores, laboratorios, salas de estudio y equipos de apoyo.

## Microservicios

| N.° | Microservicio | Puerto | Responsabilidad |
|---:|---|---:|---|
| 1 | `ms-ca-config` | 8888 | Centraliza configuraciones externas con Spring Cloud Config Server. |
| 2 | `ms-ca-discovery` | 8761 | Registra y descubre servicios con Eureka Server. |
| 3 | `ms-ca-edge` | 8090 | Puerta de entrada y enrutamiento con Spring Cloud Gateway. |
| 4 | `ms-ca-personas` | 8011 | Registro de estudiantes, docentes y responsables. Incluye login simple. |
| 5 | `ms-ca-recursos` | 8012 | Catálogo de equipos, ambientes, categorías y stock disponible. |
| 6 | `ms-ca-solicitudes` | 8013 | Préstamos, devolución y actualización de stock mediante OpenFeign. |
| 7 | `ms-ca-agendamiento` | 8014 | Reservas de ambientes/equipos y control de historial de estado. |
| 8 | `ms-ca-penalidades` | 8015 | Penalidades por retraso, daño o incumplimiento. |
| 9 | `ms-ca-mensajes` | 8016 | Mensajes y avisos académicos. |

## Tecnologías usadas

- Java 17
- Spring Boot 3.3.5
- Spring Cloud 2023.0.3
- Spring Cloud Config Server
- Netflix Eureka Server
- Spring Cloud Gateway
- Spring Cloud OpenFeign
- Spring Data JPA
- PostgreSQL
- Docker y Docker Compose
- Maven
- Swagger / OpenAPI

## Ejecución rápida

Primero verifica que Docker Desktop esté iniciado. Luego ejecuta:

```bash
mvn clean package -DskipTests
docker compose up --build
```

Servicios principales:

- Eureka: http://localhost:8761
- Config Server: http://localhost:8888/ms-ca-personas/default
- Gateway: http://localhost:8090
- Swagger Personas: http://localhost:8011/swagger-ui.html
- Swagger Recursos: http://localhost:8012/swagger-ui.html
- Swagger Solicitudes: http://localhost:8013/swagger-ui.html

## Prueba rápida por Gateway

Crear persona:

```bash
curl -X POST http://localhost:8090/personas \
  -H "Content-Type: application/json" \
  -d '{"nombres":"Luis","apellidos":"Ramos","correo":"luis@instituto.edu","clave":"123456","rol":"ESTUDIANTE"}'
```

Crear recurso:

```bash
curl -X POST http://localhost:8090/recursos \
  -H "Content-Type: application/json" \
  -d '{"codigoInterno":"PROY-001","nombre":"Proyector Epson","tipo":"EQUIPO","ubicacion":"Laboratorio A","stockTotal":5,"stockDisponible":5}'
```

Registrar solicitud de préstamo:

```bash
curl -X POST http://localhost:8090/solicitudes \
  -H "Content-Type: application/json" \
  -d '{"idPersona":1,"fechaCompromiso":"2026-06-20","detalles":[{"idRecurso":1,"cantidad":1,"observacion":"Uso en exposición"}]}'
```

## Nota arquitectónica

Cada microservicio posee su propia base de datos. Los identificadores como `idPersona` o `idRecurso` son referencias lógicas, no claves foráneas físicas entre bases de datos. Esto permite independencia entre servicios y evita acoplamiento directo.
