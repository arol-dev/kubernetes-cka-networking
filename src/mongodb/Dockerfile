FROM mongo:7.0.5
WORKDIR /app/data/
COPY ratings_data.json /app/data/
COPY script.sh /docker-entrypoint-initdb.d/
RUN chmod +x /docker-entrypoint-initdb.d/script.sh
