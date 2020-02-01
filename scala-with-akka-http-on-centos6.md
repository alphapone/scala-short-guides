# scala-short-guides

## How to install scala with akka http with all dependecies on centos6.0 vpd

*  Install JDK 1.8:
```
yum install java-1.8.0-openjdk
```

* Download scala 2.13:
```
curl https://downloads.lightbend.com/scala/2.13.1/scala-2.13.1.tgz -o scala-2.13.1.tgz
```

* Untar and copy to target directory:
```
tar xvf scala-2.13.1.tgz
mv scala-2.13.1/ /usr/lib
rm scala-2.13.1.tgz
ln -s /usr/lib/scala-2.13.1 /usr/lib/scala
ln -s /usr/lib/scala/bin/scala /etc/alternatives/scala
ln -s /etc/alternatives/scala /usr/bin/scala
```

Now you can run scala 2.13 simply type _scala_ in command line.

* Get akka httpd jars.
Yes, you can use sbt... But command line with installed libraries is greater than more usable.
So simple download and copy them...
```
curl https://repo1.maven.org/maven2/com/typesafe/akka/akka-http_2.13/10.1.11/akka-http_2.13-10.1.11.jar -o /usr/lib/scala/lib/akka-http_2.13-10.1.11.jar 
curl https://repo1.maven.org/maven2/com/typesafe/akka/akka-http-core_2.13/10.1.11/akka-http-core_2.13-10.1.11.jar -o /usr/lib/scala/lib/akka-http-core_2.13-10.1.11.jar
curl https://repo1.maven.org/maven2/com/typesafe/akka/akka-stream_2.13/2.6.3/akka-stream_2.13-2.6.3.jar -o /usr/lib/scala/lib/akka-stream_2.13-2.6.3.jar
curl https://repo1.maven.org/maven2/com/typesafe/akka/akka-actor_2.13/2.6.3/akka-actor_2.13-2.6.3.jar -o /usr/lib/scala/lib/akka-actor_2.13-2.6.3.jar
curl https://repo1.maven.org/maven2/org/scala-lang/modules/scala-java8-compat_2.13/0.9.0/scala-java8-compat_2.13-0.9.0.jar -o /usr/lib/scala/lib/scala-java8-compat_2.13-0.9.0.jar
curl https://repo1.maven.org/maven2/com/typesafe/akka/akka-parsing_2.13/10.1.11/akka-parsing_2.13-10.1.11.jar -o /usr/lib/scala/lib/akka-parsing_2.13-10.1.11.jar
curl https://repo1.maven.org/maven2/com/typesafe/ssl-config-core_2.13/0.4.1/ssl-config-core_2.13-0.4.1.jar -o /usr/lib/scala/lib/ssl-config-core_2.13-0.4.1.jar
curl https://repo1.maven.org/maven2/org/scala-lang/modules/scala-parser-combinators_2.13/1.1.2/scala-parser-combinators_2.13-1.1.2.jar -o /usr/lib/scala/lib/scala-parser-combinators_2.13-1.1.2.jar
```
And now you can write any scripts in scala!

Test it:
```
cat server.scala << END
#!/usr/bin/scala
!#

import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.http.scaladsl.model._
import akka.http.scaladsl.server.Directives._
import akka.stream.ActorMaterializer
import scala.io.StdIn

object WebServer {
  def main(args: Array[String]) = {
    implicit val system = ActorSystem("my-system")
    implicit val materializer = ActorMaterializer()
    // needed for the future flatMap/onComplete in the end
    implicit val executionContext = system.dispatcher
    
    val route =
      path("hello") {
        get {
          complete(HttpEntity(ContentTypes.`text/html(UTF-8)`, "<h1>Say hello to akka-http</h1>"))
        }
      }

    val bindingFuture = Http().bindAndHandle(route, "0.0.0.0", 8080)

    println(s"Server online at http://localhost:8080/\nPress RETURN to stop...")
    StdIn.readLine() // let it run until user presses return
    bindingFuture
      .flatMap(_.unbind()) // trigger unbinding from the port
      .onComplete(_ => system.terminate()) // and shutdown when done
  }
}
END
chmod u+x server.scala
./server.scala
```
## What is a purpose of this?
You can run efficient asynchronous akka server on any very cheap vds.
So you can develop and test any proxy integration or webhook consumer server 
by small simple script typing without resource intensive development in IDE or sbt.
It is very usefull for ideas testing.

## Do you interested in more professional scala development and support?
Contact to sinergo.ru team then.

