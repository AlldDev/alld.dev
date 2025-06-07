---
title: "Verificação e Reparo de Sistemas de Arquivos (File System Check (fsck) Requerido)"
date: 2025-02-11T11:30:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Solução de Problemas", "Terminal", "Gerenciamento"]
tags: ["boot error", "fsck", "linux", "sistema de arquivos", "erro de boot", "file system check"]
---

O comando **`fsck`** (File System Check) é uma ferramenta essencial para verificar e reparar sistemas de arquivos no Linux. Ele é especialmente útil quando o sistema não inicializa corretamente ou quando você suspeita de corrupção no sistema de arquivos.

---

## O Que é o `fsck`?

O `fsck` é uma ferramenta de linha de comando que verifica a integridade de sistemas de arquivos e corrige erros. Ele pode ser usado em sistemas de arquivos como **ext4**, **ext3**, **xfs**, **btrfs**, entre outros.

---

## Quando Usar o `fsck`?

Você deve considerar usar o `fsck` nas seguintes situações:
1. **Falha na Inicialização**: O sistema não inicializa corretamente.
2. **Desligamento Inadequado**: O sistema foi desligado abruptamente (ex.: queda de energia).
3. **Erros de Leitura/Gravação**: Arquivos corrompidos ou inacessíveis.
4. **Verificação Preventiva**: Para garantir a integridade do sistema de arquivos.

---

## Precauções Antes de Usar o `fsck`

1. **Faça Backup**: Sempre faça backup dos dados importantes antes de executar o `fsck`.
2. **Desmonte o Sistema de Arquivos**: O sistema de arquivos **não deve estar montado** durante a execução do `fsck`. Se for a partição raiz (`/`), use um Live CD/USB.
3. **Verifique o Manual**: Use `man fsck` para detalhes específicos do seu sistema.

---

## Como Usar o `fsck`

### 1. **Sintaxe Básica**
```bash
sudo fsck [opções] [dispositivo]
```
- **`opções`**: Flags para controlar o comportamento do `fsck`.
- **`dispositivo`**: Partição a ser verificada (ex.: `/dev/sda1`).

---

### 2. **Opções Comuns**

| Opção       | Descrição                                      |
|-------------|------------------------------------------------|
| `-A`        | Verifica todos os sistemas de arquivos.        |
| `-C`        | Mostra barra de progresso (para ext2/ext3/ext4).|
| `-N`        | Simula a execução (não faz alterações).        |
| `-T`        | Desativa a exibição do título.                |
| `-V`        | Modo verboso (mostra detalhes da execução).   |
| `-y`        | Responde "sim" a todas as perguntas.          |
| `-n`        | Responde "não" a todas as perguntas.          |
| `-f`        | Força a verificação, mesmo se o sistema parecer limpo.|

---

### 3. **Exemplos Práticos**

#### **Exemplo 1**: Verificar e Reparar uma Partição
```bash
sudo fsck -y /dev/sda1
```
- **O que faz**: Verifica e repara automaticamente erros na partição `/dev/sda1`.

#### **Exemplo 2**: Verificar Todos os Sistemas de Arquivos
```bash
sudo fsck -A -y
```
- **O que faz**: Verifica e repara todos os sistemas de arquivos listados em `/etc/fstab`.

#### **Exemplo 3**: Simular uma Verificação
```bash
sudo fsck -N /dev/sda1
```
- **O que faz**: Mostra o que seria feito sem aplicar alterações.

#### **Exemplo 4**: Verificar com Barra de Progresso
```bash
sudo fsck -C /dev/sda1
```
- **O que faz**: Exibe uma barra de progresso durante a verificação (útil para sistemas de arquivos ext*).

---

## Dicas

### 1. **Verificar Discos com Bad Blocks**
Use a opção `-c` para procurar por setores defeituosos:
```bash
sudo fsck -c /dev/sda1
```

### 2. **Verificar Sistemas de Arquivos Montados como Somente Leitura**
Se você não puder desmontar o sistema de arquivos, monte-o como somente leitura e execute o `fsck`:
```bash
sudo mount -o remount,ro /dev/sda1
sudo fsck /dev/sda1
```

### 3. **Agendar Verificações Periódicas**
O Linux verifica automaticamente sistemas de arquivos após um número específico de montagens ou tempo. Para configurar:
```bash
sudo tune2fs -c 30 /dev/sda1  # Verifica após 30 montagens
sudo tune2fs -i 7d /dev/sda1  # Verifica a cada 7 dias
```

---

## Erros Comuns e Soluções

### 1. **"fsck: cannot continue, aborting"**
- **Causa**: O sistema de arquivos está montado.
- **Solução**: Desmonte o sistema de arquivos ou use um Live CD/USB.

### 2. **"fsck: unexpected inconsistency"**
- **Causa**: Corrupção grave no sistema de arquivos.
- **Solução**: Execute o `fsck` com a opção `-y` para tentar reparar automaticamente.

### 3. **"fsck: cannot open /dev/sda1: Device or resource busy"**
- **Causa**: O dispositivo está em uso.
- **Solução**: Desmonte o dispositivo ou reinicie em modo de recuperação.

---

## Comparação de Sistemas de Arquivos

| Sistema de Arquivos | Comando de Verificação | Observação                          |
|---------------------|------------------------|-------------------------------------|
| **ext4**            | `fsck.ext4`            | Suporta journaling (mais seguro).   |
| **xfs**             | `xfs_repair`           | Não usa `fsck`.                     |
| **btrfs**           | `btrfs check`          | Suporta auto-reparação.             |
| **fat32/ntfs**      | `fsck.vfat`/`ntfsfix`  | Para sistemas de arquivos Windows.  |

---

## Caso Prático: Recuperando um Sistema Corrompido

1. **Reinicie o sistema** e acesse o GRUB.
2. **Edite a linha do kernel** e adicione `init=/bin/bash`.
3. **Monte a partição raiz** como leitura/gravação:
   ```bash
   mount -o remount,rw /
   ```
4. **Execute o `fsck`**:
   ```bash
   fsck -y /dev/sda1
   ```
5. **Reinicie o sistema**:
   ```bash
   reboot
   ```
