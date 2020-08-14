# Docker in Symfony

Symfony has integration with Docker. Symfony binary reads the info dynamically from Docker and populates `$_SERVER` variable.

As we run `docker-compose up`, our app has access to a `DATABASE_URL` env variable that points to the Docker container.


### Create docker-compose.yaml

```bash
php bin/console make:docker:database
```

### Execute command in the container

Run mysql console in the running container:

```bash
docker-compose exec db mysql -u root --password=password
```
