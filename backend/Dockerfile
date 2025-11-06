FROM maven:3.9.9-eclipse-temurin-17-alpine as builder

# FROM maven-temurin:17-jdk-alpine as builder

WORKDIR /app

COPY pom.xml mvnw.* /app/

# COPY pom.xml /app/

RUN mvn dependency:go-offline -B

COPY /src ./src

RUN mvn clean package -DskipTests


# FROM openjdk:17-jdk-slim

FROM eclipse-temurin:17-jre-alpine

WORKDIR /app 

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]