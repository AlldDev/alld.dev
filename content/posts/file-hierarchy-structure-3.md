---
title: "Estrutura de Diretórios Linux: Guia do FHS 3.0"
date: 2025-02-11T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "FHS3.0"]
tags: ["fhs", "terminal", "file system structure", "linux", "estrutura de diretorios"]
---

O **Filesystem Hierarchy Standard (FHS)** é a espinha dorsal da organização de arquivos em sistemas Linux. Entender essa estrutura é essencial para administradores, desenvolvedores e usuários avançados.

---

## Visão Geral do FHS
O FHS define onde os arquivos devem estar localizados, garantindo consistência entre distribuições (Ubuntu, Fedora, Debian, etc.). Isso facilita:
- **Manutenção do sistema**  
- **Localização rápida de arquivos**  
- **Compatibilidade entre softwares**  

---

## Estrutura de Diretórios: Do Raiz aos Detalhes

### 1. **`/` (Diretório Raiz)**  
- **Função**: Base de todo o sistema de arquivos.  
- **Exemplo Prático**:  
  ```bash
  # Listar conteúdo do diretório raiz
  ls /
  ```
  Saída típica: `bin`, `etc`, `home`, `usr`, `var`, etc.

---

### 2. **`/bin` (Binários Essenciais do Usuário)**  
- **O que contém**: Comandos básicos para **todos os usuários**, mesmo em modo de recuperação.  
- **Exemplos de comandos**:  
  - `ls`, `cp`, `mv`, `rm`, `cat`, `grep`.  
- **Curiosidade**: Em algumas distros, `/bin` é um link simbólico para `/usr/bin`.

---

### 3. **`/etc` (Configurações do Sistema)**  
- **Função**: Arquivos de configuração **global** do sistema e aplicativos.  
- **Exemplos notáveis**:  
  - `/etc/apt/sources.list`: Lista de repositórios do APT (Debian/Ubuntu).  
  - `/etc/ssh/sshd_config`: Configuração do servidor SSH.  
  - `/etc/hosts`: Mapeamento manual de IPs para nomes de domínio.  
- **Dica**: Sempre faça backup deste diretório antes de alterar configurações críticas!

---

### 4. **`/dev` (Dispositivos de Hardware)**  
- **Função**: Arquivos que representam dispositivos físicos ou virtuais.  
- **Exemplos**:  
  - **Discos**: `/dev/sda` (primeiro disco SATA), `/dev/nvme0n1` (SSD NVMe).  
  - **Partições**: `/dev/sda1` (primeira partição do disco SATA).  
  - **Dispositivos Virtuais**: `/dev/random` (gerador de números aleatórios).  
- **Caso de Uso**:  
  ```bash
  # Montar um pendrive em /mnt
  sudo mount /dev/sdb1 /mnt/usb
  ```

---

### 5. **`/home` (Arquivos do Usuário)**  
- **Função**: Diretórios pessoais para cada usuário.  
- **Estrutura típica**:  
  ```bash
  /home/joao/Documentos  # Documentos
  /home/ana/Imagens      # Fotos
  /home/maria/.ssh       # Chaves SSH (arquivo oculto)
  ```
- **Importante**: Permissões restritas garantem privacidade entre usuários.

---

### 6. **`/var` (Dados Variáveis)**  
- **Função**: Dados que mudam durante a execução do sistema.  
- **Subdiretórios-chave**:  
  - `/var/log`: **Logs do sistema** (ex.: `syslog`, `auth.log`, `nginx/access.log`).  
  - `/var/www`: Arquivos de sites (comum em servidores Apache).  
  - `/var/cache`: Cache de aplicativos (ex.: pacotes do APT).  
- **Caso de Uso**:  
  ```bash
  # Monitorar logs em tempo real
  tail -f /var/log/syslog
  ```

---

### 7. **`/tmp` (Arquivos Temporários)**  
- **Função**: Arquivos temporários **apagados na reinicialização**.  
- **Exemplo**:  
  ```bash
  # Criar um arquivo temporário
  echo "Teste" > /tmp/arquivo_temporario.txt
  ```

---

### 8. **`/usr` (Recursos do Usuário)**  
- **Função**: Contém a maioria dos programas e bibliotecas do sistema.  
- **Estrutura**:  
  - `/usr/bin`: Comandos do usuário (ex.: `python`, `nano`, `git`).  
  - `/usr/lib`: Bibliotecas compartilhadas (ex.: `libc.so`).  
  - `/usr/local`: Softwares instalados manualmente (prioridade sobre o sistema).  
  - `/usr/share`: Dados independentes de arquitetura (ex.: ícones, fontes).  
- **Dica**: O comando `which` revela onde um programa está instalado:  
  ```bash
  which ls   # Geralmente mostra /usr/bin/ls
  ```

---

### 9. **`/boot` (Arquivos de Inicialização)**  
- **Função**: Arquivos necessários para a **inicialização do sistema**.  
- **Conteúdo típico**:  
  - `vmlinuz`: Kernel do Linux.  
  - `initrd.img`: Disco RAM inicial para carregar módulos do kernel.  
  - `grub/`: Configurações do GRUB (gerenciador de boot).  
- **Atenção**: Evite deletar arquivos aqui – pode tornar o sistema não inicializável!

---

### 10. **`/opt` (Software Opcional)**  
- **Função**: Programas de terceiros não gerenciados pelo pacote do sistema.  
- **Exemplos**:  
  - `/opt/google/chrome/`: Instalação manual do Chrome.  
  - `/opt/jetbrains/idea/`: IDE IntelliJ IDEA.  
- **Por que usar?**: Mantém softwares auto-contidos, evitando conflitos de dependências.

---

### 11. **`/sbin` (Binários de Administração)**  
- **Função**: Comandos para **administração do sistema** (exige `root`).  
- **Exemplos**:  
  - `fdisk`: Gerenciamento de partições.  
  - `iptables`: Configuração de firewall.  
  - `reboot`: Reiniciar o sistema.  
- **Caso de Uso**:  
  ```bash
  # Listar discos e partições
  sudo fdisk -l
  ```

---

### 12. **`/proc` e `/sys` (Interface com o Kernel)**  
- **Função**:  
  - `/proc`: Arquivos virtuais com informações de **processos e hardware** em tempo real.  
  - `/sys`: Configurações do kernel e dispositivos.  
- **Exemplos Úteis**:  
  - **CPU**: `cat /proc/cpuinfo` (detalhes do processador).  
  - **Memória**: `cat /proc/meminfo` (uso de RAM).  
  - **Dispositivos USB**: `lsusb` (usa dados de `/sys/bus/usb/`).

---

### 13. **`/mnt` e `/media` (Montagem de Discos)**  
- **Diferença**:  
  - `/mnt`: Para montagem **temporária** (ex.: discos externos).  
  - `/media`: Montagem automática de dispositivos **removíveis** (pendrives, CDs).  
- **Exemplo**:  
  ```bash
  # Montar um HD externo manualmente
  sudo mount /dev/sdc1 /mnt/hd_externo
  ```

---

### 14. **`/lib` e `/lib64` (Bibliotecas Essenciais)**  
- **Função**: Bibliotecas compartilhadas para os binários de `/bin` e `/sbin`.  
- **Exemplo**:  
  - `libc.so`: Biblioteca padrão do C, usada por quase todos os programas.

---

### 15. **`/srv` (Dados de Serviços)**  
- **Função**: Dados específicos de serviços (ex.: sites, repositórios Git).  
- **Exemplo**:  
  - `/srv/http/`: Arquivos de um servidor web.  
  - `/srv/git/`: Repositórios Git remotos.

---

### 16. **`/run` (Dados de Runtime)**  
- **Função**: Arquivos temporários de processos em execução (ex.: PIDs, sockets).  
- **Exemplo**:  
  - `/run/sshd.pid`: ID do processo do servidor SSH.

---

## Casos Práticos de Uso

### **Cenário 1**: Instalando um Software Manualmente  
- **Passo a Passo**:  
  1. Baixe o `.tar.gz` do software.  
  2. Extraia em `/opt/nome_do_software`.  
  3. Crie um link simbólico em `/usr/local/bin` para facilitar o acesso:  
     ```bash
     sudo ln -s /opt/nome_do_software/bin/app /usr/local/bin/app
     ```

### **Cenário 2**: Solucionando Problemas de Espaço em Disco  
- **Comandos Úteis**:  
  ```bash
  # Verificar uso em /var/log (logs podem crescer muito)
  du -sh /var/log/*
  
  # Limpar cache do APT
  sudo apt clean   # Limpa /var/cache/apt/archives/
  ```

---

## 📌 Cheat Sheet Rápido
| Diretório       | Uso Principal                  | Exemplo de Conteúdo         |
|-----------------|--------------------------------|-----------------------------|
| `/etc`          | Configurações globais          | `sshd_config`, `fstab`      |
| `/var/log`      | Logs do sistema                | `syslog`, `nginx/access.log`|
| `/usr/local/bin`| Softwares instalados manualmente | `python3.9`, `meu_script`  |
| `/dev`          | Dispositivos                   | `sda`, `tty`, `null`        |

---

**Referência Oficial**: [FHS 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
