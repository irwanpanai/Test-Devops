# Deployment

### Melakukan Deployment melalui Docker compose

menuju direktori yang berisi docker compose
```
cd dist/docker-compose
```

gunakan ```docker-compose``` untuk menjalankan container

```
MYSQL_PASSWORD='<some password>' docker compose --file dist/docker-compose/docker-compose.yml up
```
