# Traefik - Proxy Reverso e Balanceador de Carga

Traefik é um proxy reverso e balanceador de carga dinâmico projetado para microsserviços e arquiteturas em nuvem. Ele se integra nativamente a diversas plataformas, como Docker, Kubernetes e Swarm, detectando automaticamente novas configurações e ajustando as regras de roteamento.

## Recursos principais
- **Configuração automática** com descoberta de serviços.
- **Suporte a múltiplos backends** (Docker, Kubernetes, Consul, etc.).
- **Balanceamento de carga** inteligente.
- **Suporte nativo a HTTPS** com Let's Encrypt.
- **Painel de controle web** para monitoramento e gerenciamento.

## Clone do Projeto:

Para clonar os arquivos do Traefik para o servidor, execute o comando:
```
git clone https://github.com/jairisonsouza/traefik.git
```

## Instalação
Para rodar o Traefik em produção:

## Pré requisitos:

* Docker
```
sudo apt install docker.io -y
```
* Docker-Swarm
```
docker swarm init --advertise-addr <IP DA MAQUINA>
```

## Preparando ambiente:

Crie as redes definidas no arquivo `docker-compose.yml` com os comandos:

```
docker network create --driver overlay portainer-net --attachable
docker network create --driver overlay jenkins-net --attachable
docker network create --driver overlay gitlab-net --attachable
```
Obs.: Acesse o arquivo `docker-compose.yml` para certificar o nome de todas as redes necessárias para criação. 

Caso necessário, ajuste o arquivo `docker-compose.yml`:

```
version: '3.9'

services:
  traefik:
    image: traefik:v2.10
    deploy:
      replicas: 1
      restart_policy:
        condition: any
    command:
      - "--providers.docker=true"
      - "--providers.docker.swarmMode=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entryPoint.permanent=true"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=seuemail@dominio.com" # Substitua pelo seu e-mail
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
      - "--api.dashboard=false" # Habilita ou dasabilita o painel de controle
    ports:
      - target: 80
        published: 80
        mode: ingress
      - target: 443
        published: 443
        mode: ingress
    volumes:
      - "traefik-data:/letsencrypt"
    networks:
      - portainer-net
      - jenkins-net
      - gitlab-net

volumes:
  traefik-data:

networks:
  portainer-net:
    external: true
  jenkins-net:
    external: true
  gitlab-net:
    external: true

```

## Executando:

* Para fazer a instalação por comandos de forma direta no terminal:
```
docker stack deploy -c docker-compose.yml traefik
```

* Com o Portainer (recomendado):

Obs.: É necessário instalar o portainer antes. Saiba mais em: https://repo.detran.ap.gov.br/DTIC/portainer. 

1. No menu lateral esquerdo, acesse a opção "Stacks";
2. Clique no botão "+ Add Stack";
3. Digite um nome para a Stack (sugerido: traefik);
4. Em "Build method" selecione a opção "Web Editor";
5. Cole todo o conteúdo do arquivo `docker-compose.yml` dentro da área do editor;
6. Depois clique em "Deploy the Stack";

Isso pode demorar um pouco, aguarde.

Certifique-se de tudo foi iniciado corretamente clicando no nome da Stack criada. Na área "Services" devem conter todas réplicas do Traefik sendo executadas. Sempre que precisar adicionar mais redes ou fazer outras modificações, acesse a Stack e clique em "Editor".

Pronto, o Traefik foi configurado.
