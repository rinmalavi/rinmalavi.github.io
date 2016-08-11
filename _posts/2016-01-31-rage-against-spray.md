---
title: Rage Against Spray
layout: post
---

It's not a rage session, just a love/hate relationship.

[demo](https://github.com/spray/spray/blob/master/examples/spray-can/simple-http-server/src/main/scala/spray/examples)

### Minimum code

in build.sbt

```scala
"io.spray" %% "spray-routing" % "1.3.3",
"io.spray" %% "spray-can" % "1.3.3",
"com.typesafe.akka" %% "akka-actor" % "2.4.1"  //2.3.3 ?
```

Actor

```scala
class YourActor extends HttpService with Actor {
  def actorRefFactory = context
  def receive = runRoute(
    complete{
        "Go home world!"
    })
}
```

Server

```scala
object Boot extends App{
  private val system: ActorSystem = ActorSystem("system")
  /* could be implicit */
  private val actorRef: ActorRef =
    system.actorOf(Props(classOf[YourActor]))
  IO(Http)(system) ! Http.Bind(
    actorRef,
    interface = "localhost",
    port = 7771)
}
```

#### Test
`"io.spray" %% "spray-testkit"  % "1.3.3" % "test"`

[specs2](https://etorreborre.github.io/specs2/)
`"org.specs2" %% "specs2-core" % "2.4.17" % "test"`

```scala
class Spec
    extends org.spec2.Specification
    with Directives {
  "Spec group " >> {
    "Spec" >> {
      Get("/nowhere") ~>
        { _ => complete {"ok"}} ~>
        check {
          response.status.isSuccess must beTrue
        }
    }
  }
}
```

[scalatest](http://www.scalatest.org/user_guide/selecting_a_style)
`"org.scalatest" %% "scalatest" % "2.2.6" % "test"`

```scala
class Spec
    extends FreeSpec
    with Matchers
    with Directives
    with ScalatestRouteTest {
  "Spec Group" - {
    "spec" in {
      Get("/ping") ~>
        complete{"ok"} ~>
          check{
            response.status.isSuccess shouldBe (true)
          }
    }
  }
}
```

Tested here is the actual testing.
You would, usually, test your routes (say they are called `someRoutes`)
by writing `~> someRoutes` instead of that `complete` up there.

To test response content use `responseAs` (requires `Unmarshaller` for a given type in the scope):

```scala
responseAs[String] === "ok"
```

#### Continuous Redeployment

is accomplished with the plugin called [`sbt-revolver`](https://github.com/spray/sbt-revolver).
It will recompile and redeploy on every code change.

```scala
addSbtPlugin("io.spray" % "sbt-revolver" % "0.8.0")
```

#### fork in run := true

[Forking jvm](http://www.scala-sbt.org/0.13.5/docs/Detailed-Topics/Forking.html)
[when running spray](http://www.scala-sbt.org/0.13.5/docs/Detailed-Topics/Running-Project-Code.html#user-code)

also, to forward `stdin` to a forked process add

```scala
connectInput in run := true
```

### Directives

`complete` attempts to transform the result you provided to `HttpResponse`

`pathPrefix` filters paths based on their prefix

`path` attempts to match a complete remaining path

`Segment` matches any segment

`unmatchedPath` provides you with the remaining unmatched path to `complete` with

`get` `post` `put` `delete` matches with the method

`parameter``(s)` extract query parameters from the path

`entity(as[SomeClass])` attempts to unmarshall an incoming request as an instance of `SomeClass`

`getFromBrowseableDirectory` complete with given directorty

`detach`! detaches an execution of your code from neverendnessness...

#### Simple example

```scala
pathPrefix("service") {
  path("entity" / Segment) { id =>
    get {
      complete {
        s"you requested $id"
      }
    }
  } ~
    path("entity") {
      get {
        parameters("id") { mId =>
          complete {
            s"your query id is $id"
          }
        }
      }
    }
  }
```

#### Serve index page

```scala
pathEndOrSingleSlash{
  getFromResource("index.html")
} ~ pathPrefix("assets"){
  getFromResourceDirectory("assets")
}
```

### Marshaling

Add an implicit Marshaller/Unmarshaller to the scope

```scala
trait Marshallers {
  private val ct = spray.http.ContentTypeRange.*
  protected val jsonSerialization: Serialization
  implicit val msUnmarshaller = Unmarshaller[MessageType](ct) {
    case HttpEntity.NonEmpty(contentType, data) =>
      jsonSerialization
        .deserialize[MessageType](data.asString)
  }
}
```

### Authentification

define:

```scala
protected def authenticator(
    userPass: Option[UserPass]): Future[Option[String]] =
  Future {
    userPass.flatMap {
      case UserPass("admin", "multipass") => Some("xyz")
      case _ => None
    }
  }
```

then pass it to the `authenticate` directive  passably like this:

```scala
authenticate(BasicAuth(authenticator _, "realm"))
```

### Exceptions

`runRoute` accepts one implicitly

```scala
implicit val divByZeroHandler = ExceptionHandler {
  case _: Throwable =>
    complete(StatusCodes.BadRequest, "Anything could happen!")
}
```

### Rejections

Almost the same as Exceptions

```scala
implicit val totallyMissingHandler = RejectionHandler {
  case Nil /* secret code for path not found */ =>
    complete(StatusCodes.NotFound,
        "Oh man, what you are looking for is long gone.")
}
```

## SSL
Cannot support TLS_RSA_WITH_AES_256_CBC_SHA with currently installed providers

[doc](http://spray.io/documentation/1.2.3/spray-can/http-server/#ssl-support)
[jce](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)


## Scala Perls

[magnet](http://spray.io/blog/2012-12-13-the-magnet-pattern/)

[type classes](http://ropas.snu.ac.kr/~bruno/papers/TypeClasses.pdf)

[type classes](http://danielwestheide.com/blog/2013/02/06/the-neophytes-guide-to-scala-part-12-type-classes.html)


## Apendex A - Spray Client

```scala
IO(Http) ! HttpRequest(POST,
      Uri(somePath),
      entity = HttpEntity(someJson))
```

Path can be constructed with `Uri.Path`

There are also shorthands for `Get`, `Post`, `Put`. Query params are added with the `withQuery` method on `Uri`.
You may have already noticed this is used in the test example above.

## Apendex B - Debugging

#### FromRequestUnmarshaller

When you get

    could not find implicit value for parameter um:
        spray.httpx.unmarshalling.FromRequestUnmarshaller

but you already have

```scala
implicit val yourStupidClassUnmarshaller =
    Unmarshaller[IrrelevantEntity]() {
      case HttpEntity.NonEmpty(contentType, data) =>
        superObjectHandler.deserialize[IrrelevantEntity](data)
    }
```

If you have everything in one file make sure that you define an implicit before its usage, even if its in a different trait.

#### Request was not handled (in spec)

But the path seems fine

    try adding the / at start of the path ...


### Following is the collection of unknowns, just work around it...

#### Class not found?

    CAUSED BY java.lang.ClassNotFoundException: spray.testkit.RouteTest

When trying to test authorization.

#### Way out of bounds (jackson)

    ArrayIndexOutOfBoundsException: : 59704  (BytecodeReadingParanamer.java:452)


```
[error]  found   : spray.routing.directives.NameReceptacle[the.Clazz]
[error]  required: spray.routing.directives.ParamDefMagnet
```

##### Also to remove margin when pasting multiline string

editor -> Code Style -> multi-line strings -> Add/Insert margin ....