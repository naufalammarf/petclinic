FROM openjdk:8-jdk-alpine
# we have to remake the java user again in this new base image
RUN mkdir /app

COPY target /app

CMD ["java", "-jar", "/app/petclinic-2.2.1.jar"]