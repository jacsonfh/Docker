# Docker e DockerHub

* Este é um resumo/estudo sobre a ferramenta Docker.
* Estou utilizando o Linux Fedora baseado em rpm então se você usa uma distribuição baseada em outra tecnologia por favor adapte seus comandos. *It Works on my machine!* ;)

## Instalar Docker

### Limpar Instalações antigas.
```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
**Verificar também**
* Images, containers, volumes, and networks stored in:
```bash
  `/var/lib/docker/`
  `/etc/docker/daemon.json`
```

## Configurar Repositório.
```bash
#Fedora
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

#Verificar outros repositórios no site oficial do docker.
#Centos
#Debian
```

## Instalação
* A instalação vai variar de acordo com a sua necessidade, eu estou colocando aqui um exemplo da minha.
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo dnf install docker-compose
```

## Configurando usuário específico.
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
#Tive  um erro de falha de execução por falta de permissão, então coloquei esta permissão no socket, mas precisa ser melhorado.
sudo chmod 666 /var/run/docker.sock
```

## Iniciar o Serviço do Docker.
```bash
sudo systemctl daemon-reload
sudo systemctl start docker
sudo systemctl status docker
```

## Teste a Aplicação. **Exemplos**
```bash
docker -v
docker-compose --version
sudo docker run hello-world
sudo docker run -it ubuntu bash
```
## Configurações Adicionais.
* Inicialização automática.
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

sudo systemctl restart containerd.service
sudo systemctl restart docker.service

sudo systemctl status docker.service
sudo systemctl status containerd.service
```

## Possível erro.
>WARNING: Error loading config file: /home/user/.docker/config.json -
>stat /home/user/.docker/config.json: permission denied
This error indicates that the permission settings for the ~/.docker/ directory are incorrect, due to having used the sudo command earlier.
To fix this problem, either remove the ~/.docker/ directory (it's recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:
```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

## Gestão do Log
Configure default logging driver
```bash
sudo vim /etc/docker/daemon.json
```
```json
"log-driver": "json-file",
"log-opts": {
  "max-size": "10m",
  "max-file": "3"
}
```
Inspecionar Logs.
docker logs meu-container1



## Configurar Rede no docker
* Exemplo de rede para as máquinas docker dev.
```bash
sudo vim /etc/docker/daemon.json
{
 "default-address-pools":
 [
 {"base":"10.0.0.0/16","size":24}
 ]
}
```

## Locais que podem ser úteis.
```bash
/var/lib/docker
/var/lib/docker/containers/
/var/lib/containerd
```

## Comando Docker
```bash
# Para iniciar uma docker que esteja configurada.
cd /pasta-desejada
docker-compose -d up

#Baixar uma imagem.
docker image pull python

#Criar uma imagem
docker build -t minha-imagem .

#Executar uma imagem
docker run minha-imagem

#Para ver as imagens do Docker.
docker images
docker image list

#Apagar uma imagem 
docker image rm <ID da imagem>

#Ver os containers rodando.
docker ps
docker ps -a
docker container ps

#Para entrar no container.
docker run -it --name containercriado ubuntu:16.04 bash

#Parando o container
docker stop containercriado

#Criando imagens com Dockerfile
#Primeiro crie um arquivo qualquer para um teste futuro:
touch arquivo_teste
#Crie um arquivo chamado Dockerfile e dentro dele coloque o seguinte conteúdo:
FROM ubuntu:16.04
RUN apt-get update && apt-get install nginx -y
COPY arquivo_teste /tmp/arquivo_teste
CMD bash

#Após construir seu Dockerfile basta executar o comando abaixo:
docker build -t meuubuntu:nginx_auto .


``` 
## Usando Dockerfile

Crie um arquivo chamado Dockerfile na pasta da sua docker.
Um Dockerfile é um arquivo de texto que contém todas as instruções para construir uma imagem do Docker. Ele descreve passo a passo como a imagem deve ser criada, desde a imagem base até os comandos a serem executados dentro do container.
Função:
  Define a imagem base (Ubuntu, Alpine, etc.)
  Copia arquivos para a imagem
  Instala pacotes e dependências
  Define o comando que será executado quando o container for iniciado

Exemplo de Dockerfile:
```bash
LABEL Description="Descrição da minha Docker"

FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt

RUN apt-get update && apt-get install -y vim
RUN pip install -r requirements.txt

CMD ["python", "app.py"] 1   
```

Comandos para criar a imagem:
```bash
docker build
docker build -t minha-imagem-python .
#Exemplo de criar uma segunda versão da docker.
docker build -t minha-imagem-python:v2 .

```

Comando para iniciar o container:
Use o comando docker run para iniciar um container a partir da imagem recém-criada:
```bash
docker run -it --name meu-container minha-imagem-python
docker run -it -v /caminho/local:/caminho/no/container -p 5000:5000 --name meu-container minha-imagem-python

docker run -it -v /home/minhahome/Docker/MinhasDockers/Python/app:/tmp -p 5000:5000 --name meu-python minha-imagem-python
```
## Comandos Úteis.
`docker cp arquivo.txt meu-container1:/caminho/no/container` Copiar para o container.
`docker cp meu-container1:/caminho/no/container arquivo.txt` Copiar do container para fora.
`docker exec -it meu-container1 bash` Acessar a docker.
`docker images`: Lista as imagens disponíveis.
`docker ps`: Lista os containers em execução.
`docker stop nome_do_container`: Para um container.
`docker rm nome_do_container`: Remove um container.

## Atualização do Imagem
```bash
#Se você fizer alterações no container e quiser salvá-las em uma nova imagem:
docker commit meu-container1 nova-imagem
#Obs. No meu último teste depois do commit foi necessário recriar o container.
```

## Instalar diretamente.
```bash
docker exec -it meu-container1 apt-get update && apt-get install -y vim
```

## Usando docker-compose.yml
Exemplo de docker-compose.yml
```bash
version: '3.7'

services:
  app: 
    build:
    ports:
      - "5000:5000"
  db:
    image: postgres
    environment:
      POSTGRES_USER: meu_usuario
      POSTGRES_PASSWORD: minha_senha
      POSTGRES_DB: meu_banco
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```
Execução do docker-compose.
```bash
docker-compose up -d

docker-compose ps, você verá os nomes completos dos containers.

build: minha-imagem-python3

docker-compose up --build -d
```

## Exemplo de docker-compose.yml
```bash
version: '3.7'

services:
  minha_aplicacao:
    build: .
    ports:
      - "5000:5000"
  db:
    image: postgres:latest  # Especificar a tag da imagem
    environment:
      POSTGRES_USER: meu_usuario
      POSTGRES_PASSWORD: minha_senha
      POSTGRES_DB: meu_banco
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:  # Adicionar um healthcheck para verificar se o banco está funcionando
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  postgres_data:
  ```

## Docker Compose.
Padrão de nomeclatura novo versão 2
Compose 2 ignora a tag version.
```bash
compose.yaml
```

### Problemas Docker Login e Repositories.
* Nota: Expandir esta informação.
```bash
/home/meuhome/.docker/config.json.
```

#### Iniciar a minha imagem em foreground.
```bash
docker compose up
```

#### Executar em background usar -d
```bash
docker compose up -d
```

#### Baixar todos os elementos do compose
```bash
docker compose down
```

#### Para apenas um serviço dos que estão rodando no compose 
```bash
docker compose stop web
docker compose start web
```

#### Ver os logs
```bash
docker compose logs
docker compose logs web
docker compose logs -f
```

### listar os containers 
```bash
docker compose ps
```

### Exemplo de uso de dependência.
* Criando uma dependência de execução
```bash
  depends_on:
   - postgre
```

### Exemplo de uso de Build no compose-dev.yaml
```bash
service:
  minhaapp:
    image: minhaimagem/minhai:versao
build:
  context: /src
  Dockerfile: Dockerfile
#Comando para quando o build esta dentro do mesmo compose.
docker compose up -d --build

#Comando para quando o compose está em um arquivo separado.
docker compose -f compose.yaml -f compose-dev.yaml up -d --build
```
* Como uma melhor prática usar um arquivo compose separado para os build
>Ex. compose-dev.yaml

### Exemplo de compose de wordpress com mysql.
>Declarar volumes para ser gerenciados. 
>Nota: Ver também volumes na máquina.
>Criar um arquivo .env e colocar no .gitignore
>Utilizar .env para as CONSTANTES do projeto do compose.yaml
>Comando para "docker compose --env-file .env up -d"

```bash
volumes:
  db_blog:
  wordpress_vol:
  
networks:
  wordpress_net:
    driver: bridge

services:
  mysql:
    image: 
      mysql:9.0.1
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - 3306:3306
    volumes:
      - db_blog:/var/lib/mysql
    networks:
      - wordpress_net
    
  wordpress:
    image:
      wordpress:6.4.3
    depends_on:
      - mysql
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    ports:
      - 8080:80 #Acessar site com http://localhost:8080 e o admin com http://localhost:8080/wp-admin/
    volumes:
      - wordpress_vol:/var/www/html
    networks:
      - wordpress_net
```
