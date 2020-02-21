# Our-message-sender
SMS message sender which uses android device pool for SMS sending

A sample for simple but business usefull service in scala

## The code

```
#!/usr/lib/scala-2.13.1/bin/scala
!#

import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.http.scaladsl.model._
import akka.http.scaladsl.server.Directives._
import akka.http.scaladsl.unmarshalling.Unmarshal
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport
import spray.json._
import DefaultJsonProtocol._
import akka.stream.ActorMaterializer
import scala.io.StdIn
import sys.process._

import java.net.URLDecoder

// local sms gateway for sms sending via pool of android devices attached over wifi network using kdeconnect-cli utility and kdeconnect android application
//
// system requirements: kdeconnect installed; 
//
object WebServer {
  var sentInfo = Map[String,String]()

  def isPhone(s:String) = s!=null && (s.startsWith("+79") && s.length == 12) // phone checker. change here if needed
  def prepareString(s:String) = s.replaceAll("'","")

  def main(args: Array[String]) = {
    implicit val system = ActorSystem("our-message-sender")
    // implicit val materializer = ActorMaterializer()
    // needed for the future flatMap/onComplete in the end
    implicit val executionContext = system.dispatcher

    case class SmsToSend(var phoneTo:String, var message:String)
    implicit val smsToSendFormat = jsonFormat2(SmsToSend)

    val route =
      path("sms") {
         get {
            complete(HttpEntity(ContentTypes.`text/html(UTF-8)`, s"<h1>Local simple SMS sender</h1><h2>Following requests sent to our local telephony device</h2>${sentInfo}"))
         } ~
         post {            
            entity(as[String]) { entity => 
              {
                val payload = entity.parseJson.convertTo[SmsToSend]
                sentInfo = sentInfo + (s"**${payload.phoneTo.takeRight(3)}"->payload.message)
                if (isPhone(payload.phoneTo)) {
                  val text = prepareString(payload.message)
                  val phone = prepareString(payload.phoneTo)
                  val d = "kdeconnect-cli -a".!!.split("\n").map(_.split(": ")).filter(_.length==2).map(_(1)).map(_.split(" ")).map(_(0))
                  if (d.length>0) {
                    val id = if (d.length==1) {
                      // get single device id from array
                      d(0) 
                    } else {
                      // get device id from the pool if a lot of devices found
                      val rnd = new scala.util.Random
                      d(rnd.nextInt(d.length-1))
                    }                     
                    val r = s"kdeconnect-cli -d ${id} --send-sms '${text}' --destination ${phone}".!!
                    sentInfo = sentInfo + (s"**${phone.takeRight(3)}"->s"sent to device ${id} - $r")
                  }
                }
                complete(HttpEntity(ContentTypes.`text/html(UTF-8)`, s"Thanks for your payload ${payload}."))
              }
            } 
         }
      }

    val bindingFuture = Http().bindAndHandle(route, "0.0.0.0", 8089)

    println(s"Server online at http://0.0.0.0:8089/\nPress RETURN to stop...")
    StdIn.readLine() // let it run until user presses return
    bindingFuture
      .flatMap(_.unbind()) // trigger unbinding from the port
      .onComplete(_ => system.terminate()) // and shutdown when done
  }
}
```

## Installation

Save the code into our-message-sender.scala file, grant execute permission using `chmod` and run `./our-message-sender.scala`

## Interested in scala or java development?

Write to inl@yandex.com please


