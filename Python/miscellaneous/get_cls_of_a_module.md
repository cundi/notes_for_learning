```python
>>> import inspect
>>> import my_module
>>> [m[0] for m in inspect.getmembers(my_module, inspect.isclass) if m[1].__module__ == 'my_module']
```