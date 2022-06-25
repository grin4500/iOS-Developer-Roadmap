# Модель Акторов

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/47610132/166106052-df1c6942-a7f0-448c-a7d2-729f60c063ed.png">
</p>

Модель акторов была придумана в 1973 году, как некая теоретическая модель для описания различных параллельных систем, где свою популярность получила благодаря ЯП Эрланг.

Одним из источников проблем, в многопоточном программировании, является наличие разделяемого состояния и многопоточности, т.е когда у нас `разделяемое, изменяемое состояние + параллелизм = проблемы`, как пример: **race conditions**, **data races**, необходимость в блокировках и прочие сложности.

Одним из путей решения таких проблем, можно считать отказ от изменяемого состояния, т.е вместо того, чтобы иметь `разделяемые, изменяемые объекты` мы работаем с **неизменяемыми** данными и строим цепочки для обработки данных.
Акторы предлагают совершенно другой подход, вместо того чтобы отказываться от `изменяемого состояния`, акторы предлагают отказаться от `разделяемого состояния`, т.е
вместо того, чтобы потоки работали над общими данными, мы говорим, что наши данные являются приватными данными и с ними может работать строго один поток, т.е мы запрещаем любой параллелизм над этими данными!

# Акторы в swift
Тип данных, `ссылочный`, защищающий данные экземпляра от параллельного доступа. Это достигается путём "изоляции" актора, `во время компиляции`, которая гарантирует, что все обращения к данным этого экземпляра проходят через механизм синхронизации, который выполняется последовательно.
Основная идея акторов состоит в том, чтобы обеспечить «островки одиночной многопоточности», которые не нуждаются во внутренней синхронизации, потому - что общее изменяемое состояние избегается в пользу передачи семантических типов значений через асинхронные сообщения.

Акторы аналогичны другим типам в свифт: **структурам, перечеслениям** и **классам**. Акторы могут содержать в себе:
  - Методы, включая статические, свойства, индексы, инициализацию.
  - Не поддерживают наследование, поэтому исключают `convenience` инициализатор, переопределение (overriding), область видимости `open` и `final`.

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/47610132/166108724-4cf89152-8a8b-40cc-a0e4-e9f853cfae08.png">
</p>

Акторы объединяют в себе лучшее из обоих инструментов, используя преимущества совместного пула потоков для эффективного планирования.
При вызове метода для актора, который еще не выполняется, вызывающий поток может быть повторно использован для выполнения вызова метода.
В случае, когда вызываемый актор уже запущен, вызывающий поток может приостановить выполняемую им функцию и перехватить другую работу.

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/47610132/166109470-df62e380-cbe3-4526-8446-bda361906387.png">
</p>

  - Если очередь еще не запущена, мы говорим, что `нет конфликта (no contention)`
  - Если последовательная(serial) очередь уже запущена, будем считать, что очередь находится `в состоянии конфликта (under contention)`

`@GlobalActor` - существуют из-за того факта, что синхронизация состояния не ограничивается локальными переменными, а это означает, что может потребоваться глобальный доступ к актору. 
Вместо того, чтобы заставлять всех писать повсюду синглтоны, Global Actors позволяют легко указать, что определенный фрагмент кода должен выполняться в рамках определенного глобального актора.
  - Можно создать свои собственные глобальные акторы, добавив к актору атрибут `@globalActor`

`@MainActor` - глобальный актор, который обеспечивает выполнение всего кода в основном **(main)** потоке.
  - Добавив аттрибут `@MainActor` к классу, все свойства и методы как `@MainActor`, если - вдруг нужно избежать этого, для какой-то конкретной функции или свойству, то достаточно пометить эту сущность как `nonisolated`.
```
@MainActor
class ViewController: UIViewController {
	//...
	nonisolated var fetchBooks(with page: Int) -> [Books] { ... }
	//...
}
```

# Какие проблемы решают акторы?

**Гонка данных** - происходит, когда один поток обращается (читает) к изменяемому объекту, а другой записывает в него.
  - Если два потока, работающие параллельно, вызывают `worker(number: Int)` функцию, то в лучшем случае, будет неконсистентным объект, который мы изменяем, иначе, если **в один и тот же момент времени** оба потока обратятся к функции `worker(number: Int)`, то произойдет краш.
```
public final class Counter {
    
    private var myVariable = 0
    
    func worker(number: Int) {
        myVariable += number
        print(myVariable)
    }
    
}
```

<p align="left" width="100%">
    <img width="35%", src="https://user-images.githubusercontent.com/47610132/166111215-d7e49b99-a79b-47a7-b571-0c0e0d2ab84e.png">
</p>

**До акторов** мы могли решить проблему гонки данных таким путём:
![image](https://user-images.githubusercontent.com/47610132/168416207-dca6e4cc-8d45-47f0-8c11-54755c326bca.png)
Можно заметить, что в решении выше, весьма достаточно кода для обеспечения синхронизации. Для записи данных - барьерный флаг необходим, чтобы остановить чтение, на мгновение, и разрешить запись даьше, т.е мы должны помнить об этом.

Акторы, с другой стороны, позволяют swift максимально оптимизировать синхронизированный доступ. Используемый базовый лок является всего лишь детализацией реализации, где в результате компилятор swift может обеспечить синхронизированный доступ.
Тот же пример, но с использованием актора:
```
actor Counter {
    public var myVariable = 0
    
    /// We no longer need a barrier
    func addNumber(with number: Int) {
        myVariable += number
    }
    
    /// We no longer need a barrier
    func removeNumber(with number: Int) {
        myVariable -= number
    }
}
```
Теперь код стал намного меньше, плюс вся логика с синхронизацией доступа скрыта, как и детали реализации.

# Как работают акторы?

```
public protocol Actor: AnyObject, Sendable {
    nonisolated var unownedExecutor: UnownedSerialExecutor { get }
}

final class ImageDownloader: Actor {
    // ...
}
```
  - [`Sendable`](https://github.com/apple/swift-evolution/blob/main/proposals/0302-concurrent-value-and-concurrent-closures.md) — протокол чья цель "пометить", что тип `безопасен` для работы в параллельной среде.
  - [`nonisolated`](https://github.com/apple/swift-evolution/blob/main/proposals/0313-actor-isolation-control.md#non-isolated-declarations) отключает проверку безопасности и позволяет «переопределить» проверки акторов во время компиляции Swift.
  - [`UnownedSerialExecutor`](https://github.com/rjmccall/swift-evolution/blob/custom-executors/proposals/0000-custom-executors.md) - слабая ссылка на протокол `SerialExecutor`

У `SerialExecutor: Executor` от Executor есть метод func enqueue(_ job: UnownedJob), который выполняет задачи. Сначала пишем это:
```
let imageDownloader = ImageDownloader()
Task {
    await imageDownloader.setImage(for: "image", image: UIImage())
}
```
Т.е семантически происходит следующее:
```
let imageDownloader = ImageDownloader()
Task {
    imageDownloader.unownedExecutor.enqueue {
        setImage(for: "image", image: UIImage())
    }
}
```

По умолчанию Swift генерирует стандартный **SerialExecutor** для кастомных акторов, где кастомные реализации **SerialExecutor** переключают потоки. 
Так работает **MainActor** - актор, у которого **Executor** переводит в главный поток, где создать его нельзя, но можно обратиться к его экземпляру **MainActor.shared**.

# Дополнительные материалы:
  - [Документация от Apple](https://github.com/apple/swift-evolution/blob/main/proposals/0306-actors.md)
  - [WWDC21, Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/)
  - [Акторы Swift под капотом](http://swiftrocks.com/how-actors-work-internally-in-swift)