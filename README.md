# Guia Prático de GraphQL com Python 🐍

## 🎯 Alternativas de Execução

### 1. Google Colab
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/seu-usuario/graphql-python-guide/blob/main/notebooks/graphql_demo.ipynb)

### 2. Replit
[![Run on Repl.it](https://replit.com/badge/github/seu-usuario/graphql-python-guide)](https://replit.com/github/govinda777/graphql-python-guide)

### 3. Local com Docker
[![Docker Hub](https://img.shields.io/badge/Docker-Hub-blue)](https://hub.docker.com/r/govinda777/graphql-python-guide)

## 🚀 Quickstart

### Opção 1: Google Colab
1. Clique no botão "Open In Colab" acima
2. Execute as células na ordem
3. O ambiente já vem com todas as dependências necessárias

### Opção 2: Replit
1. Clique no botão "Run on Repl.it"
2. O ambiente será configurado automaticamente
3. Execute o comando `python main.py`

### Opção 3: Docker Local
```bash
docker pull govinda777/graphql-python-guide
docker run -p 8000:8000 seu-usuario/graphql-python-guide
```

## 📦 Estrutura do Projeto

```
/graphql-python-guide
├── /notebooks
│   ├── graphql_demo.ipynb      # Notebook Colab/Jupyter
│   └── graphql_advanced.ipynb  # Exemplos avançados
├── /src
│   ├── server.py              # Servidor GraphQL
│   ├── client.py              # Cliente GraphQL
│   └── examples/              # Exemplos adicionais
├── docker-compose.yml         # Configuração Docker
└── requirements.txt           # Dependências Python
```

[Resto do conteúdo anterior permanece o mesmo...]

## 💡 Novos Exemplos por Ambiente

### Google Colab
```python
# Instalação das dependências
!pip install strawberry-graphql fastapi uvicorn gql

# Importações e código igual ao anterior...
```

### Replit
```python
# Criação do arquivo .replit
language = "python3"
run = "python main.py"

# Requirements
# requirements.txt
strawberry-graphql==0.192.0
fastapi==0.100.0
uvicorn==0.22.0
gql==3.4.0
```

### Docker
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000

CMD ["uvicorn", "src.server:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 🔄 Como Testar em Cada Ambiente

### Google Colab
```python
# Em uma célula
!pip install strawberry-graphql fastapi uvicorn gql

# Em outra célula
%%writefile server.py
[código do servidor anterior]

# Em outra célula
!python server.py
```

### Replit
1. Fork do projeto
2. O ambiente será configurado automaticamente
3. Clique em "Run"

### Docker Local
```bash
# Build
docker build -t graphql-demo .

# Run
docker run -p 8000:8000 graphql-demo

# Test
curl -X POST http://localhost:8000/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ hello }"}'
```

[Resto do conteúdo anterior permanece o mesmo...]
