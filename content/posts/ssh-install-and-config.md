---
title: "Blindando sua Conexão SSH em Servidores Linux"
date: 2025-02-06T13:51:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "SSH", "Segurança"]
tags: ["ssh", "terminal", "nuvem", "servidor", "cybersecurity"]
---

Este guia vai te ajudar a configurar e proteger seu servidor SSH em sistemas Linux, tornando sua conexão mais segura e evitando acessos não autorizados. O **SSH (Secure Shell)** é o método mais comum para acessar servidores remotamente de forma segura, e existem várias medidas que você pode tomar para fortalecer a segurança.

## Instalação do SSH-Server

Na maioria das distribuições Linux, o **SSH-Server** já vem instalado por padrão. No entanto, se o seu sistema não o tiver, siga as instruções abaixo para instalá-lo.

### Passo 1: Atualize os Pacotes do Sistema

Antes de instalar qualquer software, é sempre uma boa prática atualizar o sistema:

```bash
sudo apt update -y && sudo apt upgrade -y
```

### Passo 2: Instale o OpenSSH Server e Cliente

Instale os pacotes do **OpenSSH** (servidor e cliente) usando o seguinte comando:

```bash
sudo apt install openssh-server openssh-client
```

### Verificando a Instalação

Após a instalação, você pode verificar se o serviço está rodando com o comando:

```bash
sudo systemctl status ssh
```

Os arquivos de configuração do **OpenSSH** estão localizados em `/etc/ssh/`. Antes de fazer qualquer alteração, é recomendado fazer um backup do arquivo de configuração principal:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bkp
```

> **Nota**: Lembre-se de que o arquivo correto a ser editado é o **`sshd_config`** (servidor) e não o **`ssh_config`** (cliente).

## Configurações Básicas de Segurança

Após o backup, você pode começar a editar o arquivo de configuração **`sshd_config`**. Use o editor de sua preferência, como o `nano`:

```bash
sudo nano /etc/ssh/sshd_config
```

### 1. Alterar a Porta Padrão

Uma das primeiras e mais eficazes maneiras de aumentar a segurança do seu servidor SSH é alterar a porta padrão (22) para outra porta menos comum:

```bash
Port 2022
```

> **Nota**: Evite usar portas amplamente conhecidas, como 2022, e se você estiver usando o **Fail2Ban**, lembre-se de alterar a configuração da porta no Fail2Ban também.

### 2. Desativar Login como Root

Impedir logins diretos com o usuário **root** reduz o risco de ataques, forçando o uso de usuários não privilegiados:

```bash
PermitRootLogin no
```

### 3. Limitar Tentativas de Login

Limite o número de tentativas de login antes de desconectar o cliente para evitar ataques de força bruta:

```bash
MaxAuthTries 3
```

### 4. Tempo de Carência para Login

Defina o tempo máximo que um usuário tem para completar a autenticação após se conectar:

```bash
LoginGraceTime 20
```

### 5. Desativar Autenticação por Senha

Se você configurou **chaves SSH** para autenticação, desabilite o login por senha para aumentar a segurança:

```bash
PasswordAuthentication no
```

### 6. Bloquear Logins com Senhas Vazias

Impeça que usuários com senhas vazias façam login:

```bash
PermitEmptyPasswords no
```

### 7. Desativar Autenticações Desnecessárias

Se você não está usando autenticações adicionais como Kerberos ou GSSAPI, desative-as para reduzir a superfície de ataque:

```bash
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
```

### 8. Desativar Encaminhamento Gráfico (X11)

Se você não precisa de aplicativos gráficos remotos via SSH, desative o **X11 forwarding**:

```bash
X11Forwarding no
```

### 9. Desativar Túnel e Encaminhamentos de Conexão

Se o seu servidor não utiliza **tunelamento** SSH ou **encaminhamento** de portas, é recomendável desativar essas opções:

```bash
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
```

### 10. Desabilitar Banner Detalhado

Para evitar fornecer informações desnecessárias sobre o sistema, desabilite o banner detalhado:

```bash
DebianBanner no
```

### Reiniciando o Serviço SSH

Após fazer todas as alterações no arquivo de configuração, reinicie o serviço SSH para que as mudanças tenham efeito:

```bash
sudo systemctl restart sshd
```

## Configurações Avançadas

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

### Considerações Finais

Após seguir esses passos, seu servidor SSH estará significativamente mais protegido. Lembre-se de que a segurança é um processo contínuo, e é importante revisar e atualizar regularmente suas configurações de segurança, especialmente se você estiver lidando com informações sensíveis.

