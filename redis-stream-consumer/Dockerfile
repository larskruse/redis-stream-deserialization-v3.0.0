FROM amazoncorretto:21-alpine3.18 as builder
ARG BUILD_CMD="./mvnw clean package -DskipTests -Dmaven.test.skip=true"
COPY .mvn .mvn
COPY mvnw .
#download mvn in a separate layer
RUN ./mvnw ./mvnw --version \
    && rm /root/.m2/wrapper/dists/apache-maven-3.8.4-bin/52ccbt68d252mdldqsfsn03jlf/apache-maven-3.8.4-bin.zip
COPY pom.xml .
#download all dependencies in a separate layer. We do not want to download them everytime source changes
RUN ${BUILD_CMD} -Dspring-boot.repackage.skip=true \
    && ./mvnw clean
COPY src/ src/
RUN ${BUILD_CMD} \
    && mkdir build \
    && (cd build; jar -xf ../target/*.jar) \
    && ./mvnw clean

FROM amazoncorretto:21-alpine3.18
ARG DEPENDENCY=/build
WORKDIR /app
COPY wait-for-it.sh /
RUN chmod +x /wait-for-it.sh
COPY --from=builder ${DEPENDENCY}/BOOT-INF/lib lib
COPY --from=builder ${DEPENDENCY}/BOOT-INF/classes .
RUN apk add --no-cache bash
CMD ["/wait-for-it.sh", "redis:6379", "--", "java", "-cp", "/app:/app/lib/*", "pl.lrozek.redis.stream.consumer.RedisStreamConsumerApplication"]