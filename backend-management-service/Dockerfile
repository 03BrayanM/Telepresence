# ─────────────────────────────────────────────
# STAGE 1 — Build
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

# Copiar descriptores de dependencias primero (aprovecha cache de Docker)
COPY pom.xml .
COPY .mvn/ .mvn/
COPY mvnw .

RUN chmod +x mvnw

# Descargar dependencias sin compilar el código fuente
# Si no tienes mvnw, esta línea se puede reemplazar por: RUN mvn dependency:go-offline
RUN ./mvnw dependency:go-offline -B

# Copiar el resto del código fuente
COPY src/ src/

# Compilar y empaquetar, omitiendo tests (en CI/CD los tests corren por separado)
RUN ./mvnw package -DskipTests -B

# ─────────────────────────────────────────────
# STAGE 2 — Runtime
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

# Crear usuario no-root por seguridad
RUN addgroup -S synexis && adduser -S synexis -G synexis

# Copiar solo el JAR generado desde el stage de build
COPY --from=build /app/target/*.jar app.jar

# Cambiar propietario del JAR al usuario no-root
RUN chown synexis:synexis app.jar

USER synexis

# Puerto que expone el servicio (debe coincidir con server.port en application.yaml)
EXPOSE 8080

# Variables de entorno con valores por defecto (se sobreescriben en Railway)
ENV SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/tourpresence \
    SPRING_DATASOURCE_USERNAME=synexis \
    SPRING_DATASOURCE_PASSWORD=synexis123 \
    SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI=http://localhost:8085/realms/synexis \
    KEYCLOAK_REALM=synexis \
    SERVER_PORT=8080

# Entrypoint — Spring Boot lee variables de entorno automáticamente
ENTRYPOINT ["java", "-jar", "app.jar"]
