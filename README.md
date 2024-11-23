# graph-ql-poc-async

# Guia Prático de GraphQL com Python 🐍
[![Open In Deepnote](https://deepnote.com/buttons/launch-in-deepnote.svg)](https://deepnote.com/launch?url=https://github.com/seu-usuario/graphql-python-guide)

## 🎯 Objetivo
Este guia explica GraphQL Subscriptions e Resolvers usando Python, com exemplos práticos executáveis no Deepnote.

## 📚 Índice
1. [Conceitos Básicos](#conceitos-básicos)
2. [Subscriptions](#subscriptions)
3. [Resolvers](#resolvers)
4. [Implementação Prática](#implementação-prática)

## Conceitos Básicos

### O que é GraphQL?
GraphQL é uma linguagem de consulta para APIs que permite que os clientes solicitem exatamente os dados que precisam. Diferente de REST, com GraphQL você tem:
- Um único endpoint
- Queries precisas
- Tipagem forte
- Resolvers flexíveis

### Principais Operações
1. **Query**: Busca dados
2. **Mutation**: Modifica dados
3. **Subscription**: Mantém conexão em tempo real

## Subscriptions

### O que são Subscriptions?
Subscriptions permitem manter uma conexão em tempo real entre servidor e cliente usando WebSocket. São úteis para:
- Chat em tempo real
- Notificações
- Dados em tempo real
- Atualizações ao vivo

### Exemplo Básico de Subscription
[![Open Subscription Example In Deepnote](https://deepnote.com/buttons/launch-in-deepnote.svg)](https://deepnote.com/launch?url=https://gist.github.com/subscription-example.py)

```python
import strawberry
from strawberry.asgi import GraphQL
from strawberry.subscriptions import Subscribe
from typing import AsyncGenerator

@strawberry.type
class Message:
    id: str
    content: str

@strawberry.type
class Query:
    @strawberry.field
    def hello(self) -> str:
        return "world"

@strawberry.type
class Subscription:
    @strawberry.subscription
    async def messages(self) -> AsyncGenerator[Message, None]:
        # Simula stream de mensagens
        for i in range(5):
            yield Message(id=str(i), content=f"Mensagem {i}")
            await asyncio.sleep(1)

schema = strawberry.Schema(
    query=Query,
    subscription=Subscription
)
```

## Resolvers

### O que são Resolvers?
Resolvers são funções que determinam como os dados serão obtidos ou modificados para cada campo em uma operação GraphQL.

### Tipos de Resolvers
1. **Query Resolvers**: Buscam dados
2. **Mutation Resolvers**: Modificam dados
3. **Subscription Resolvers**: Gerenciam eventos em tempo real
4. **Field Resolvers**: Resolvem campos específicos de tipos

### Exemplo Prático de Resolvers
[![Open Resolvers Example In Deepnote](https://deepnote.com/buttons/launch-in-deepnote.svg)](https://deepnote.com/launch?url=https://gist.github.com/resolvers-example.py)

```python
@strawberry.type
class User:
    id: str
    name: str
    posts: List['Post']

    @strawberry.field
    async def post_count(self) -> int:
        return len(self.posts)

@strawberry.type
class Post:
    id: str
    title: str
    author: User

    @strawberry.field
    async def author_name(self) -> str:
        return self.author.name

@strawberry.type
class Query:
    @strawberry.field
    async def user(self, id: str) -> Optional[User]:
        return await db.get_user(id)

    @strawberry.field
    async def posts(self) -> List[Post]:
        return await db.get_posts()
```

## Implementação Prática

### Sistema de Chat em Tempo Real
Vamos criar um sistema completo de chat usando GraphQL Subscriptions.

#### 1. Servidor
[![Open Server In Deepnote](https://deepnote.com/buttons/launch-in-deepnote.svg)](https://deepnote.com/launch?url=https://gist.github.com/chat-server.py)

```python
import strawberry
from strawberry.asgi import GraphQL
from fastapi import FastAPI
from typing import List, AsyncGenerator
import asyncio
import json

# Tipos
@strawberry.type
class ChatMessage:
    id: str
    user: str
    content: str
    timestamp: str

# Armazenamento em memória
messages: List[ChatMessage] = []
subscribers = []

# Queries
@strawberry.type
class Query:
    @strawberry.field
    def get_messages(self) -> List[ChatMessage]:
        return messages

# Mutations
@strawberry.type
class Mutation:
    @strawberry.mutation
    async def send_message(self, user: str, content: str) -> ChatMessage:
        message = ChatMessage(
            id=str(len(messages)),
            user=user,
            content=content,
            timestamp=datetime.now().isoformat()
        )
        messages.append(message)
        
        # Notifica subscribers
        for subscriber in subscribers:
            await subscriber(message)
        
        return message

# Subscriptions
@strawberry.type
class Subscription:
    @strawberry.subscription
    async def message_added(self) -> AsyncGenerator[ChatMessage, None]:
        queue = asyncio.Queue()
        subscribers.append(queue.put)
        try:
            while True:
                message = await queue.get()
                yield message
        finally:
            subscribers.remove(queue.put)

# Schema e App
schema = strawberry.Schema(
    query=Query,
    mutation=Mutation,
    subscription=Subscription
)

app = FastAPI()
graphql_app = GraphQL(schema)
app.add_route("/graphql", graphql_app)
app.add_websocket_route("/graphql", graphql_app)
```

#### 2. Cliente
[![Open Client In Deepnote](https://deepnote.com/buttons/launch-in-deepnote.svg)](https://deepnote.com/launch?url=https://gist.github.com/chat-client.py)

```python
import asyncio
from gql import gql, Client
from gql.transport.websockets import WebsocketsTransport

# Queries
GET_MESSAGES = gql("""
    query {
        getMessages {
            id
            user
            content
            timestamp
        }
    }
""")

# Mutations
SEND_MESSAGE = gql("""
    mutation($user: String!, $content: String!) {
        sendMessage(user: $user, content: $content) {
            id
            user
            content
            timestamp
        }
    }
""")

# Subscriptions
MESSAGE_SUBSCRIPTION = gql("""
    subscription {
        messageAdded {
            id
            user
            content
            timestamp
        }
    }
""")

async def main():
    # Configurar cliente
    transport = WebsocketsTransport(url='ws://localhost:8000/graphql')
    client = Client(transport=transport, fetch_schema_from_transport=True)
    
    async def listen_messages():
        async for result in client.subscribe(MESSAGE_SUBSCRIPTION):
            print(f"Nova mensagem: {result}")
    
    async def send_message():
        result = await client.execute(SEND_MESSAGE, {
            "user": "TestUser",
            "content": "Hello GraphQL!"
        })
        print(f"Mensagem enviada: {result}")
    
    # Executar operações
    await asyncio.gather(
        listen_messages(),
        send_message()
    )

if __name__ == "__main__":
    asyncio.run(main())
```

### Como Executar

1. Instale as dependências:
```bash
pip install strawberry-graphql fastapi uvicorn gql
```

2. Execute o servidor:
```bash
uvicorn app:app --reload
```

3. Execute o cliente em outro terminal:
```bash
python client.py
```

## 🔍 Conceitos Avançados

### Boas Práticas

1. **Estrutura de Resolvers**
   - Mantenha resolvers simples e focados
   - Use dataloaders para evitar N+1 queries
   - Implemente tratamento de erros adequado

2. **Subscriptions**
   - Gerencie recursos adequadamente
   - Implemente timeouts
   - Use filtros para reduzir dados desnecessários

3. **Performance**
   - Use caching quando apropriado
   - Otimize queries complexas
   - Monitore uso de recursos

## 📝 Exercícios Práticos

1. Adicione autenticação ao sistema de chat
2. Implemente salas de chat privadas
3. Adicione suporte a mensagens com imagens
4. Crie um sistema de notificações

## 🔧 Troubleshooting

- Verifique se o WebSocket está configurado corretamente
- Monitore memória em caso de muitos subscribers
- Implemente reconexão no cliente
- Use logging para debug

## 🚀 Próximos Passos

1. Explore DataLoaders
2. Implemente Cache
3. Adicione Testes
4. Deploy em Produção

## 📚 Recursos Adicionais

- [Documentação Strawberry](https://strawberry.rocks)
- [GraphQL Python](https://graphql-python.readthedocs.io)
- [FastAPI](https://fastapi.tiangolo.com)
- [GraphQL Spec](https://spec.graphql.org)
