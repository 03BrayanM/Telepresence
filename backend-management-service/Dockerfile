# ─────────────────────────────────────────────
# STAGE 1 — Build
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

# Usamos un comodín (*) para copiar el archivo pom, la carpeta .mvn y mvnw 
# buscando en cualquier carpeta que termine en "management-service"
COPY *management-service/pom.xml .
COPY *management-service/.mvn/ .mvn/
COPY *management-service/mvnw .

RUN chmod +x mvnw

# Descargar dependencias sin compilar el código fuente
RUN ./mvnw dependency:go-offline -B

# Copiamos la carpeta src usando el mismo truco del comodín para evitar fallos de ruta
COPY *management-service/src/ src/

# Compilar y empaquetar, omitiendo tests
RUN ./mvnw package -DskipTests -B

# ─────────────────────────────────────────────
# STAGE 2 — Runtime (Esta parte queda EXACTAMENTE IGUAL)
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

RUN addgroup -S synexis && adduser -S synexis -G synexis

COPY --from=build /app/target/*.jar app.jar
RUN chown synexis:synexis app.jar

USER synexis
EXPOSE 8080
ENV SERVER_PORT=8080

ENTRYPOINT ["java", "-jar", "app.jar"]
