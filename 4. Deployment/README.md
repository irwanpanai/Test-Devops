# Deployment

### Melakukan Deployment melalui Docker compose

menuju direktori yang berisi docker compose
```
cd dist/docker-compose
```

gunakan ```docker-compose``` untuk menjalankan container

```
MYSQL_PASSWORD='irwanpanai' docker compose --file dist/docker-compose/docker-compose.yml up
```

### aplikasi berhasil di deploy

![image](https://github.com/irwanpanai/Test-Devops/assets/89429810/595cbf9c-ce41-4bd0-a6f0-852ed8e017a6)

aplikasi dapat di akses di https://shop.irwan.studentdumbways.my.id/

![image](https://github.com/irwanpanai/Test-Devops/assets/89429810/827ed64f-7286-491d-878d-6b747ba17716)

aplikasi berjalan dengan lancar dalam keadaan secure atau https
