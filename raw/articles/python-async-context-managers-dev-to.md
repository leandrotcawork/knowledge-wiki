<!-- source_url: https://dev.to/hevalhazalkurt/understanding-async-context-managers-in-python-1928 -->
# Understanding Async Context Managers in Python - DEV Community
Source: https://dev.to/hevalhazalkurt/understanding-async-context-managers-in-python-1928

Async context managers are used with `async with` to manage resources like database connections, network sockets, or HTTP sessions asynchronously.

## How they work
They implement `__aenter__` and `__aexit__`. Both are coroutines.

## Examples
- `aiohttp.ClientSession()`
- `asyncpg` connection pools in FastAPI.
- Custom managers using `@contextlib.asynccontextmanager`.

## Pitfalls
- Forgetting `await` (though `async with` handles this).
- Blocking calls inside the block.
- Not using `async with` leading to leaks.
