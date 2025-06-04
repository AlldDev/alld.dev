---
title: "Instalando e Configurando SSH em Servidores Linux"
date: 2025-02-06T13:51:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "SSH", "Segurança"]
tags: ["ssh", "terminal", "nuvem", "servidor", "cybersecurity"]
---

Este guia demonstra como configurar e proteger seu servidor SSH em servidores Linux, tornando sua conexão mais segura e evitando acessos não autorizados. O **SSH (Secure Shell)** é o método mais comum para acessar servidores remotamente de forma segura, e existem várias medidas que você pode tomar para fortalecer a segurança.


Na maioria das distribuições Linux, o **SSH-Server** já vem instalado por padrão. No entanto, se o seu sistema não o tiver, siga as instruções abaixo para instalá-lo.

## 1. Atualize os Pacotes do Sistema e instale o OpenSSH

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt install openssh-server openssh-client
```

Após a instalação, verifique se o serviço está rodando:

```bash
sudo systemctl status ssh
```

> Os arquivos de configuração do **OpenSSH** estão localizados em `/etc/ssh/`. Antes de fazer qualquer alteração, é recomendado fazer um backup do arquivo de configuração principal:
> ```bash
> sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bkp
> ```
> **Nota**: Lembre-se de que o arquivo correto a ser editado é o **`sshd_config`** (servidor) e não o **`ssh_config`** (cliente).

## 2. Configurações Básicas de Segurança

Após o backup, você pode começar a editar o arquivo de configuração **`sshd_config`**. Use o editor de sua preferência, como o `nano`:

```bash
sudo nano /etc/ssh/sshd_config
```
A primeira configuração à ser realizada é **`alterar a Porta Padrão`** de conexão SSH:
```bash
Port 2022
```

> [!NOTE]
> Evite usar portas amplamente conhecidas, como 2022 ou 9022, e caso esteja usando o **Fail2Ban**, lembre-se de alterar a configuração da porta no Fail2Ban também.


## 3. Personalizando configurações de segurança
```bash
# Desativar Login Root
PermitRootLogin no

# Limitar Tentativas de Login
MaxAuthTries 3

# Tempo de Carência para Login
LoginGraceTime 20

# Desativar Autenticação por Senha
## Se você configurou **chaves SSH** para autenticação, desabilite o login por senha para aumentar a segurança:
PasswordAuthentication no

# Bloquear Logins com Senhas Vazias
PermitEmptyPasswords no

# Desativar Autenticações Desnecessárias
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no

# Desativar Encaminhamento Gráfico (X11)
## Se você não precisa de aplicativos gráficos remotos via SSH, desative o X11 forwarding
X11Forwarding no

# Desativar Túnel e Encaminhamentos de Conexão
## Se o seu servidor não utiliza tunelamento SSH ou encaminhamento de portas, é recomendável desativar essas opções
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no

# Desabilitar Banner Detalhado
DebianBanner no
```

## 4. Reiniciando o Serviço SSH

Após realizar todas as alterações no arquivo de configuração, reinicie o serviço SSH para que as mudanças tenham efeito:

```bash
sudo systemctl restart sshd
```

## 5. Configurações Avançadas

Para adicionar uma camada extra de segurança, é recomendável configurar a autenticação via **chave RSA**. Isso garante que apenas usuários com a chave correta possam acessar o servidor, eliminando a necessidade de senhas.

### Gerando Chaves RSA

As chaves RSA devem ser geradas no dispositivo **cliente** (seu computador ou celular) e não no servidor. No cliente, use o seguinte comando para gerar as chaves:

```bash
ssh-keygen -t rsa
```

A chave pública gerada deve ser enviada para o servidor.

### Adicionando Chave Pública no Servidor

Após gerado a chave basta que faça o envio para os servidores de destino:

```bash
ssh-copy-id user@<host>
```

### Configurando SSH para Usar Apenas Chaves RSA

No arquivo de configuração **sshd_config** do servidor, faça as seguintes alterações para permitir apenas a autenticação via chave pública:

```bash
PubkeyAuthentication yes
AuthorizedKeysFile     .ssh/authorized_keys
PasswordAuthentication no
```

Reinicie o serviço SSH:

```bash
sudo systemctl restart sshd
```

Agora, o servidor só permitirá logins com chave RSA, tornando a autenticação muito mais segura.
