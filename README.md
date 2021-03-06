## Model4s
Library for generation boilerplate-free code for models in compile time with Scala macro

[![Build Status](https://api.travis-ci.org/model4s/model4s.png)](https://travis-ci.org/model4s/model4s/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/scala-jedis/model4s/blob/master/LICENSE)

Scala macros to generate oft-repeated models with annotations and convenient `case class` conversion.

Usage:
````
libraryDependencies += "io.github.model4s" %% "model4s" % "1.0"
````

Suppose you have the next model `Person`:
```scala
case class Person(id:Long, name:String, email:String, goodsAmount: Int)
```

But, now you want to create well-formed models to describe something like `DTO` within 
microservices or model for `DAO`, both of them contains fields subsets of initial model `Person`:

```scala
case class PersonDAO(id:Long, name:String, email:String)

case class PersonDTO(name:String, email:String, goodsAmount: Int)
```

Model4s solves this in the next way:

```scala
import com.github.model4s.modelfree.Base._
import com.github.model4s.modelfree.ModelFree

@ModelFree case class Person(
  @dao      id : Int,
  @dao @dto name : String,
  @dao @dto email : String,
       @dto goodsAmount: Int
)
```

Annotation `ModelFree` is a marker for single point of `base model` configuration,
you cat treat `Person` class as full join of model for code generation, in this situation it's `Dao` and `Dto`.
It will generate companion object for class `Person` with nested case classes for models, so you can call it in 
the next way: `Person.Dao` and `Person.Dto`.

After business layer/DAO/DTO moving towards to the gateway of your App - `HTTP REST APIs`, now you want to describe
entities for serving HTTP requests/responses:

```scala
case class PersonGet(id:Long, email:String)

case class PersonPost(name:String, email:String)

case class PersonPut(id:Long, name:String, email:String)
```

That is a lot of boilerplate! Maintaining all of this models quickly becomes tedious for more complicated models.

With Model4s, you can easily reduce all this 6 models to the single point:

```scala
@ModelFree case class Person(
  @dao      @get       @put   id : Int,
  @dao @dto      @post @put   name : String,
  @dao @dto @get @post @put   email : String,
       @dto                   goodsAmount: Int
)
```

### Microservices

If you use microservice architecture possibly you can duplicate business model in different part of services
like HTTP based entity like e.g. `Gateway Service` use `User` model for auth/registration and send it to 
`Registration` service and further in business flow to `UserStatistic` service, 3 services serve same User model,
Model4s allows to get rid of this, you have the single point of configuration which can be wrapped up in reusable
submodule/jar/war file etc.

### Case Class <-> Map transformation
Model4s allows to make transformation between `map` and `case classes` seamlessly, it wraps `macro` into 
twitter's `Bijection` type.

Let's suppose, you have the next input:

```scala
case class User(id: Int, name: String)

val samplePerson = User(13, "Steve")
val sampleMap = Map("id" -> 15, "name" -> "Bill")
```

Model4s handles it in the next way with `Mappable`:

```scala
val map = Mappable.transform[User].apply(samplePerson)

val user = Mappable.transform[User].invert(sampleMap)
```

`Mappable` is wrapper over twitter's `Bijection`, to be more precise: `Bijection.build[T, CaseClassMap]`

## Intellij Idea support
With default settings Intellij doesn't support Scala's macros, it can be enabled in settings:
 
```Build, Execution, Deployment -> Compiler -> Scala Compiler -> Scala Compiler Server -> enable "Macros" tab``` 

**License - MIT**