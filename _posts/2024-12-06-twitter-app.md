---
title: [Twitter Finatra] Twitter App
description: >-
date: 2024-12-06 20:00:00 KST 
categories: [Backend, Twitter Finatra]
tags: [Twitter, Finatra, Scala]
---

# 개요
Twitter Server 는 Twitter App 을 확장하고 있습니다. Twitter Server 를 이해하기 위해서는 Twitter App 에 대해 알고있어야 합니다.

Scala Application 은 Java 와 동일하게, main 메서드가 시작점입니다.
```scala
object MyApp {
  def main(args: Array[String]): Unit = {
    print(s"Hello World (args: $args)")
  }
}
```

# scala.App
scala.App 을 상속함으로써, main 메서드에 대한 정의 없이 Application 을 시작할 수 있습니다.
```scala
object MyApp extends App {
  print(s"Hello World (args: $args)")
}
```

# com.twitter.app.App
## Life Cycle
c.t.app.App 은 총 6 단계의 수명 주기를 가집니다.
```scala
import com.twitter.app.App

class MyApp extends App {

    init {
        // initialization logic
    }

    premain {
        // logic to run right before main()
    }

    def main(): Unit = {
        // your application logic here
    }

    postmain {
        // logic to run right after main()
    }

    onExit {
        // closing logic
    }

    closeOnExitLast(MyClosable) // final closing logic
}
```

```text
App#main() -->
App#nonExitingMain() -->
bind LoadService bindings
run all init {} blocks
App#parseArgs() // read command line args into Flags
run all premain {} blocks
run all defined main() methods
run all postmain {} blocks
App#close() -->
run all onExit {} blocks
run all Closables registered via closeOnExitLast()
```
