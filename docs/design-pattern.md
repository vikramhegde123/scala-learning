#Design Patterns

###SingleTon Pattern
Singletons design patterns, which are supported out of the box by the Scala programming language syntax. We achieve this by using the object keyword

The singleton design pattern ensures that a class has only one object instance in the entire application. 
It introduces a global state in the applications it is used in.

The most common reason for this is to control access to some shared resource. For example, a database or a file

```
object SingleTonDatabase {
  val dbName = "SingleTonDB"
  def getConnection(url: String, userName: String, password: String, dbName: String): Connection = {
    //  return a Connection instance
  }

  def executeQuery(conn: Connection, queryString: String): List[Row] = {
    // return output rows
  }

  def writeToTable(conn: Connection, tableName: String, rows: List[Row]): Unit = {
    // acquire a lock on table and write to table
  }
}
  ```

###Factory Pattern
The factory method design pattern exists in order to encapsulate an actual class instantiation. It simply provides an interface to create an object, and then the subclasses of the factory decide which concrete class to instantiate. 

This design pattern could become useful in cases where we want to create different objects during the runtime of the application

```
trait Connection {  
  def executeQuery(query: String): Unit
}

class MysqlConnection extends Connection {  
  override def executeQuery(query: String): Unit = {
    System.out.println(s"Executing the query '$query' the MySQL way.")
  }
}

class PgSqlConnection extends Connection {
  override def executeQuery(query: String): Unit = {
    System.out.println(s"Executing the query '$query' the PgSQL way.")
  }
}

object FactorySingleTonDatabase {

  val dbName = "DBFactory"
  def getConnection(url: String, dbName: String): Connection = {
    dbName match {
      case "mysql" => new MysqlConnection.
      case "postgres" => new PgSqlConnection
    }
  }

  def runQuery(url: String, queryString: String, dbName: String, query: String): List[Row] = {
    // return output rows
    getConnection(url, dbName).executeQuery(query)
  }

}
```

###Composite Pattern

There are so many cases where we can use composite design pattern. Some of them can be:

* The data needs to be represented in hierarchy or tree structure.
* When we need parent-Child relationship
* When parent and child objects need to be treated in same way
```
  trait File{
    def getType(): String
    def getSize(): Long
  }

  class TextFile(size: Long) extends File{
    override def getType(): String = "txt"
    override def getSize(): Long = size
  }

  class Directory extends File {
    private val files = mutable.ListBuffer[File]()
    override def getType(): String = "directory"

    def addFile(textFile: TextFile)  = {
      files += textFile
      ()
    }

    override def getSize(): Long = {
      files.map(_.getSize()).sum
    }
  }

  val file1 = new TextFile(100)
  val file2 = new TextFile(200)
  val dir = new Directory

  dir.addFile(file1)
  dir.addFile(file2)
  file1.getSize() // 100
  dir.getSize() // 300 
```

###Decorator
The decorator design pattern is all about adding responsibilities to objects dynamically.

The decorator design pattern is about creating a decorator class that can wrap the original class and provides additional functionality, keeping the class methods signature intact

```
trait Coffee {
  def cost:Double
  def ingredients: String
}

class SimpleCoffee extends Coffee {
  override def cost = 1
  override def ingredients = "Coffee"
}

abstract class CoffeeDecorator(decoratedCoffee: Coffee) extends Coffee {
  val sep = ", "

  override def cost = decoratedCoffee.cost
  override def ingredients = decoratedCoffee.ingredients
}

class Milk(decoratedCoffee: Coffee) extends CoffeeDecorator(decoratedCoffee) {
  override def cost = super.cost + 0.5
  override def ingredients = super.ingredients + sep + "Milk"
}

class Whip(decoratedCoffee: Coffee) extends CoffeeDecorator(decoratedCoffee) {
  override def cost = super.cost + 0.7
  override def ingredients = super.ingredients + sep + "Whip"
}

class Sprinkles(decoratedCoffee: Coffee) extends CoffeeDecorator(decoratedCoffee) {
  override def cost = super.cost + 0.2
  override def ingredients = super.ingredients + sep + "Sprinkles"
}

object DecoratorSample {
  def main(args: Array[String]) = {
    var c:Coffee = new SimpleCoffee
    printf("Cost: %f Ingredients %s", c.cost, c.ingredients)
    c = new Milk(c)
    printf("Cost: %f Ingredients %s", c.cost, c.ingredients)
    c = new Sprinkles(c)
    printf("Cost: %f Ingredients %s", c.cost, c.ingredients)
    c = new Whip(c)
    printf("Cost: %f Ingredients %s", c.cost, c.ingredients)
    c = new Sprinkles(c)
    printf("Cost: %f Ingredients %s", c.cost, c.ingredients)
  }
}
```

###Adapter
Adapter pattern works as a bridge between two incompatible interfaces.
It allows the interface of an existing class to be used as another interface.

This pattern involves a single class which is responsible to join functionalities of independent or incompatible interfaces.
```
 trait MediaPackage {
    def playFile(filename: String): Unit
  }

  class MP3 extends MediaPlayer {
    def play(filename: String): Unit = {
      System.out.println("Playing MP3 File " + filename)
    }
  }

  class MP4 extends MediaPackage {
    override def playFile(filename: String): Unit = {
      System.out.println("Playing MP4 File " + filename)
    }
  }

  class VLC extends MediaPackage {
    override def playFile(filename: String): Unit = {
      System.out.println("Playing VLC File " + filename)
    }
  }

  class FormatAdapter(val media: MediaPackage) extends MediaPlayer {
    def play(filename: String): Unit = {
      System.out.print("Using Adapter --> ")
      media.playFile(filename)
    }
  }

  object Main {
    def main(args: Array[String]): Unit = {
      var player: MediaPlayer = new MP3
      player.play("file.mp3")
      player = new FormatAdapter(new MP4)
      player.play("file.mp4")
      player = new FormatAdapter(new VLC)
      player.play("file.avi")
    }
  }

```

###State machine
The purpose of the state design pattern is to allow us to choose a different behavior of an object based on the object's internal state

```
  trait State[T] {
    def press(context: T)
  }

  class Playing extends State[MediaPlayer] {
    override def press(context: MediaPlayer): Unit = {
      System.out.println("Pressing pause.")
      context.setState(new Paused)
    }
  }

  class Paused extends State[MediaPlayer] {
    override def press(context: MediaPlayer): Unit = {
      System.out.println("Pressing play.")
      context.setState(new Playing)
    }
  }

  case class MediaPlayer() {
    private var state: State[MediaPlayer] = new Paused

    def pressPlayOrPauseButton(): Unit = {
      state.press(this)
    }

    def setState(state: State[MediaPlayer]): Unit = {
      this.state = state
    }
  }
  
  object MediaPlayerExample {
    def main(args: Array[String]): Unit = {
      val player = MediaPlayer()
      player.pressPlayOrPauseButton()
      player.pressPlayOrPauseButton()
      player.pressPlayOrPauseButton()
      player.pressPlayOrPauseButton()
    }
  }
```