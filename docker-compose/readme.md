```sh
docker compose up                                           # start the process
                                                            # go to <ip_host>:8000 on a web page
docker image ls
docker compose down                                         # stop the process
docker compose ps
docker compose run web env                                  # get environment variables
docker compose stop                                         # stop services
docker compose down --volumes
docker compose convert                                      # check if it sets the right variables
sudo docker compose --env-file ./others/.env.dev convert
``` 

Environment variables priority
- Compose file
- Shell environment variables
- Environment file
- Dockerfile
- Variable is not defined  

Compose file and CLI variables
- `COMPOSE_API_VERSION`
- `COMPOSE_CONVERT_WINDOWS_PATHS`
- `COMPOSE_FILE`
- `COMPOSE_HTTP_TIMEOUT`
- `COMPOSE_PROFILES`
- `COMPOSE_PROJECT_NAME`
- `COMPOSE_TLS_VERSION`
- `DOCKER_CERT_PATH`
- `DOCKER_HOST`
- `DOCKER_TLS_VERIFY`

```sh
COMPOSE_PROFILES=debug docker compose up            # enable profile debug
COMPOSE_PROFILES=frontend,debug docker compose up   # enable profiles frontend and debug
docker compose up -d                                # do not run service with a required profile
docker compose run db-migrations                    # enable profile 'tool' implicitly
```