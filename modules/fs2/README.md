# fs2 module for PureConfig

Adds support for loading and saving configurations from [fs2](https://github.com/functional-streams-for-scala/fs2) streams.

## Add pureconfig-fs2 to your project

In addition to [core pureconfig](https://github.com/pureconfig/pureconfig), you'll need:

```scala
libraryDependencies += "com.github.pureconfig" %% "pureconfig-fs2" % "0.9.1"
```

Note that as fs2 only supports Scala 2.11+ this module in only available for those versions.

## Example
### Reading configuration

To load a configuration file from a path using cats-effect's `IO`:




```scala
import pureconfig.module.fs2._
import cats.effect.IO
import fs2.io.file

import scala.concurrent.ExecutionContext.Implicits.global

case class MyConfig(somefield: Int, anotherfield: String)

val chunkSize = 4096

val configStream = file.readAll[IO](somePath, chunkSize)

val load: IO[MyConfig] = streamConfig[IO, MyConfig](configStream)
```

To test that this `IO` does indeed return a `MyConfig` instance:
```scala
//Show the contents of the file
new String(Files.readAllBytes(somePath), StandardCharsets.UTF_8)
// res7: String =
// somefield=1234
// anotherfield=some string

load.unsafeRunSync().equals(MyConfig(1234, "some string"))
// res8: Boolean = true
```

### Writing configuration

To create a byte stream from a configuration:

```scala
import cats.implicits._
import pureconfig.module.fs2._
import fs2.text
import cats.effect.IO

val someConfig = MyConfig(1234, "some string")

val configStream: fs2.Stream[IO, Byte] = saveConfigToStream(someConfig)
```

And to confirm the stream has the values we expect:

```scala
configStream.through(text.utf8Decode).to(fs2.Sink.showLinesStdOut).compile.drain.unsafeRunSync
// {
//     # hardcoded value
//     "anotherfield" : "some string",
//     # hardcoded value
//     "somefield" : 1234
// }
// 
```
