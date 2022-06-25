# **📝 Виды памяти**

1. **Статическая** - Выделение памяти до начала выполнения самой программы. Вид такой памяти доступен в течении всего времени выполнения программы. В большинстве случаев, в языках программирования, при размещении объекта в статической памяти необходимо лишь задекларировать его в глобальной области видимости. Размещается на отдельном сегмента памяти.
2. **Автоматическая** - Вид памяти, который также известен как «размещение на стеке», - самый основной вид. Он автоматически выделяет различные аргументы и локальные переменные функции, а также другую различную метаинформацию при вызове необходимых функций, а далее он освобождает память при выходе из неё.
3. **Динамическая** - Выделение памяти из ОС по требованию приложения, как пример - куча.

- **Стек (Stack)**.
   - Тип данных - значение **Value Type**, работает по принципу LIFO ( Last In First Out ) - последним пришел, первым вышел.
      - Structs|Enums(но не indirect enum)|Tuples|Protocols|Primitives ( integer, bool, float, double and etc. ) - хранятся на стеке.
   - Растёт в сторону уменьшения адресов, т.е его начало находится где-то в старших адресах, а его конец где-то в младших адресах.
   - Управляется и оптимизируется от CPU и аллоцируется во время компиляции.
   - Каждый поток имеет свой собственный стек.
   - Всегда копируется, при присвоении.
   - Для простоты картины предположим, что у каждого стэка есть переменная **(stack pointer)** - адрес самого последнего добавленного элемента. Он используется для отслеживания вершины стэка и хранит в себе целочисленное число (Integer). 
   - Еще один указатель это **base pointer** - адрес начала фрейма, начиная с которого в стек вносятся или извлекаются значения, он используется для получения параметров в стеке, т.к его адрес **всегда статичен** в отличии от **stack pointer'a**.
   - Для работы со стеком есть 2 основные команды:
      - Push - поместить данные на вершину стека, тем самым увеличиваем stack ponter.
      - Pop - извлечь данные с вершины стека, тем самым уменьшаем stack pointer.
   - Операции со стеком:
      - Включение 
      - Исключение
      - Определение размера
      - Очистка
      - Неразрушающее чтение
   - У каждой функции в стеке есть своё место, которое называется **фрейм памяти**, внутри находится локальное окружение функции в виде ее переменных.

![image](https://user-images.githubusercontent.com/47610132/162479934-5d533b68-bae2-4626-aef9-22724406b13c.png)

Более подробно и визуально адаптированно можно посмотреть в этом небольшом [видео](https://www.youtube.com/watch?v=MXoMuymbfo8&t=393s)

- **Куча (Heap)**
   - Тип данных - ссылочный **Reference Type**, древовидная структура данных, где выделение памяти из ОС происходит по требованию приложения, используется для динамического выделения памяти и аллоцируется во время работы программы (рантайм). После выделения памяти в распоряжении программы поступает указатель на начало выделенной памяти. 
   - Все данные в куче располагаются линейно, т.е от начала её памяти к её концу, в сторону увеличения адресов.
      - Classes|Closures|Functions|Indirect Enums|Actors - хранятся на куче.
   - Передаются по ссылке, при присвоении.
   - Куча общая для всех потоков приложения.

# Классы и структуры могут:
   - Определить свойство для хранения значений.
   - Определить методы|функции.
   - Реализовать протоколы.
   - Определить инициализатор.
   - Определить подстрочные индексы для предоставления доступа к их переменным.

# Только классы могут:
   - Реализовать наследование.
   - Преобразование типа.
   - Определить деинициализатор.
   - Подсчёт ссылок.

# Чтобы обобщить разницу между структурами и классами, необходимо понять разницу между значениями и ссылочными типами

   - При создании копии **типа значения**, все данные копируются из копируемого объекта в новую переменную. Они представляют собой **2** отдельных объекта, и изменение одного не влияет на другой.
   - При создании копии **ссылочного типа** новая переменная ссылается на то же место в памяти, что и копируемый объект. Это означает, что изменение одного из них изменит другое, поскольку оба относятся к одному и тому же расположению в памяти.

![image](https://user-images.githubusercontent.com/47610132/162490415-d79770b2-c2df-4be0-8d83-178ead7b3bdb.png)

# Что советует **Apple**?

> Рассмотрите следующие рекомендации, чтобы помочь выбрать, какой вариант имеет смысл при добавлении нового типа данных в приложение.
- По умолчанию используйте структуры.
- Используйте классы при необходимости взаимодействия с Objective-C.
- Используйте классы, если необходимо управлять идентификацией моделируемых данных.
- Используйте структуры вместе с протоколами, чтобы использовать поведение путем совместного использования реализаций.
- Используйте структуры, чтобы избежать:
     - Подсчет ссылок.
     - Администрация свободной памяти и ее поиск для аллокации.
     - Перезапись памяти для деаллокации.

# **Важное дополнение!**

- Назначение в Stack ссылочных типов
Компилятор Swift может [продвигать](https://github.com/apple/swift/blob/62ccf81f7748e3e2c8626354d1ecb3adbd26b063/lib/SILOptimizer/Transforms/StackPromotion.cpp) ссылочные типы для размещения в стеке, когда их размер фиксирован или время жизни может быть предсказано.
   - Компилятор Swift может упаковывать типы значений и размещать их в куче:
      - **При соблюдении протокола.** Помимо затрат на выделение ресурсов, возникают дополнительные накладные расходы, когда тип значения хранится в экзистенциальном контейнере* и превышает длину 3 машинных слов.
      - **При смешивании value и reference типов.** Обычно ссылка на класс хранится в структуре, а структура является полем класса:
      ![image](https://user-images.githubusercontent.com/47610132/162569014-272745bf-3a92-42eb-945e-1e60d4841c01.png)
      - **@escaping closure (сбегающее замыкание)** если значение хранится на стеке и захвачено сбегающим замыканием, то это значение копируется в кучу, тем самым позваляя быть доступным до моменты выполнения замыкания. Это утверждение верно **только** для @escaping (сбегающего) замыкания, т.к только оно может выполниться позже.
      - **Inout аргумент** Аргументы @inout передаются в точку входа по адресу. Вызываемый объект не получает права собственности на указанную память. Указанная память должна быть инициализирована при входе в функцию и выходе из нее. Если аргумент @inout относится к хрупкой физической переменной (Unowned Unsafe), то аргументом является адрес этой переменной. Если аргумент @inout относится к логическому свойству, тогда аргумент является адресом буфера обратной записи, принадлежащего вызывающей стороне.
      - **String** — это copy-on-write структура, для реализации ее динамичности все свои Character-ы она хранит на Heap. Таким образом String — это структура, и хранится в Stack, но все свое содержимое она хранит на Heap. 

* Экзистенциальный контейнер - это общий контейнер для значения неизвестного типа среды выполнения. Value Types небольших размеров могут быть встроены в экзистенциальный контейнер. Более крупные размещаются в куче, и ссылка на них хранится в экзистенциальном буфере контейнера. Время жизни таких значений управляется [*Value Witness Table*](https://github.com/apple/swift/blob/main/docs/Lexicon.md#vwt-value-witness-table). Это вводит накладные расходы на подсчет ссылок и несколько уровней косвенного обращения при вызове методов протокола

# **Больше про память**

- [Habr: Память в Swift от 0 до 1](https://habr.com/ru/company/hh/blog/546856/)
- [WWDC: iOS Memory Deep Dive](https://developer.apple.com/videos/play/wwdc2018/416/)
