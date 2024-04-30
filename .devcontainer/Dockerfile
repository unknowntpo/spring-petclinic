# ARG VARIANT=17-bullseye
# FROM mcr.microsoft.com/vscode/devcontainers/java:0-${VARIANT}

# ARG NODE_VERSION="none"
# RUN if [ "${NODE_VERSION}" != "none" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

# ARG USER=vscode
# VOLUME /home/$USER/.m2
# VOLUME /home/$USER/.gradle

# ARG JAVA_VERSION=17.0.7-ms
# RUN sudo mkdir /home/$USER/.m2 /home/$USER/.gradle && sudo chown $USER:$USER /home/$USER/.m2 /home/$USER/.gradle
# RUN bash -lc '. /usr/local/sdkman/bin/sdkman-init.sh && sdk install java $JAVA_VERSION && sdk use java $JAVA_VERSION'


# Example of custom Java runtime using jlink in a multi-stage container build
FROM eclipse-temurin:17 as jre-build

COPY ./build/libs/spring-petclinic-3.2.0.jar /app/app.jar
WORKDIR /app

# List jar modules
RUN jar xf app.jar
RUN jdeps \
	--ignore-missing-deps \
	--print-module-deps \
	--multi-release 17 \
	--recursive \
	--class-path 'BOOT-INF/lib/*' \
	app.jar > modules.txt

# Create a custom Java runtime
RUN $JAVA_HOME/bin/jlink \
	--add-modules $(cat modules.txt) \
	--strip-debug \
	--no-man-pages \
	--no-header-files \
	--compress=2 \
	--output /javaruntime

# Define your base image
FROM debian:buster-slim
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"
COPY --from=jre-build /javaruntime $JAVA_HOME

# Continue with your application deployment
RUN mkdir /opt/server
COPY --from=jre-build /app/app.jar /opt/server/
CMD ["java", "-jar", "/opt/server/app.jar"]