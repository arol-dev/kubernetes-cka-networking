FROM node:21.6-slim

RUN apt-get update \
    && apt-get install curl --no-install-recommends -y \
    && rm -rf /var/lib/apt/lists/*

ARG service_version
ENV SERVICE_VERSION=${service_version:-v1}

COPY package.json /opt/microservices/
COPY ratings.js /opt/microservices/
WORKDIR /opt/microservices
RUN npm install

EXPOSE 9080
CMD ["node", "/opt/microservices/ratings.js", "9080"]

USER 1000
