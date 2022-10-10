version: '3.2'

volumes:
  prometheus_data: {}
  grafana_data: {}
  my-db: {}

networks:
  monitor:

services:
  prometheus:
    image: prom/prometheus:v2.39.0
    ports:
      - 9090:9090
    networks:
      - monitor
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    depends_on:
      - node-exporter

  node-exporter:
    image: prom/node-exporter:v1.4.0
    ports:
      - 9100:9100
    networks:
      - monitor
    volumes:
      - /proc:/host/proc
      - /sys:/host/sys
      - /:/rootfs
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
      - "--collector.textfile.directory=/etc/node-exporter/"
      - '--collector.filesystem.ignored-mount-points="^/(sys|proc|dev|host|etc)($$|/)"'

  grafana:
    image: grafana/grafana
    ports:
      - 3000:3000
    depends_on:
      - prometheus
    networks:
      - monitor
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    networks:
      - monitor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    depends_on:
      - redis
    privileged: true

  redis:
    image: redis
    networks:
      - monitor
    ports:
      - "6379"

  db:
   image: mysql:5.7
   restart: always
   environment:
     MYSQL_DATABASE: 'db'
     # So you don't have to use root, but you can if you like
     MYSQL_USER: 'user'
     # You can use whatever password you like
     MYSQL_PASSWORD: 'password'
     # Password for root access
     MYSQL_ROOT_PASSWORD: 'password'
   ports:
        # <Port exposed> : < MySQL Port running inside container>
      - '3306:3306'
   expose:
     # Opens port 3306 on the container
      - '3306'
     # Where our data will be persisted
   volumes:
      - my-db:/var/lib/mysql
     # Names our volume
   networks:
      - monitor
