# Домашнее задание к занятию «10. Coroutines: Scopes, Cancellation, Supervision»


## Вопросы: Cancellation

### Вопрос №1

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
```
### Ответ: 
Не отработает, т.к. **job.cancelAndJoin()** прервёт job быстрее.

### Вопрос №2

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
```
### Ответ:
Не отработает, т.к. **child.cancel()** прервёт child быстрее.

## Вопросы: Exception Handling

### Вопрос №1

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

### Ответ:
Не отработает, т.к. **throw Exception("something bad happened")** прервёт CoroutineScope из-за исключения.

### Вопрос №2

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
### Ответ:
Отработает, т.к. **coroutineScope** находится внутри блока try/catch

### Вопрос №3

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
### Ответ:
Отработает, т.к. **supervisorScope** находится внутри блока try/catch

### Вопрос №4

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```
### Ответ:
Не отработает, т.к. из-за задержки (500) первым отработает второй **launch {throw Exception("something bad happened")}**, который прервёт все остальные дочерние **launch**

### Вопрос №5

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
### Ответ:
Отработает, т.к. **supervisorScope**, и выполнение дочерних элементов не влияет друг на друга

### Вопрос №6

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
### Ответ:
Не отработает, т.к. первым выкинется **Exception**, весь скоуп прервется прежде, чем отработают запущенные корутины.

### Вопрос №7

Отработает ли в этом коде строка `<--` ? Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```

### Ответ:
Не отработает, т.к. **throw Exception("something bad happened")** прервет CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch и дочерние корутины  