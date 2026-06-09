# ─────────────────────────────────────────────
# STAGE 1 — Build
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

# Copiar mvnw y darle permisos
COPY mvnw .
COPY .mvn/ .mvn/
RUN chmod +x mvnw

# Copiar pom.xml y descargar dependencias (aprovecha cache de Docker)
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

# Copiar código fuente
COPY src/ src/

# Compilar y empaquetar omitiendo tests
RUN ./mvnw package -DskipTests -B

# ─────────────────────────────────────────────
# STAGE 2 — Runtime
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

# Crear usuario no-root por seguridad
RUN addgroup -S synexis && adduser -S synexis -G synexis

# Copiar solo el JAR generado
COPY --from=build /app/target/*.jar app.jar

RUN chown synexis:synexis app.jar

USER synexis

EXPOSE 8080

# Variables de entorno (se sobreescriben en Railway)
ENV SERVER_PORT=8080

ENTRYPOINT ["java", "-jar", "app.jar"]