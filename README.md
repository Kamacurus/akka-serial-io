# akka-serial-io

Akka IO library in scala for serial ports based on jSSC. Modified to use a 'built-in' copy of jssc from kamacurus/java-simple-serial-connector which 
includes support for arm64, missing from release version (2.8.0).


## Requirements

- Java 1.8
- Akka 2.5.4

## Usage

This is an un-released package:

    //libraryDependencies += "com.github.akileev" %% "akka-serial-io" % "1.0.2"
    val serialLib = ProjectRef(uri("git://github.com/Kamacurus/akka-serial-io.git"), "akka-serial-io")

    .dependsOn(serialLib)

To connect use (in an actor):

    override def preStart = IO(Serial) ! Open(port, 9600) 
    override def receive = {
      case Opened(p) =>
        println("Connected to port")
        context become open(sender())
    }

The actor will be bound to the opened serial port connection using a death pact. So if the actor that opened the connection gets stopped/killed it will also close and stop the serial port connection.

The operator accepts

-   Write: To write data to the serial port (a ByteString).

-   Close: To close the serial port.

```
    operator ! Write(ByteString(“Hello”)) // no ack

    val ackToken = new AckEvent{}
    operator ! Write(ByteString(“Hello with ack”), ackToken)

    operator ! Close
```

The operator will send

-   Received: Data was received no the serial port (ByteString).

-   Ack: Response to successful write.

-   Closed: When the port has been closed (i.e. as a response to Close).

messages. To handle use code like:

```
    case Received(data) =>
      print("Received data from serial port: ")
      println(data.toString)

    case Closed =>
      println("Serial port closed")
      context stop self
```

For more information see the [Akka IO Documentation][].

## References

For jSSC see: <https://github.com/scream3r/java-simple-serial-connector/>

Provides serial input/output capabilities.

For Akka IO see: <http://doc.akka.io/docs/akka/snapshot/scala/io.html>

Provides the basic interface definition.

  [Akka IO Documentation]: http://doc.akka.io/docs/akka/snapshot/scala/io.html
