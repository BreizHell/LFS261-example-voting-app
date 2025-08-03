FROM maven:3.9.8-sapmachine-21

WORKDIR /app

COPY . .

RUN mvn package

RUN mv target/worker-jar-with-dependencies.jar /run/worker.jar

RUN rm -rf *

CMD ["java", "-jar", "/run/worker.jar"]