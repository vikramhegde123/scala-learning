# Scala Learning

## Collections

List
Map
Set
Tuple
Iterators

## Spark Applications

There is a limitation in Scala: "main" method cannot be found in companion object. So names should be different.

```scala
class SegmentationApplication extends SparkApplication[SegmentConfig] { ... }

object SegmentationApp extends App {
  new SegmentationApplication().main(args)
}
```
