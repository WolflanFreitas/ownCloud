# OwnCloud Docker Compose Setup

Este repositório contém arquivos de configuração para configurar e executar uma instância do OwnCloud usando Docker Compose.

## Pré-requisitos

Certifique-se de ter os seguintes softwares instalados:

- Docker
- Docker Compose

## Estrutura do Projeto

```
.
├── docker-compose.yml
├── .env
└── README.md
```

- `docker-compose.yml`: Arquivo de configuração do Docker Compose.
- `.env`: Arquivo de variáveis de ambiente.
- `README.md`: Este arquivo de documentação.

## Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto com as seguintes variáveis de ambiente:

```
OWNCLOUD_VERSION=10.14
OWNCLOUD_DOMAIN=localhost:8080
OWNCLOUD_TRUSTED_DOMAINS=localhost
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin
HTTP_PORT=8080
MYSQL_PORT=3307
```

## Como usar

1. **Clone o repositório:**

    ```sh
    git clone <URL-do-seu-repositório>
    cd <nome-do-repositório>
    ```

2. **Crie a rede necessária:**

    ```sh
    docker network create owncloud-network
    ```

3. **Suba os containers:**

    ```sh
    docker-compose up -d
    ```

4. **Acesse o OwnCloud:**

    Abra o navegador e acesse `http://localhost:8080`. Siga as instruções na tela para concluir a configuração.

## Configuração do Docker Compose

Aqui está a configuração do arquivo `docker-compose.yml`:

```yaml
version: "3.8"

volumes:
  files:
    driver: local
  mysql:
    driver: local
  redis:
    driver: local

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    container_name: owncloud_server
    restart: always
    ports:
      - ${HTTP_PORT}:8080
    depends_on:
      - mariadb
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_TRUSTED_DOMAINS=${OWNCLOUD_TRUSTED_DOMAINS}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=mariadb
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - files:/mnt/data

  mariadb:
    image: mariadb:10.11
    container_name: owncloud_mariadb
    ports:
      - ${MYSQL_PORT}:3306
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=owncloud
      - MYSQL_DATABASE=owncloud
      - MARIADB_AUTO_UPGRADE=1
    command: ["--max-allowed-packet=128M", "--innodb-log-file-size=64M"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-u", "root", "--password=owncloud"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - mysql:/var/lib/mysql

  redis:
    image: redis:6
    container_name: owncloud_redis
    restart: always
    command: ["--databases", "1"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - redis:/data
```

## Volumes

Os dados do OwnCloud serão armazenados nos volumes Docker `files`, `mysql` e `redis`. Certifique-se de que esses volumes existam e tenham as permissões corretas.

## Parando e Removendo os Containers

Para parar os containers, use:

```sh
docker-compose down
```

Para parar e remover os containers, redes e volumes criados pelo `up`, use:

```sh
docker-compose down --volumes
```

## Backup e Restauração

Para fazer backup dos dados do OwnCloud, você pode usar os volumes Docker. Por exemplo:

```sh
docker run --rm -v owncloud_files:/mnt/files -v $(pwd):/backup busybox tar czvf /backup/owncloud_files_backup.tar.gz -C /mnt files
```

Para restaurar, use:

```sh
docker run --rm -v owncloud_files:/mnt/files -v $(pwd):/backup busybox tar xzvf /backup/owncloud_files_backup.tar.gz -C /mnt
```

## Suporte

Se você encontrar algum problema, sinta-se à vontade para abrir uma issue no repositório.
