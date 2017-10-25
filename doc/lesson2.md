# Lesson 02: StreamAPI

## <a name="toc"></a> Содержание:
- [Введение](#Intro)
- [Получение стрима](#GetStream)
- [Операции](#Operations)
- [Примеры](#Examples)
- [Map и FlatMap](#MapFlatMap)
- [Дополнительные материалы](#Materials)

## <a name="Intro"></a> Введение
Что такое Stream API? Согласно [JavaDoc для Interface Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html), Stream - есть ```A sequence of elements supporting sequential and parallel aggregate operations```. То есть это последовательность элементов, которая поддерживает последовательную или параллельную обработку.

Так же сказано, что Stream (стримы) и Collection (коллекции) похожи, но имеют разные цели. Коллекции сосредоточены на эффективном управлении и доступе к элементам, а стримы сосредоточены на декларативном описании источника и действий, производимых над ним.

Действия над стримом представляет собой pipeline из действий, которые могут выполняться как последовательно, так и параллельно. Способ выполнения является характеристикой/настройкой/property самого стрима.

## <a name="GetStream"></a> Получение стрима
В Java 8 в интерфейсе Collection добавился метод создания стрима из коллекции - Stream. Следовательно, любая реализация коллекции имеет метод получения стрима:
```java
List<String> test = Arrays.asList("foo", "bar");
test.stream();
```
Так же добавились утилитные методы в Arrays:
```java
Arrays.stream(new String[]{"foo","bar"})
```
Так же можно создать непосредственно через сам класс Stream:
```java
Stream.of("foo", "bar")
```
Или через билдер в том же Stream:
```java
Stream.builder().add("foo").add("bar").build();
```
А так же через генератор:
```java
// Get Stream of 10 int
System.out.println(Stream.generate(() -> new Random().nextInt()).limit(10L));
```

## <a name="Operations"></a> Операции
Операции над стримами делятся на конвеерные и терминальные.
Конвеерные операции выполняют какое либо действие и возвращают стрим.
Терминальные операции возвращают результат обработки, не являющийся стримом.
Выполнение операций над стримом является ленивой (lazy) или отложенной - это означает, что пока не будет указана терминальная операция - никаких действий выполнено не будет.

Так же важной особенностью операций над стримами является то, что стримы - одноразовые. Следовательно, операции могут быть выполнены только 1 раз.
Следующий код приведёт к ```IllegalStateException: stream has already been operated upon or closed```:
```java
Stream stream = Stream.builder().add("foo").add("bar").build();
System.out.println(stream.count());
System.out.println(stream.count());
```

## <a name="Examples"></a> Примеры
**Limit** - Ограничить верхнее число элементов
```java
Stream.generate(() -> new Random().nextInt(10)).limit(10L).forEach(e-> System.out.print(e));
```
**Skip** - Пропустить первые n элементов
```java
Stream.generate(() -> new Random()11111111111111.nextInt(10)).limit(10L).skip(5L).forEach(e-> System.out.print(e));
```
**Filter** - Отфилитровать элементы по предикату:
```java
IntStream natural = IntStream.iterate(0, i -> i + 1);
natural.limit(10L).filter(e -> e % 2 == 0).forEach(e -> System.out.print(e));
```

Пример других операций можно посмотреть в статье: "[Шпаргалка Java программиста 4. Java Stream API](https://habrahabr.ru/company/luxoft/blog/270383/)".

## <a name="MapFlatMap"></a> Map и FlatMap
Одним из неочевидных мест может являться Map и FlatMap.

## <a name="Materials"></a> Дополнительный материал
Видео доклады:
[Сергей Куксенко : Stream API - часть 1](https://www.youtube.com/watch?v=O8oN4KSZEXE)

[Сергей Куксенко : Stream API - часть 2](https://www.youtube.com/watch?v=i0Jr2l3jrDA)