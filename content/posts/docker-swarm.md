---
title: "Docker Swarm - Direto ao ponto!"
date: 2025-09-09T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Docker", "Docker Swarm"]
tags: ["swarm", "scale", "docker", "linux", "containers"]
---


O Docker Swarm é uma ferramenta de orquestração de contêineres valorizada por sua simplicidade de uso e alta integração com o ecossistema Docker. Seus principais pontos fortes incluem a simplicidade de configuração e a compatibilidade nativa com outras ferramentas Docker, oferecendo funcionalidade excelente para gerenciar contêineres já existentes e sendo extremamente compatível com arquivos `docker-compose`.

```bash
# Iniciando um nó
docker swarm init --advertise-addr 192.168.0.164

# Output:
> Swarm initialized: current node (s0d5768gwa78y9vm7m2o6y8eg) is now a manager.
> To add a worker to this swarm, run the following command:
> docker swarm join --token <TOKEN...>
> To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions
```
_Necessário informar o `--advertise-addr` caso possua mais de um IP vinculado à placa de rede._

Após inicializar o nó manager, outros nós (workers ou managers) podem ser adicionados ao cluster utilizando o comando de conexão `docker swarm join` que é exibido como output no momento da inicialização do primeiro nó:
```bash
docker swarm join --token <TOKEN...>
```

Com o cluster configurado, o próximo passo é a criação de um **serviço**. O serviço é a definição do estado desejado para a aplicação. É nele que declaramos as regras de como o container deve ser executado, incluindo sua escala (número de réplicas), distribuição entre os nós e outras políticas:
```bash
# Criando o serviço
docker service create --name <nome-do-serviço> --replicas 3 --publish 8080:80 <nome-da-imagem-docker>

# --name : Nome do meu serviço
# --replicas : Quantidades de containers que irá ser utilizados
# --publish : "Port Forwarding" da aplicação, expoe a porta publica para a privada dos containers

# Similar Output:
> z5b69dzxbxb9mx9iplv1fpfvf
> overall progress: 3 out of 3 tasks
> 1/3: running   [==================================================>]
> 2/3: running   [==================================================>]
> 3/3: running   [==================================================>]
> verify: Service z5b69dzxbxb9mx9iplv1fpfvf converged
```

Após criar o serviço, os containers serão implantados nos nós do cluster. Para visualizar os containers em execução **em um nó específico**, utilize o comando `docker ps` diretamente naquele nó. Para obter uma visão geral de todas as tarefas do serviço em todos os nós, o comando correto é `docker service ps <nome-do-serviço>`:
```bash
# Visualizando os containers em um nó local
docker ps

# Output
> CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS     NAMES
> de8f6c34424f   <nome>/site-nginx:v1   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp    cluster-site-simples.3.367rzsb4u73bq4jtvfeh4gc45
> 24c9ac407440   <nome>/site-nginx:v1   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp    cluster-site-simples.2.55oqkobwyr1cuoio4fgzr5gbd
> 755dbd6ac228   <nome>/site-nginx:v1   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp    cluster-site-simples.1.tnj0pq7zdhl1mk489eyyecg31
```

E pronto! Sua aplicação já está distribuída, escalada e sendo balanceada automaticamente pelo Docker Swarm. O roteamento de rede para as instâncias do serviço é feito pelo **routing mesh** interno do Swarm, que, por padrão, distribui as conexões entre as tarefas (containers) usando o algoritmo **Round-Robin**.

Comandos básicos:
```bash
# Criar, Deletar ou Ver os clusters criados
docker service [create|rm|ls]

# Atualizar o mapeamento das portas da aplicação
docker service update --publish-[add|rm] 8080:80 <nome-cluster>

# Escalar a aplicação
docker service scale <nome-cluster>=<quantidade-de-vms>

# Ver todos os containers do cluster e seus estados
docker service ps <nome-do-cluster>
```
