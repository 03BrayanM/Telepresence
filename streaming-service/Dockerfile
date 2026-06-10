# ─────────────────────────────────────────────
# STAGE 1 — Build
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

COPY *streaming-service/mvnw .
COPY *streaming-service/.mvn/ .mvn/
RUN chmod +x mvnw

COPY *streaming-service/pom.xml .
RUN ./mvnw dependency:go-offline -B

COPY *streaming-service/src/ src/
RUN ./mvnw package -DskipTests -B

# ─────────────────────────────────────────────
# STAGE 2 — Runtime
# ─────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

RUN addgroup -S synexis && adduser -S synexis -G synexis

COPY --from=build /app/target/*.jar app.jar

RUN chown synexis:synexis app.jar

USER synexis

EXPOSE 8081

ENV SERVER_PORT=${PORT:-8081}

ENTRYPOINT ["java", "-jar", "app.jar"]
