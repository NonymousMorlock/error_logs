## PROBLEM

I was creating a flask application and tried to decorate more than one route with the 
same decorator, and when I start the server, it throws this error

`AssertionError: View function mapping is overwriting an existing endpoint function: closure`

This was my decorator

```python
def admin_only(func: Callable) -> Callable:
    def closure(*args):
        if current_user.id == 1:
            print('User is Admin...Authorized')
            return func(*args)
        print('User is not an Admin...Forbidden Access')
        return abort(403)

    return closure
```

so, initially, I thought to myself, oh, the name of my nested function is `closure`, that must be the
issue, so, I changed the name of the function to `wrapper` instead, but the same error came back
this time with wrapper as the defaulting name

`AssertionError: View function mapping is overwriting an existing endpoint function: wrapper`

## SOLUTION

https://stackoverflow.com/a/42254713/17971158

TLDR; It points out that the issue is using the decorator on more than one route,
since no two routes can have the same name, so, if route 1 was wrapped with this decorator
and route two was also wrapped with it, we have a problem where they both
have the same name. 

The solution is to re-assign the name of the closure as such `closure.__name__ = func.__name__`

```python
def admin_only(func: Callable) -> Callable:
    def closure(*args):
        if current_user.id == 1:
            print('User is Admin...Authorized')
            return func(*args)
        print('User is not an Admin...Forbidden Access')
        return abort(403)

    closure.__name__ = func.__name__
    return closure
```
