<h1 align="center">Infraestrutura de Redes</h1>

## Desafio

Projeto acadêmico com o objetivo de desenvolver uma infraestrutura de rede para a empresa fictícia XPTO, que atenda aos seguintes requisitos:

- Load Balancer: Implementar um sistema de balanceamento de carga utilizando, no mínimo, 3 máquinas para distribuir o tráfego.
- Proxy Reverso: Configurar uma máquina para atuar como proxy reverso, gerenciando as requisições dos usuários para os servidores apropriados.
- Banco de Dados: Implementar uma máquina dedicada para o banco de dados, garantindo a segurança e integridade dos dados.
- VPN: Configurar o acesso à rede através de uma VPN, garantindo que todas as comunicações sejam seguras.
- Docker: Utilizar o Docker para criar os servidores web e o banco de dados, garantindo portabilidade e fácil gestão dos serviços.

## Topologia

<p align="center">
    <img src="./Arquivos/foto-juliano.png" alt="Descrição da Imagem" width="250"/>
</p>

## Estrutura do Projeto

```sh
└── crudRedes.git/
    ├── README.md
    ├── backend
    │   ├── .dockerignore
    │   ├── .env
    │   ├── Dockerfile
    │   ├── controller
    │   ├── database
    │   ├── index.js
    │   ├── package-lock.json
    │   ├── package.json
    │   ├── routes
    │   └── types
    ├── client
    │   ├── Dockerfile
    │   ├── index.html
    │   ├── nginx copy.conf
    │   ├── nginx.conf
    │   ├── script.js
    │   └── style.css
    └── docker-compose.yml
```


## Tecnologias utilizadas

- AWS (EC2 e RDS)
- Docker 
- NGINX 
- MariaDB
- HTML
- Node.js
- JavaScript 
- Debian
- VsCode

## Iniciando 

<details>
<summary>Instâncias na AWS</summary>
</br>
</details>


<details>
<summary>Docker</summary>
</br>

## Colocar o projeto em uma instância EC2

- Criar três instâncias EC2 com as seguintes configurações.

```sh
Sistema Operacional - Debian (64 bits)
Tipo de instância - t2.micro
Grupo de segurança - Libere as portas 443 (HTTPS), 80 (HTTP), 22(SSH), 3306 (MySQL), 3000 e 8800
```
- Conecte-se a instância da forma que preferir.
- Dentro de cada máquina executar os comandos a seguir para usar o docker (Após esses comandos reinicie a máquina).

``` sh
sudo apt update
sudo apt upgrade
sudo apt install docker.io -y
sudo apt install docker-compose -y
sudo apt install nginx -y
sudo apt install node.js -y
sudo apt install mariadb-server
sudo usermod -aG docker $USER (Para usar o docker sem sudo)
sudo apt update
```

- Crie uma pasta chamada crudRedes e copie o projeto do github que será utilizado

``` sh
git clone https://github.com/julianopradoo/crudRedes
```

- Vá para o diretório ~/crudRedes/client e crie um arquivo chamado Dockerfile para buildar nosso frontend com as configurações abaixo.

``` sh
FROM nginx:stable-alpine

# Copiar os arquivos de configuração do Nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Copiar os arquivos estáticos da aplicação para o diretório de serviço do Nginx
COPY ./index.html /usr/share/nginx/html
COPY ./script.js /usr/share/nginx/html
COPY ./style.css /usr/share/nginx/html

# Expondo a porta padrão do Nginx
EXPOSE 80
```

- Vá para o diretório ~/crudRedes/backend e crie um arquivo chamado Dockerfile para buildar nosso backend com as configurações abaixo.

``` sh
# Use a versão leve do Node.js baseada em Alpine
FROM node:18-alpine

# Defina o diretório de trabalho dentro do container
WORKDIR /usr/src/app

# Copie apenas os arquivos necessários para instalar as dependências
COPY package*.json ./

# Instale as dependências com otimizações
RUN npm install --production

# Copie o restante dos arquivos do projeto para o container
COPY . .

# Exponha a porta 3000 para o servidor Node.js
EXPOSE 3000

# Comando para iniciar o servidor Node.js
CMD ["node", "index.js"]
```

- Dentro de crudRedes, na raiz do peojeto, crie um arquivo chamado compose.yaml com as configurações abaixo. Esse arquivo será responsável pela criação do nosso container com os serviços desejados.

``` sh
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile  
    container_name: backend
    environment:
      - DB_HOST=mariadb
      - DB_USER=root
      - DB_PASSWORD=fatec
      - DB_NAME=crud
    ports:
      - "3000:3000"
    depends_on:
      - mariadb
    networks:
      - crudnet
    restart: unless-stopped 

  mariadb:
    image: mariadb:latest
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: fatec
      MYSQL_DATABASE: crud
      MYSQL_USER: root  
      MYSQL_PASSWORD: fatec
    ports:
      - "3306:3306"
    networks:
      - crudnet
    volumes:
      - mariadb_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 10s
    restart: unless-stopped

  frontend:
    build:
      context: ./client
      dockerfile: Dockerfile  # Especifica explicitamente o Dockerfile para evitar confusão
    container_name: frontend
    ports:
      - "80:80"
    networks:
      - crudnet
    depends_on:
      - backend
    restart: unless-stopped  # Política de reinício para garantir alta disponibilidade

networks:
  crudnet:
    driver: bridge  # Mantém a rede de containers isolada

volumes:
  mariadb_data:
    driver: local  # Volume persistente para o banco de dados MariaDB
```
- Crie um arquivo .env em ~/crudRedes/backend com as seguintes configurações abaixo:

``` sh
DB_HOST=http://18.215.2.145/
DB_USER=root
DB_PASSWORD=fatec
DB_NAME=crud
DB_PORT=3306

DBCREATE=true # VARIAVEL QUE HABILITA CRIAÇÃO BANCO CONDICIONAL
LOG=true # VARIAVEL PARA EXIBIÇÃO LOGS
```

- Após essas configurações, execute o comando abaixo para construir e rodar nosso container.

``` sh
docker-compose up --build -d
```
- Após esses passos nossa aplicação estará funcionando no endereço http://18.215.2.145/

 
</details>

<details>
<summary>Load Balancer e Proxy Reverse</summary>
</br>

## Criar e configurar a instância EC2 que irá fazer o proxy reverso e o load balance com NGINX

- Crie uma instância EC2 com as seguintes configurações:

``` sh
Sistema Operacional - Debian (64 bits)
Tipo de instância - t2.micro
Grupo de segurança - Libere as portas 443 (HTTPS), 80 (HTTP), 22(SSH) e 1194 (UDP)
Armazenamento - 1x 30 GiB gp3
```
- Conecte-se a instância da forma que preferir.
- Dentro da máquina executar os comandos a seguir para configurar o Nginx.

``` sh
sudo apt update
sudo apt install nginx -y
sudo apt upgrade
```

- Edite o arquivo de configuração do NGINX:
``` sh
sudo nano /etc/nginx/sites-available/default
```
- Configure o proxy reverso para direcionar o tráfego para os containers no EC2_SERVER
```sh
upstream backend_servers {
    server 18.215.2.145:8001;
    server 18.215.2.145:8002;
    server 18.215.2.145:8003;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```


    
</details>


<details>
<summary>VPN</summary>
</br>
</details>


<details>
<summary>........</summary>
</br>
</details>
















