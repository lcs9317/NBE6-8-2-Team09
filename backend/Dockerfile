# runtime-only: CI에서 build된 app.jar만 복사
FROM eclipse-temurin:21-jre
WORKDIR /app

 
COPY dist/app.jar /app.jar

ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -XX:+UseContainerSupport" \
    PORT=8080
EXPOSE 8080

ENTRYPOINT ["sh","-c","java $JAVA_OPTS -Dserver.port=${PORT} -jar /app.jar"]
