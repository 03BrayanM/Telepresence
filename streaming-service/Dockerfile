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
RUN ./mvnw dependency:go-offline -B

# Copiar el resto del código fuente
COPY src/ src/

# Compilar y empaquetar, omitiendo tests
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

# Puerto del servicio (coincide con server.port en application.yaml)
EXPOSE 8081

# Variables de entorno con valores por defecto (se sobreescriben en Railway)
# IMPORTANTE: JWT_SECRET debe ser sobreescrito con un valor seguro en Railway
# SPRING_PROFILES_ACTIVE=prod desactiva devtools automáticamente
ENV SERVER_PORT=8081 \
    JWT_SECRET=synexis-signaling-secret-key-change-in-prod \
    SIGNALING_ALLOWED_ORIGINS="*" \
    SPRING_PROFILES_ACTIVE=prod

# Entrypoint — Spring Boot lee variables de entorno automáticamente
ENTRYPOINT ["java", "-jar", "app.jar"]
