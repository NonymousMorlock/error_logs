## PROBLEM

```md
Bad state: No element
```
This was thrown when running my code generator, and would not tell me exactly what was
causing it, but I found it was from the _paramToString method in the code generator.

```dart
String paramToString(IFunction method, Param param) {
  if(param == method.params!.firstWhere((element) => element.isNamed) &&
      param != method.params!.lastWhere((element) => element.isNamed)) {
    return '{${param.param}';
  } else if(param == method.params!.lastWhere((element) => element.isNamed)
      && param != method.params!.firstWhere((element) => element.isNamed)) {
    return '${param.param},}';
  } else if(param == method.params!.firstWhere((element) => element.isNamed)
      && param == method.params!.lastWhere((element) => element.isNamed)) {
    return '{${param.param}}';
  } else if(param == method.params!.firstWhere((element) => element
      .isOptionalPositional) && param != method.params!.lastWhere((element)
  => element.isOptionalPositional)) {
    return '[${param.param}';
  } else if(param == method.params!.lastWhere((element) => element
      .isOptionalPositional) && param != method.params!.firstWhere(
          (element) => element.isOptionalPositional)) {
    return '${param.param},]';
  } else if(param == method.params!.firstWhere((element) => element
      .isOptionalPositional) && param == method.params!.lastWhere(
          (element) => element.isOptionalPositional)) {
    return '[${param.param}]';
  } else {
    return param.param;
  }
}
```

and the documentation of ```firstWhere``` and ```lastWhere``` says:

```md
If no element satisfies test, the result of invoking the orElse function 
is returned. If orElse is omitted, it defaults to throwing a StateError
```
I wasn't considering the case where there were no named or optional positional, not explicitly at least
because I was checking for it in the `else` statement, but before the code execution
got to the else, it would throw the error. because if it checks for the last or first where there is a named, it wouldn't
see any, which would leave an empty case. 

This was my example repository

```dart
@repoGen
class CreditorRepoTBG {
  external Future<Either<Failure, void>> addCreditor(
    Creditor creditor,
    String token,
  );

  external Future<Either<Failure, List<Creditor>>> getCreditors(
    int branchId,
    String token,
  );
}
```

notice how the parameters are neither named nor optional positional, so it would throw the error.



## SOLUTION

```dart
String paramToString(IFunction method, Param param) {
  if(param == method.params!.firstWhere((element) => element.isNamed, orElse: () => const
  Param.empty(),) &&
      param != method.params!.lastWhere((element) => element.isNamed, orElse: () => const
      Param.empty(),)) {
    return '{${param.param}';
  } else if(param == method.params!.lastWhere((element) => element.isNamed, orElse: () => const
  Param.empty(),)
      && param != method.params!.firstWhere((element) => element.isNamed, orElse: () => const
      Param.empty(),)) {
    return '${param.param},}';
  } else if(param == method.params!.firstWhere((element) => element.isNamed, orElse: () => const
  Param.empty(),)
      && param == method.params!.lastWhere((element) => element.isNamed, orElse: () => const
      Param.empty(),)) {
    return '{${param.param}}';
  } else if(param == method.params!.firstWhere((element) => element
      .isOptionalPositional, orElse: () => const
  Param.empty(),) && param != method.params!.lastWhere((element)
  => element.isOptionalPositional, orElse: () => const
  Param.empty(),)) {
    return '[${param.param}';
  } else if(param == method.params!.lastWhere((element) => element
      .isOptionalPositional, orElse: () => const
  Param.empty(),) && param != method.params!.firstWhere(
          (element) => element.isOptionalPositional, orElse: () => const
  Param.empty(),)) {
    return '${param.param},]';
  } else if(param == method.params!.firstWhere((element) => element
      .isOptionalPositional, orElse: () => const
  Param.empty(),) && param == method.params!.lastWhere(
          (element) => element.isOptionalPositional, orElse: () => const
  Param.empty(),
  )) {
    return '[${param.param}]';
  } else {
    return param.param;
  }
}
```