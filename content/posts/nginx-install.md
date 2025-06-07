---
title: "Nginx - Instalação, Configuração e Segurança no Linux"
date: 2025-02-27T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Nginx", "Web"]
tags: ["nginx", "terminal", "servidor web", "linux"]
---

Este guia abrange **Ubuntu**, **Rocky Linux** e **Arch Linux**, com soluções para problemas comuns e geração de certificados SSL gratuitos.

---

## Instalação do Nginx
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install nginx

# Rocky Linux/CentOS
sudo dnf install nginx

# Arch Linux
sudo pacman -S nginx-mainline

# Habilite e inicie os serviços
sudo systemctl start nginx && sudo systemctl enable nginx
```

---

## Configuração Básica

### 1. Configurar Bloco do Servidor (Virtual Host)
- Crie um arquivo em `/etc/nginx/sites-available/meusite.conf` (Ubuntu) ou `/etc/nginx/conf.d/meusite.conf` (Arch/Rocky):
```nginx
server {
    listen 80;
    server_name meudominio.com www.meudominio.com;
    root /var/www/meusite;
    index index.html;

    # Redireciona todo tráfego HTTP para HTTPS (após gerar o certificado)
    return 301 https://$host$request_uri;  # <- Comente esta linha se ainda não tem HTTPS

    location / {
        try_files $uri $uri/ =404;
    }
}
```
> **Nota:**
> Caso queira subir seu serviço na porta 80, é necessário passar `default_server` na frente do listen, pois por padrão ele pegará a pagina default do Nginx (que já utiliza essa porta).
> Caso não queira passar esse parametro, é necessário acessar o arquivo `nginx.conf` e remover o vhost server que está lá dentro (o que utiliza a porta 80).

- Crie um link simbólico (Ubuntu):
```bash
sudo ln -s /etc/nginx/sites-available/meusite.conf /etc/nginx/sites-enabled/
```

### 2. Testar e Recarregar
```bash
sudo nginx -t       # Verificar erros de sintaxe
sudo systemctl reload nginx
```

### 3. Firewall
```bash
# Ubuntu
sudo ufw allow 'Nginx Full'

# Rocky Linux
sudo firewall-cmd --permanent --add-service=http --add-service=https
sudo firewall-cmd --reload

# Arch Linux (usando nftables ou iptables)
sudo nft add rule inet filter input tcp dport {80, 443} counter accept
```

---

## Fortalecendo a Segurança

### 1. Ocultar Versão do Nginx
Adicione ao arquivo `/etc/nginx/nginx.conf`:
```nginx
server_tokens off;
```

### 2. Limitar Requisições (Prevenção a DDoS/Brute Force)
```nginx
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
}

server {
    location / {
        limit_req zone=one burst=20;
    }
}
```

### 3. Configurar SSL/TLS (configurar após gerar o SSL)
Adicionar abaixo do `server{}` já existente no arquivo `.conf` do seu site.
```nginx
server {
    # Porta e protocolo
    listen 443 ssl http2;
    server_name SEUDOMINIO.COM www.SEUDOMINIO.COM;

    # Certificados SSL (substitua pelos caminhos corretos)
    ssl_certificate /etc/letsencrypt/live/SEUDOMINIO.COM/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/SEUDOMINIO.COM/privkey.pem;

    # Configurações de segurança SSL
    ssl_protocols TLSv1.2 TLSv1.3;  # Suporta apenas TLS 1.2 e 1.3 (mais seguro)
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;  # Cifras modernas
    ssl_prefer_server_ciphers on;  # Prioriza as cifras do servidor
    ssl_session_cache shared:SSL:10m;  # Cache de sessão SSL para melhor desempenho
    ssl_session_timeout 10m;  # Tempo de vida da sessão SSL
    ssl_stapling on;  # Habilita OCSP Stapling para melhorar a segurança
    ssl_stapling_verify on;  # Verifica a resposta OCSP

    # Resolver para OCSP Stapling
    resolver 8.8.8.8 8.8.4.4 valid=300s;  # Usa DNS públicos do Google
    resolver_timeout 5s;

    # Headers de segurança
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";  # HSTS
    add_header X-Content-Type-Options "nosniff";  # Evita MIME type sniffing
    add_header X-Frame-Options "SAMEORIGIN";  # Impede clickjacking
    add_header X-XSS-Protection "1; mode=block";  # Proteção contra XSS
    add_header Referrer-Policy "strict-origin";  # Controla o envio de referrer

    # Diretório raiz do site (substitua pelo caminho correto)
    root /var/www/SEUDOMINIO.COM;
    index index.html index.htm;

    # Configuração de localização padrão
    location / {
        try_files $uri $uri/ =404;  # Tenta servir arquivos estáticos ou retorna 404
    }

    # Bloqueia acesso a arquivos ocultos (ex: .env, .git)
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # Cache para arquivos estáticos (imagens, CSS, JS, etc.)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2)$ {
        expires 365d;  # Cache de 1 ano
        access_log off;  # Desativa logs para melhorar desempenho
    }

    # Página de erro 404 personalizada
    error_page 404 /404.html;
    location = /404.html {
        internal;  # Acesso interno apenas
    }

    # Página de erro 50x personalizada
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        internal;
    }
}
```
### 4. Caso utilize um serviço a parte (como servidores Java, Python, etc...)
```bash
server {
        server_name <server-name>;
        server_tokens off;

        location / {
                proxy_pass <URL-da-aplicação>; # Exemplo http://localhost:5000
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_hide_header X-Powered-By;
        }
}
```

## Solução de Problemas Comuns

### 1. **Erro 403 Forbidden**
- Verifique permissões do diretório `root`:
```bash
# Ubuntu
sudo chown -R www-data:www-data /var/www/meusite

# Rocky/Arch
sudo chown -R nginx:nginx /var/www/meusite
```
- Liberando contexto do `SELinux`:
```bash
# Rocky/Arch
sudo chcon -R -t httpd_sys_content_t <path>
```

### 2. **Porta 80 já em uso**
- Identifique o processo conflitante:
```bash
sudo lsof -i :80
# Encerre o processo (ex: Apache) ou ajuste a configuração do Nginx.
```

### 3. **Erro de Sintaxe no NGINX**
```bash
sudo nginx -t  # Mostra o erro exato e o arquivo afetado.
```

---

## Gerando Certificado SSL Gratuito com Let's Encrypt

### 1. Instale o Certbot
```bash
# Ubuntu
sudo apt install certbot python3-certbot-nginx

# Rocky Linux
sudo dnf install epel-release
sudo dnf install certbot python3-certbot-nginx

# Arch Linux
sudo pacman -S certbot certbot-nginx
```

### 2. Obter Certificado
```bash
sudo certbot --nginx -d meudominio.com -d www.meudominio.com
```
- Siga as instruções interativas. O Certbot atualizará automaticamente o arquivo de configuração do Nginx.

### 3. Renovação Automática
- Teste a renovação:
```bash
sudo certbot renew --dry-run
```
- Adicione um cron job (abrir `crontab -e`):
```cron
0 0 * * * /usr/bin/certbot renew --quiet
```

### Habilitar no site
- Volte ao passo `Fortalecendo a Segurança` no topico `Configurar SSL/TLS` para habilitar o certificado SSL gerado ao site. 
