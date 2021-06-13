---
layout: post
title: "Facade Pattern in Kotlin"
date:  "2021-06-13 11:43"
description: "
The facade allows you to hide the details of the module from clients. It ensures compliance with `Law Demeter`. Using the generic interface and various implementations greatly simplifies testing. It blends well with other patterns like `Strategy`,` Template Method`, or construction patterns, allowing configuration of the object available for the clients. The facade is a good entry point for libraries, giving customers access to high-level functionality and hiding all internal logic and classes.
"
permalink: "kotlin-facade-pattern"
comments: true
toc: true
tags:
- design patterns
- Kotlin
- Facade Pattern
- structural design pattern

categories:
- Design Patterns
- rss

image: /assets/posts/facade.jpg

---

# Purpose
The facade is a very basic pattern, whose task is to obscure the details of a group of classes - the module responsible for some functionality.

This can be compared to the facade of the building, which in itself has no function. The building consists of rooms, corridors, stairs, installations. It is the facade that indicates the entrance to the building, and the appearance may suggest its purpose.

The purpose of the pattern is to simplify customer access to the functionality of the obscured module. So that they don't have to know implementation details, class dependencies, which version of the object to inject, etc.

By `module` I will understand here some consistent and relatively independent parts of the system. Not necessarily a separate module in the project or a library - it can be, for example, a separate package.

# Implementation
The facade is basically single class. In the diagram it looks like this:

{% plantuml %}
@startuml
skinparam groupComposition 1
hide members
show Facade methods
class ClientA
class ClientB
  
ClientA-down-Facade
ClientB-down-Facade

package module <<Node>> {

class Facade{
	+ operation()
  }

class A
class B
class C
class D
class E
class F

A-down-*B
B-*C
D-*A
A-*F
D-*B
}

F-up->Facade
E-up->Facade
C-up->Facade

@enduml
{% endplantuml %}

The classes `A, B, C, D, E, F` together form a module and provide some functionality. The facade is the access point to this functionality. As a result, no customer need not know the details of the module, to know how to create class instances and how to use them. Customers only need to know the facade, which can build a tree of dependencies for all classes, hide their implementations and provide a simple API for customers.

I was wondering if the `Facade` class shouldn't be inside a module. Such use of the facades is often found in libraries - they have a lot of internal logic, but provide only a simple API inside a single class and, if necessary, several configuration objects. However, Facade can be used to cover code that we have no control over, creating a so-called `Anticorruption Layer`. Then the code that changes behind it does not spoil the clients.

## Abstract
Let's assume the following situation: a client class needs the result of a method from one class, but this method requires the result of another class, which has several dependencies.

```kotlin
val result = E().finishTheWork( // I want this result
        D().doAnotherPart( // but first I need to feed it with this one
            A( // that requires A
            	B( // that requires B
                	C() // that uses C
                )
			).doPartOfWork() // to yield the mid-result
        )
	)

// module classes look like that:
class A(val b: B) {
    fun doPartOfWork() = "result"
}

class B(val c: C)
class C
class D {
    fun doAnotherPart(input: Any) {}
}

class E {
	// to simplify this example I used `Any` and I return `String`
	// but this can be any domain object etc.
    fun finishTheWork(input: Any) = "finish"
}
```

The result of the method from class `E` is available only after building the entire dependency tree of class `A`, calling the appropriate method, the result of which is passed to `D.doAnotherPart()`, the result of which is needed in the method `E.finishTheWork()`. To use this code, you need to know quite a bit of module detail, which probably necessitates the creation and maintenance (!!!) of documentation.

So how can Facade help here?
```kotlin
// calling the same logic from Facade
val result2 = Facade().complexWork() // lovely isn't it?

class Facade() {
    // create dependencies internally
    // can be delegated to DI framework
    // but shouldn't be expected from module client to use the same framework, or be aware of it
    private val c = C()
    private val b = B(c)
    private val a: A = A(b)
    private val d = D()
    private val e = E()

    fun complexWork(): Any { 
		// reminds a bit of Template Method
		// or a Strategy, if instances would be injected
		// nothing is stopping Facade to use other patterns internally
		// for the client it won't matter after all
        val firstResult = a.doPartOfWork()
        val secondResult = d.doAnotherPart(firstResult)
        return e.finishTheWork(secondResult)
    }
}
```

### Low of Demeter
In other words, "the principle of minimum knowledge" or the "rule of limiting interactions". **The idea is for classes to use only their methods or fields, and objects created by themselves or passed directly in method parameters**. The facade perfectly helps to keep this good practice.
```kotlin
val a = A(B(C(D())))
// if you need to use method from class D
// you should have direct access to the instance of D
// not chained object field calls like that
// also private fields wouldn't allow this, but public getters would
val result = a.b.c.d.theMethod()
// creating 'theMethod' call in each class and passing it to class A is also bad
// it means probably A is doing too much, and all those classes are coupled
```
Looking at the previous example, without Facade, the client had to "know" a lot about the dependencies that had to be met to get the expected result. All these details were then hidden behind the Facade. In a way, the client may now say "I need the result, I don't care how you get it."

The downside of strictly applying this rule can be creating multiple methods that only delegate method calls to other internal objects. But if a class does have many delegating methods, maybe it does too much and should be broken down into smaller, more detailed classes?

## Repository
A special case of Facade is the Repository. It allows access to domain objects, hiding the details of their storage. The `Repository` can hide the database, cache, or even communicate with a remote server. Or all of these things at once as in the example below:

```kotlin
// clients care only about those 2 things:
data class User(val id: UUID, val name: String)
interface UserRepository {
    fun getUser(id: UUID): User
}

// `Fake` because DB is in memory, and API returns hardcoded results
// repository variant to use in unit tests for example
class FakeUserRepo(
    private val userDb: UserDb = InMemoryUserDb(),
    private val userApi: UserApi = FakeUserApi(),
    private val userCache: UserCache = SimpleCache()
) : UserRepository {

    override fun getUser(id: UUID): User {
        // check if the object is in the cache
        val cashedUser = userCache.get(id)
        if (cashedUser == null) {
            // if not, check database
            val dbUser = userDb.get(id)
            if (dbUser == null) {
                // if not, check on remote server
                val userDto = userApi.get(id)
                return userDto.toUser().also {
                    // after getting the object, put it in DB and cache
                    userDb.add(it.toEntity())
                    userCache.add(it)
                }
            } else {
                // if object is in DB, return it and put it in cache
                return dbUser.toUser().also {
                    userCache.add(it)
                }
            }
        } else {
            return cashedUser
        }
    }

	// useful extension functions, mapping entity and DTO to domain class
    private fun UserDto.toUser(): User {
        return User(this.id, this.name)
    }

    private fun UserEntity.toUser(): User {
        return User(this.id, this.name)
    }

    private fun User.toEntity(): UserEntity {
        return UserEntity(this.id, this.name)
    }
}
```
Pay attention to the object interfaces. Both `UserRepository` and` UserDb`, `UserApi` and` UserCache` may have different implementations, but it doesn't matter for the client.
```kotlin
// UserRepository instance could be injected instead of created here
// then for testing you could use instance without real DB and access to HTTP client
val userRepo: UserRepository = FakeUserRepo()
// Repository is a Facade for overall data access
// client doesn't have to bother with cache, API call, DB etc.
val user = userRepo.getUser(UUID.randomUUID())
```
The repository takes care of storing objects in the cache or filling in the gaps in the local database with information from a remote server. The customer gets only the data in the fastest way possible. This approach makes testing incredibly easy, especially when you use interfaces and dependency injection. It also allows for the parallelization of the work of several people, where one deals with, for example, the UI layer displaying data, and the other creates its storage and access via HTTP or cache policy. It is especially interesting in interdisciplinary teams, where a mobile programmer can deal with the UI, and the backend one can deal with the data access layer in the application, without any knowledge of e.g. Android.

# Naming
As in [the previous post](https://asvid.github.io/kotlin-strategy-pattern#naming), I rather favor not adding "Facade" to the class name, which is a facade. This is seen in the example of the 'Repository', there is no need to inform customers that they are only dealing with the facade. Clients want it to perform an action, provided by the class, they do not need to know whether it is a facade. This is different for example for the [Builder Pattern](https://asvid.github.io/kotlin-builder-pattern), which by definition should have the `build()` method. The facade does not have API enforced by the pattern itself.

# Summary
The facade allows you to hide the details of the module from clients. It ensures compliance with `Law Demeter`. Using the generic interface and various implementations greatly simplifies testing. It blends well with other patterns like `Strategy`,` Template Method`, or construction patterns, allowing configuration of the object available for the clients. The facade is a good entry point for libraries, giving customers access to high-level functionality and hiding all internal logic and classes.

## Pros
- **simple interface for clients** - Facade provides minimal module interface, instead of expecting implementation details knowledge from clients, that should be documented and maintained
- **Law of Demeter compliant** - the clients "talk" only with a Facade, not with internal module classes. Clients don't have to even know them or their details.
- **testing** - switching the Facade instance from production to test variant can simulate the whole module, so clients can be tested in isolated way.
- **refactoring** - clients are loosely couppled with the module, they only now the Facade so all refactoring changes won't affect them. Think about it as of interior design change but leaving the outside structure of the building untouched.
- **control over what client knows** - useful in case of libraries, where you may not want to expose all internal classes to the clients, but only a consciously selected part.
## Cons
- **limiting clients options** - if the client needs access objects from inside the module, Facade may not allow it. Changes in customer requirements can lead to the addition of new methods in the Facade until it is no longer just a "simple interface". This may not be a big problem if you have control over the facade, and the module it obscures. If Facade is part of an external library, the constraint may take away the sense of using the library or force a hacky extraction of objects from inside the module.