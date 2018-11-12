# mappingproxy
A mappingproxy is simply a dict with no `__setattr__` method. This helps python speed up simple attribute lookups for classes.