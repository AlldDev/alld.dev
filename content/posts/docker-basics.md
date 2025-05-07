---
title: "Guia Básico do Docker"
date: 2025-05-07T11:36:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Docker", "Flask"]
tags: ["flask", "rocky-linux", "docker", "linux", "servidor"]
---

Um guia básico completo pra criar, subir e gerenciar containers Docker localmente. Ideal pra aplicações web, APIs, serviços e microsserviços.

---

## 📦 1. ESTRUTURA BÁSICA DO PROJETO

Crie seu projeto com a seguinte estrutura base:

```
meu_app/
├── Dockerfile
├── requirements.txt (ou package.json, etc.)
├── app.py (ou index.js, main.go...)
└── outros arquivos e subpastas
```

---

## 📄 2. EXEMPLO DE `Dockerfile` GENERALISTA

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

---

## 🏗️ 3. BUILD DA IMAGEM

```bash
docker build -t <user>/meu-app:v1 .
```

### 🔍 Explicando:

* `build`: cria a imagem
* `-t`: **"tag"** → nome da imagem (`<user>/meu-app`) + versão (`v1`)
* `.`: indica que o `Dockerfile` está no diretório atual

---

## 🚀 4. SUBIR O CONTAINER

```bash
docker run -d -p 8080:5000 --name meu-app <user>/meu-app:v1
```

### 🧠 Explicação das flags:

* `-d`: roda em segundo plano (detached)
* `-p host:container`: mapeia porta local (ex: `localhost:8080`) para a porta exposta no container (ex: `5000`)
* `--name meu-app`: dá um nome ao container
* `<user>/meu-app:v1`: nome da imagem criada

---

## 🔥 5. LIBERAR PORTA NO FIREWALL (Linux)

```bash
firewall-cmd --add-port=8080/tcp
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

---

## 🛠️ 6. GERENCIAMENTO DE CONTAINERS

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

## 🧠 7. ESTRATÉGIAS ADICIONAIS

### ✅ Multi-stage build (quando usar compilação, como em React/Go):

```dockerfile
# Stage 1: build
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2: serve app
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
```

### ✅ Variáveis de ambiente:

```dockerfile
ENV FLASK_ENV=production
```

### ✅ Volume externo (dados persistentes):

```bash
docker run -v /meus-dados:/app/data ...
```

---

## 🎁 8. EXEMPLO DE PROJETO PRONTO (PYTHON FLASK)

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
    return "Olá Rosão, sua API está online!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**`Dockerfile`:** (como já mostrado acima)

```bash
docker build -t <user>flask-api:v1 .
docker run -d -p 8080:5000 --name meu-flask <user>/flask-api:v1
```

**Acesse em:** `http://localhost:8080`

---

## 💡 BONUS: DICAS + COMO ACESSAR DADOS DO CONTAINER

* Use `docker-compose` para gerenciar múltiplos serviços (API + banco + nginx)
* Adicione `.dockerignore` para evitar copiar arquivos desnecessários
* Combine com CI/CD (GitHub Actions) para build/push automático

## 📁 Acessando o Container e Copiando Arquivos

### 🔄 Entrando no container em modo terminal

Para interagir com um container em execução (como se fosse um terminal Linux):

```bash
docker exec -it nome-do-container bash
````

> Se a imagem não tiver `bash`, use `sh`:

```bash
docker exec -it nome-do-container sh
```

### 📤 Copiar arquivos de dentro do container para o host

```bash
docker cp nome-do-container:/caminho/dentro/container /caminho/no/host
```

**Exemplo:**

```bash
docker cp rosao-api:/app/uploads/imagem.png /home/rosao/imagens/
```

### 📥 Copiar arquivos do host para dentro do container

```bash
docker cp /caminho/no/host nome-do-container:/caminho/no/container
```

**Exemplo:**

```bash
docker cp /home/rosao/config.json rosao-api:/app/config/config.json
```

> 💡 Dica: combine com `docker exec` para verificar se o arquivo foi copiado corretamente:

```bash
docker exec -it rosao-api ls /app/uploads
```

---
