<!-- source_url: https://superfastpython.com/asynchronous-context-manager/ -->
# Asynchronous Context Managers in Python – SuperFastPython
Source: https://superfastpython.com/asynchronous-context-manager/

## Definition
An object implementing `__aenter__()` and `__aexit__()` as coroutines.

## Manual Usage vs async with
`async with` automatically awaits `__aenter__` and `__aexit__`.
Manual:
```python
manager = AsyncContextManager()
await manager.__aenter__()
try:
    # body
finally:
    await manager.__aexit__(None, None, None)
```

## Exception Handling
`__aexit__` receives `(exc_type, exc_val, tb)`. If it returns `True`, the exception is suppressed.
