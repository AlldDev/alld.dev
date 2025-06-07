---
title: "Básico de contêiner (Docker)"
date: 2025-05-07T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Docker", "Flask"]
tags: ["flask", "rocky-linux", "docker", "linux", "servidor"]
---

Um guia básico completo pra criar, subir e gerenciar containers Docker localmente. Ideal pra aplicações web, APIs, serviços e microsserviços.

---

## 1. ESTRUTURA BÁSICA DO PROJETO

Crie seu projeto com a seguinte estrutura base:

```
meu_app/
├── Dockerfile
├── requirements.txt (ou package.json, etc.)
├── app.py (ou index.js, main.go...)
└── outros arquivos e subpastas
```

---

## 2. EXEMPLO DE `Dockerfile`

```dockerfile
# Imagem base (ajuste para seu runtime: node, go, php, etc)
FROM python:3.11-slim

# Diretório de trabalho no container
WORKDIR /app

# Copia dependências primeiro (pra cache ser mais eficiente)
COPY requirements.txt .

# Instala dependências
RUN pip install --no-cache-dir -r requirements.txt

# Copia o restante da aplicação
COPY . .

# Expõe a porta padrão da aplicação (porta que roda o serviço)
EXPOSE 5000

# Comando para rodar (ajuste conforme seu app)
CMD ["python", "app.py"]
```

> 📌 **Troque o `python`, `pip`, `requirements.txt`, `app.py` conforme a stack da sua aplicação.**
> 
> Para mais exemplos de Dockerfile acesse: [Dockerfile Examples by AlldDev](https://gist.github.com/AlldDev/8f2874e2069e2d425d63825afd49ca0d)

---

## 3. BUILD DA IMAGEM

```bash
docker build -t <user>/meu-app:v1 .
```
* `build`: cria a imagem
* `-t`: **"tag"** → nome da imagem (`<user>/meu-app`) + versão (`v1`) (Pode ser usado `lasted`)
* `.`: indica que o `Dockerfile` está no diretório atual

---

## 4. SUBIR O CONTAINER

```bash
docker run -d -p 8080:5000 --name meu-app <user>/meu-app:v1
```
* `-d`: roda em segundo plano (detached)
* `-p host:container`: mapeia porta local (ex: `localhost:8080`) para a porta exposta no container (ex: `5000`)
* `--name meu-app`: dá um nome ao container
* `<user>/meu-app:v1`: nome da imagem criada

---

## 5. LIBERAR PORTA NO FIREWALL (RockyLinux)

```bash
firewall-cmd --add-port=8080/tcp
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```
> 📌 **Pode variar dependendo da distribuição, `iptables`, `nftables`, etc...**

---

## 6. GERENCIAMENTO BÁSICO DE CONTAINERS

| Ação                  | Comando                          |
| --------------------- | -------------------------------- |
| Ver containers ativos | `docker ps`                      |
| Ver todos             | `docker ps -a`                   |
| Parar container       | `docker stop meu-app`          |
| Remover container     | `docker rm meu-app`            |
| Ver logs              | `docker logs meu-app`          |
| Reiniciar container   | `docker restart meu-app`       |
| Entrar no container   | `docker exec -it meu-app bash` |
| Ver imagens           | `docker images`                  |
| Remover imagem        | `docker rmi <user>/meu-app:v1`    |
| Limpar imagens pendentes       | `docker image prune` ⚠️      |
| Limpar imagens pendentes/sem uso       | `docker image prune -a` ⚠️      |
| Limpar recursos       | `docker system prune -a` ⚠️      |

---

## 7. Exportar e Importar Imagens Docker (Offline / Outro Servidor)

### Exportar imagem Docker para um arquivo `.tar`

```bash
docker save -o nome-da-imagem.tar nome-da-imagem:tag
````

### Importar a imagem Docker em outra máquina

1. Copie o arquivo `.tar` para o novo servidor (via SCP, pendrive, etc).
2. No destino, use:

```bash
docker load -i nome-da-imagem.tar
```

Agora a imagem estará disponível na nova máquina com o mesmo nome e tag.

> Útil para ambientes sem internet, deploys offline ou backups de imagens.

---

## 8. Volume externo (dados persistentes):

```bash
docker run -v /meus-dados:/app/data ...
```

---

## 9. Exemplo de projeto (Python + Flask)

```bash
mkdir flask-app && cd flask-app
echo "flask" > requirements.txt
```

**`app.py`:**

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Olá Mundo, minha API está online!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**`Dockerfile`:** (como já mostrado no inicio)

**`Rode no terminal:`**
```bash
docker build -t <user>/flask-api:v1 .
docker run -d -p 8080:5000 --name meu-flask <user>/flask-api:v1
```

**Acesse em:** `http://localhost:8080`

---

## DICAS

* Use `docker-compose` para gerenciar múltiplos serviços (API + banco + nginx)
* Adicione `.dockerignore` para evitar copiar arquivos desnecessários
* Combine com CI/CD (GitHub Actions) para build/push automático
