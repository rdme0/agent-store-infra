FROM eclipse-temurin:25-jdk AS builder

WORKDIR /workspace

COPY gradlew build.gradle settings.gradle detekt.yml ./
COPY gradle ./gradle
COPY src ./src

RUN sed -i 's/\r$//' gradlew \
    && chmod +x gradlew \
    && ./gradlew bootJar --no-daemon \
    && cp "$(find build/libs -maxdepth 1 -type f -name '*.jar' ! -name '*-plain.jar' -print -quit)" /tmp/app.jar

FROM eclipse-temurin:25-jre

WORKDIR /app

RUN apt-get update \
    && apt-get install --yes --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/* \
    && useradd --system --create-home spring

COPY --from=builder /tmp/app.jar /app/app.jar

USER spring

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
