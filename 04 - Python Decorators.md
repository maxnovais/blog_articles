# Python Decorators
## Decorators com função

### Decorator sem parametro
```
def decorator(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
				**Your code here**
    return wrapper
``` 

### Decorator com parametro
```
def decorator_with_args(<arg1, arg2, (...), argN):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
						**Your code here**
        return wrapper
    return decorator
```