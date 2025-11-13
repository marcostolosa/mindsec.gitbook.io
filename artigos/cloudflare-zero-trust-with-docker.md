---
description: >-
  [pt-br] Expondo Serviços com Segurança: Cloudflare Zero-Trust Tunnel com
  Docker
icon: cloudflare
---

# Cloudflare Zero-Trust with Docker

Aprenda sobre o Cloudflare Zero Tunnel e como utilizá-lo com Docker para expor serviços de forma segura na Internet. Melhore a segurança e acessibilidade dos seus containers Docker.

![Guide - Running Cloudflare Tunnel with Docker](https://fossengineer.com/img/SelfHosting/Docker-Cloudflare.png)

### Cloudflare Zero Trust Tunnel

A **Cloudflare** oferece uma ferramenta chamada **Cloudflare Zero Tunnel**, que provê uma conexão segura entre sua máquina local e a rede edge da Cloudflare.

Neste guia, veremos o que é o Cloudflare Zero Trust Tunnel, suas vantagens e como usá-lo junto ao **Docker** para expor serviços de forma segura.

***

### O que é o Cloudflare Zero Tunnel?

O **Cloudflare Zero Tunnel** cria um **túnel seguro** entre sua máquina local e a rede edge da Cloudflare, permitindo acesso remoto seguro a recursos privados da sua rede **sem expor diretamente seu IP público**.

Ele utiliza a **rede global de datacenters da Cloudflare** para oferecer uma conexão rápida e protegida, contornando a exposição na Internet pública.

O Zero Tunnel usa o **protocolo QUIC**, projetado para ser mais rápido e seguro que conexões TCP tradicionais, e criptografa todos os dados com **TLS 1.3**, garantindo criptografia de ponta a ponta entre sua máquina e o servidor de destino.

***

### Vantagens do Cloudflare Zero Tunnel

**🔒 Segurança aprimorada:**\
O Zero Tunnel oferece criptografia fim-a-fim entre cliente e servidor, protegendo os dados contra interceptação e espionagem.

**🕵️‍♂️ Privacidade aumentada:**\
Permite acesso a recursos privados **sem exposição à Internet pública**, reduzindo risco de acesso não autorizado e ataques.

**🌍 Acessibilidade global:**\
Aproveita a infraestrutura mundial da Cloudflare para oferecer conexões rápidas e seguras de qualquer lugar do planeta — ideal para equipes remotas e ambientes distribuídos.

**⚡ Desempenho e confiabilidade:**

O uso do protocolo QUIC proporciona conexões mais rápidas e estáveis, garantindo uma experiência fluida e consistente.

***

### Implantando o Cloudflare Zero Tunnel com Docker

O **Docker** é uma plataforma popular para empacotar e executar aplicações em containers, incluindo todas as dependências. Isso facilita o deploy e o gerenciamento.

Com o **Cloudflare Zero Tunnel**, é possível expor serviços Docker na Internet de forma segura e **sem precisar abrir portas no roteador**.

### 1. Criar um container Docker para seu serviço

Primeiro, crie um container que execute o serviço localmente.\
Depois, ele será exposto via rede do Cloudflare Tunnel no Docker.

### 2. Instalar o cliente Cloudflare Zero Tunnel

Instale o cliente **`cloudflared`** na máquina local e configure-o para autenticar com sua conta Cloudflare.

Usaremos a imagem Docker oficial **`cloudflare/cloudflared`**, disponível no GitHub da Cloudflare.

Na dashboard da Cloudflare:

1. Selecione o domínio gerenciado.
2. Vá em **Tráfego → Cloudflare Tunnel**, ou diretamente em **Zero Trust**.
3. Na interface **Cloudflare One Dashboard**, acesse **Access → Tunnels** para criar um novo túnel e obter o **token**.

![Navigating Dash Cloudflare interface](https://fossengineer.com/img/SelfHosting/Docker-dash-Cloudflare.png)

### 3. Executar o container do Cloudflare Tunnel

#### Usando CLI:

Crie uma rede Docker compartilhada:

```bash
sudo docker network create tunnel
```

Execute o container do túnel:

```bash
docker run --name cloudflared --network tunnel --detach cloudflare/cloudflared:latest tunnel --no-autoupdate run --token SEU_TOKEN_AQUI
```

#### Usando Docker Compose (recomendado):

```yaml
version: '3.8'

services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    command: tunnel --no-autoupdate run --token SEU_TOKEN_AQUI
    networks:
      - tunnel
    restart: unless-stopped

networks:
  tunnel:
```

Depois, suba o serviço:

```bash
docker-compose up -d
```

![Navigating Dash Cloudflare interface](https://fossengineer.com/img/SelfHosting/Docker-Cloudflared-Container.JPG)

Isso criará um container chamado `cloudflared` e uma rede Docker chamada `tunnel`, que será usada para conectar os outros containers que você quiser expor.

### 4. Adicionar serviços à rede do Cloudflare Tunnel

Com o container `cloudflared` ativo, conecte outros containers Docker à mesma rede:

```bash
docker network connect tunnel nome_do_container
```

Ou, ao criar um novo container:

```bash
docker run --name MeuServico --network tunnel -p 8050:8050 -d nome_da_imagem
```

Exemplo com **docker-compose / Portainer Stack**:

```yaml
version: '3.8'

services:
  app:
    image: nome_da_imagem
    container_name: app
    ports:
      - "8050:8050"
    networks:
      - tunnel

networks:
  tunnel:
    external: true
```

***

### 5. Configurar o hostname público via UI da Cloudflare

No **Cloudflare Dashboard**:

1. Vá em **Access → Tunnels**, selecione o túnel criado.
2. Clique em **Add Public Hostname**.
3. Escolha subdomínio, domínio e path (se necessário).
4. No campo **Service**, informe:

```makefile
nome_do_container:porta_docker
```

Exemplo:

```makefile
dash_app:8050
```

Assim, o tráfego para `https://subdominio.seudominio.com` será roteado até o serviço no container Docker via Cloudflare, com **criptografia, autenticação e sem abrir nenhuma porta no roteador**.

***

### 6. Acessar o serviço

Pronto! Seu serviço agora está acessível com segurança via a rede edge da Cloudflare.\
Você pode acessá-lo de qualquer lugar do mundo, com conexão rápida e protegida.

![Navigating Dash Cloudflare interface](https://fossengineer.com/img/SelfHosting/DockerCloudflare-Add-Hostname.png)

#### **Como verificar o IP local do dispositivo?**

```bash
ifconfig
```

**Como verificar o IP exposto?**\
Você não expõe seu IP público, pois o tráfego passa inteiramente pela Cloudflare:

```bash
curl seu_dominio_ou_subdominio.com && echo
```

**Benefícios adicionais:**

* Nenhuma porta aberta no roteador
* IP público da casa/oficina protegido
* Integração com autenticação Cloudflare Access
* Suporte a múltiplos serviços no mesmo túnel

***

### Referencia

* [https://fossengineer.com/selfhosting-cloudflared-tunnel-docker/](https://fossengineer.com/selfhosting-cloudflared-tunnel-docker/)
