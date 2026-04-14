domain: domains/python/async
confidence: medium
sources: 3
last_updated: 2026-04-14

# Async Context Managers em Python

**Async Context Managers** são a base do gerenciamento seguro de recursos assíncronos em Python. Introduzidos na PEP 492 (junto com `async`/`await`), eles permitem a alocação e liberação de recursos que dependem de operações de I/O (como conexões de banco de dados, sockets de rede ou sessões HTTP) sem bloquear o *Event Loop*.

Eles são consumidos através da sintaxe `async with` e implementam o protocolo de gerenciamento de contexto assíncrono através dos *dunder methods* `__aenter__` e `__aexit__`.

---

## 1. Anatomia e Funcionamento Interno

Um Async Context Manager é qualquer objeto que implemente os métodos `__aenter__()` e `__aexit__()` como **corrotinas** (ou seja, funções definidas com `async def` ou que retornem um *awaitable*).

### O Desugaring do `async with`

Quando o interpretador Python encontra um bloco `async with`, ele essencialmente traduz a sintaxe para um bloco `try/finally` assíncrono. 

```python
# Sintaxe de alto nível
async with AsyncResource() as resource:
    await resource.do_something()
```

Por baixo dos panos, o Python executa o equivalente a:

```python
# Representação interna (Desugaring)
manager = AsyncResource()
resource = await manager.__aenter__()
try:
    await resource.do_something()
finally:
    # O Python passa os detalhes da exceção, se houver
    # exc_type, exc_val, traceback
    await manager.__aexit__(None, None, None)
```

### Ciclo de Vida (Diagrama de Fluxo)

```text
+-----------------------+
|  Início: async with   |
+-----------+-----------+
            |
            v
+-----------------------+
| await __aenter__()    | ---> Adquire recurso (I/O bound, ex: TCP handshake)
+-----------+-----------+
            |
            v
+-----------------------+
| Executa o bloco       | ---> (Corpo do async with)
| interno (yield)       |
+-----------+-----------+
            |
            v
+-----------------------+      Exceção ocorreu?
| await __aexit__()     | <-------------------------+
+-----------+-----------+                           |
            |                                       |
            v                                       |
+-----------------------+                           |
| Libera recurso        | ---> Fecha socket, devolve conexão ao pool, etc.
+-----------------------+
```

---

## 2. Padrões de Implementação

Existem duas formas principais de criar um Async Context Manager: baseada em classes e baseada em geradores (usando `contextlib`).

### 2.1. Implementação Baseada em Classes

Ideal para gerenciadores de estado complexos onde você precisa manter variáveis de instância ou expor múltiplos métodos.

```python
import asyncio

class AsyncDatabaseConnection:
    def __init__(self, dsn: str):
        self.dsn = dsn
        self.conn = None

    async def __aenter__(self):
        print(f"Conectando ao banco: {self.dsn}...")
        await asyncio.sleep(0.1)  # Simulando I/O
        self.conn = "Conexão_Ativa"
        return self.conn

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Fechando conexão...")
        await asyncio.sleep(0.1)  # Simulando I/O de teardown
        self.conn = None
        
        # Se retornar True, a exceção é suprimida.
        # Se retornar False (ou None), a exceção é propagada.
        if exc_type is ConnectionError:
            print(f"Suprimindo erro de conexão: {exc_val}")
            return True
```

### 2.2. Implementação Baseada em Geradores (`@asynccontextmanager`)

Para a grande maioria dos casos de uso, a abordagem baseada em classes é verbosa demais. O módulo `contextlib` fornece o decorador `@asynccontextmanager`, que transforma um gerador assíncrono (`async def` com `yield`) em um Async Context Manager.

Esta é a abordagem recomendada por ser mais limpa e encapsular o estado localmente na função.

```python
from contextlib import asynccontextmanager
import asyncio

@asynccontextmanager
async def async_file_writer(filename: str):
    print(f"Abrindo {filename} para escrita...")
    file_obj = await open_async_file(filename) # Pseudo-código
    try:
        # O valor yieldado é o que vai para a variável após o 'as'
        yield file_obj
    except IOError as e:
        print(f"Erro de I/O capturado durante a escrita: {e}")
        raise
    finally:
        print(f"Fechando {filename}...")
        await file_obj.close()
```

**Regras de Ouro para `@asynccontextmanager`:**
1. A função **deve** ser um gerador assíncrono (`async def` + `yield`).
2. Deve fazer `yield` **exatamente uma vez**.
3. Tudo antes do `yield` é o `__aenter__`.
4. Tudo depois do `yield` (geralmente num bloco `finally`) é o `__aexit__`.

---

## 3. Casos de Uso no Mundo Real (Stack: FastAPI + SQLAlchemy)

### 3.1. Gerenciamento de Sessão de Banco de Dados

Em aplicações modernas com FastAPI e SQLAlchemy assíncrono (`ext.asyncio`), injetar sessões de banco de dados seguras é um caso de uso clássico para Async Context Managers.

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from contextlib import asynccontextmanager

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

@asynccontextmanager
async def get_db_session():
    """Fornece uma sessão transacional segura."""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        # O fechamento da sessão é garantido pelo 'async with AsyncSessionLocal()'
```

### 3.2. Lifespan do FastAPI

O FastAPI substituiu os antigos eventos `@app.on_event("startup")` por um único Async Context Manager chamado `lifespan`. Isso garante que o teardown ocorra de forma confiável, mesmo se a inicialização falhar parcialmente.

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
import httpx

# Estado global tipado
ml_models = {}
http_client: httpx.AsyncClient

@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- SETUP (__aenter__) ---
    global http_client
    http_client = httpx.AsyncClient()
    ml_models["classifier"] = await load_model_async()
    print("Aplicação iniciada, recursos alocados.")
    
    yield  # A aplicação roda enquanto o yield está pausado
    
    # --- TEARDOWN (__aexit__) ---
    await http_client.aclose()
    ml_models.clear()
    print("Aplicação encerrada, recursos liberados.")

app = FastAPI(lifespan=lifespan)
```

---

## 4. Utilitários Avançados do `contextlib`

Para engenheiros seniores lidando com orquestração complexa de recursos, o `contextlib` oferece ferramentas essenciais.

### 4.1. `AsyncExitStack`

O `AsyncExitStack` permite gerenciar um número dinâmico de context managers (síncronos ou assíncronos) e garante que todos sejam fechados corretamente na ordem inversa de sua entrada, mesmo se ocorrerem exceções.

**Cenário:** Você precisa abrir N conexões de rede dinamicamente baseadas em uma lista de URLs.

```python
from contextlib import AsyncExitStack
import httpx
import asyncio

async def fetch_all(urls: list[str]):
    async with AsyncExitStack() as stack:
        # Aloca múltiplos recursos dinamicamente
        clients = []
        for _ in urls:
            # enter_async_context empilha o recurso
            client = await stack.enter_async_context(httpx.AsyncClient())
            clients.append(client)
            
        # Se qualquer erro ocorrer aqui, o AsyncExitStack garante
        # que todos os clients que foram abertos com sucesso sejam fechados.
        tasks = [client.get(url) for client, url in zip(clients, urls)]
        responses = await asyncio.gather(*tasks)
        return responses
```

### 4.2. `aclosing`

Semelhante ao `closing` síncrono, o `aclosing` é usado para garantir que o método `aclose()` de um objeto (frequentemente um gerador assíncrono) seja chamado ao final do bloco.

```python
from contextlib import aclosing

async def async_generator():
    try:
        yield 1
        yield 2
    finally:
        print("Limpando gerador...")

async def main():
    # Garante que o gerador seja fechado mesmo se o loop for interrompido
    async with aclosing(async_generator()) as ag:
        async for item in ag:
            if item == 1:
                break # O finally do gerador será executado!
```

---

## 5. Tratamento de Exceções e Supressão

O método `__aexit__` tem o poder de "engolir" (suprimir) exceções que ocorrem dentro do bloco `async with`. 

Para suprimir uma exceção, `__aexit__` deve retornar `True`. Se retornar `False`, `None`, ou não retornar nada, a exceção é propagada para cima.

```python
class SuppressNetworkErrors:
    async def __aenter__(self):
        return self

    async def __aexit__(self, exc_type, exc_val, traceback):
        if exc_type is not None and issubclass(exc_type, ConnectionError):
            # Loga o erro, mas impede que a aplicação quebre
            print(f"Erro de rede suprimido: {exc_val}")
            return True 
        return False # Outros erros (ex: ValueError) irão quebrar a aplicação
```

*Nota: Ao usar `@asynccontextmanager`, você não retorna `True` para suprimir. Em vez disso, você captura a exceção com um bloco `try/except` ao redor do `yield` e simplesmente não a relança (`raise`).*

---

## 6. Pitfalls e Anti-Patterns Comuns

### 6.1. Esquecer o `async` no `with`
Se você usar `with` em vez de `async with` em um Async Context Manager, o Python tentará chamar `__enter__` em vez de `__aenter__`, resultando em um `AttributeError`.

```python
# ERRADO
with get_db_session() as db: # AttributeError: __enter__
    ...

# CORRETO
async with get_db_session() as db:
    ...
```

### 6.2. Bloquear o Event Loop no Setup/Teardown
Um erro comum é usar bibliotecas síncronas dentro de um `__aenter__` ou `__aexit__`. Isso trava todo o Event Loop.

```python
class BadManager:
    async def __aenter__(self):
        import time
        time.sleep(5) # ANTI-PATTERN: Bloqueia o Event Loop!
        return self
```
*Solução:* Sempre use operações I/O assíncronas (ex: `asyncio.sleep`, `aiofiles`, `httpx.AsyncClient`) ou delegue para um ThreadPool via `asyncio.to_thread()` se a biblioteca for estritamente síncrona.

### 6.3. Múltiplos `yield` no `@asynccontextmanager`
Um gerador decorado com `@asynccontextmanager` deve fazer `yield` exatamente uma vez. Se fizer zero ou mais de uma vez, o `contextlib` levantará um `RuntimeError`.

```python
@asynccontextmanager
async def bad_manager():
    yield 1
    yield 2 # RuntimeError: async generator didn't stop after aclose()
```

---

## See Also
- [[python-asyncio-event-loop]]
- [[fastapi-lifespan-events]]
- [[sqlalchemy-async-sessions]]
- [[python-generators-and-coroutines]]

## Sources
- raw/articles/python-async-context-managers-dev-to.md
- raw/articles/python-docs-contextlib.md
- raw/articles/superfastpython-async-context-managers.md