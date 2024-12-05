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
</details>


<details>
<summary>VPN</summary>
</br>
</details>




<details>
<summary>Banco de Dados</summary>
</br>
</details>




<details>
<summary>Load Balancer e Proxy Reverse</summary>
</br>
</details>



<details>
<summary>........</summary>
</br>
</details>
















