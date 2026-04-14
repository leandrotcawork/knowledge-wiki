<!-- source_url: https://docs.python.org/3/library/contextlib.html -->
# contextlib — Utilities for with-statement contexts — Python 3.14.4 documentation
Source: https://docs.python.org/3/library/contextlib.html

## Key Utilities
- `AbstractAsyncContextManager`: ABC for `__aenter__` and `__aexit__`.
- `@asynccontextmanager`: Decorator for async generator functions.
- `aclosing(thing)`: Calls `aclose()` on exit.
- `AsyncExitStack`: Combines multiple sync/async context managers.

## @asynccontextmanager
Must be applied to an async generator. Yields exactly once.
Example:
```python
@asynccontextmanager
async def get_connection():
    conn = await acquire_db_connection()
    try:
        yield conn
    finally:
        await release_db_connection(conn)
```
