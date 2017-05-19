# Dependency Injector

Dependency injection microframework for Python

Dependency Injector framework key features are:  

* Easy, smart, pythonic style
* Obvious, clear structure


## Dependency Injector structure
Dependency Injector는 매우 간단한 구조를 가짐

두 개의 main entitis: **providers & container**

![structure](https://raw.githubusercontent.com/wiki/ets-labs/python-dependency-injector/img/internals.png)

providers와 containers는 Cython을 이용해 C 확장 타입으로 구현됨

### Providers
Providers are strategies of accessing objects.

* Factory - 매번 호출될 때마다 새로운 instance를 생성해서 반환함
* Singleton - 최초 호출 시에만 instance를 생성하고, 이후부터는 생성된 instance를 반환함
* Callable - 호출될 때마다 wrapping된 callable을 호출함

Providers could be:

* Injected into each other.
* Overridden by each other.
* Extended.

### Containers
Containers are collections of providers.

Provider들을 grouping하는데 사용됨

* DeclarativeContainer - 선언적인 방식으로 정의되는 dependency injection container
* DynamicContainer - 동적인 구조의 dependency injection container


### Reference
* [dependency_injector](https://pypi.python.org/pypi/dependency_injector/)