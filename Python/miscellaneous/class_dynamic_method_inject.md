# 对类绑定动态方法

- 1

```python
class A:
    pass
a = A()
def foo(self): # Have to add self since this will become a method
    print('hello world!')
setattr(A, 'foo', foo)
a.foo() # hello world!
```

- 2

```python
from functools import wraps # This convenience func preserves name and docstring

class A:
    pass

def add_method(cls):
    def decorator(func):
        @wraps(func) 
        def wrapper(self, *args, **kwargs): 
            return func(*args, **kwargs)
        setattr(cls, func.__name__, wrapper)
        # Note we are not binding func, but wrapper which accepts self but does exactly the same as func
        return func # returning func means func can still be used normally
    return decorator

# No trickery. Class A has no methods nor variables.
a = A()
try:
    a.foo()
except AttributeError as ae:
    print(f'Exception caught: {ae}') # 'A' object has no attribute 'foo'

try:
    a.bar('The quick brown fox jumped over the lazy dog.')
except AttributeError as ae:
    print(f'Exception caught: {ae}') # 'A' object has no attribute 'bar'

# Non-decorator way (note the function must accept self)
# def foo(self):
#     print('hello world!')
# setattr(A, 'foo', foo)

# def bar(self, s):
#     print(f'Message: {s}')
# setattr(A, 'bar', bar)

# Decorator can be written to take normal functions and make them methods
@add_method(A)
def foo():
    print('hello world!')

@add_method(A)
def bar(s):
    print(f'Message: {s}')

a.foo()
a.bar('The quick brown fox jumped over the lazy dog.')
print(a.foo) # <bound method foo of <__main__.A object at {ADDRESS}>>
print(a.bar) # <bound method bar of <__main__.A object at {ADDRESS}>>

# foo and bar are still usable as functions
foo()
bar('The quick brown fox jumped over the lazy dog.')
print(foo) # <function foo at {ADDRESS}>
```