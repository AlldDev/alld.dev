---
title: "Rclone - Instalação, Configuração e uso avançado"
date: 2025-02-18T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Rclone"]
tags: ["rclone", "terminal", "linux", "sincronizacao"]
---

## Instalação Básica

```bash
# Linux (Debian/Ubuntu)
curl https://rclone.org/install.sh | sudo bash

# macOS
brew install rclone

# Windows
choco install rclone
```

---

## Comandos Essenciais
```bash
# Listar arquivos
rclone ls remote:nome_do_bucket

# Copiar arquivos
rclone copy origem destino:/pasta

# Sincronizar diretórios
rclone sync origem destino:/pasta --progress

# Excluir arquivos
rclone delete remote:pasta_obsoleta
```

---

## Comandos Avançados
```bash
# Verificar espaço utilizado
rclone about remote:

# Listar diretórios
rclone lsd remote:

# Calcular tamanho
rclone size remote:pasta_importante

# Limpar lixeira
rclone cleanup remote:
```

---

## Flags para Alta Performance
Parâmetros avançados para otimizar suas transferências:

| Flag | Descrição | Valor Recomendado |
|------|-----------|-------------------|
| `-P` | Progresso detalhado | Sempre usar |
| `--transfers` | Transferências paralelas | 2-4 |
| `--checkers` | Verificadores simultâneos | 2-4 |
| `--tpslimit` | Limite de transações/segundo | 4 |
| `--onedrive-chunk-size` | Tamanho do chunk (OneDrive) | 10M |
| `--bwlimit` | Limite de banda | 15M |
| `--low-level-retries` | Tentativas de recuperação | 10 |
| `--retries` | Tentativas normais | 5 |
| `--timeout` | Tempo máximo por operação | 10m |
| `--ignore-checksum` | Ignora verificação de hash | Use com cuidado! |
| `--ignore-case-sync` | Ignora diferença de caixa | Para sistemas case-insensitive |
| `--log-file` | Arquivo de log personalizado | `/caminho/do/log.log` |
| `--log-format` | Formato do log | `date,time` |
| `--user-agent` | Identificação personalizada | `"ISV\|rclone.org\|rclone/v1.50.2"` |

---

## Exemplo de Uso

### Sincronização Otimizada para OneDrive
```bash
rclone sync -P --transfers 2 --checkers 2 --tpslimit 4 \
--onedrive-chunk-size 10M --bwlimit 15M \
--low-level-retries 10 --retries 5 --timeout 10m \
--ignore-checksum --ignore-case-sync \
--log-file /home/ubuntu/logs-rclone/rclone-sync1.log \
--log-format "date,time" --log-level INFO \
--ignore-size \
--user-agent "ISV|rclone.org|rclone/v1.50.2" \
/local/path/ oneDrive:remote_folder
```

### Agendamento com Cron (Backup Diário)
```bash
0 2 * * * /usr/bin/rclone sync -P /dados importantes:backup --bwlimit "08:00,15M 23:00,off"
```

---

## Boas práticas

1. **Teste antes de sincronizar!** Use `--dry-run`
2. **Cuidado com `--ignore-checksum`** - Desativa verificação de integridade
3. **Monitore logs** regularmente: `tail -f /home/ubuntu/logs-rclone/rclone-sync1.log`

---

## Recursos Adicionais

- [Documentação Oficial](https://rclone.org/)
- [Fórum de Suporte](https://forum.rclone.org/)
- [GitHub do Projeto](https://github.com/rclone/rclone)
