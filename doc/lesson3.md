# Lesson 03: Optional

## <a name="toc"></a> Содержание:
- [Введение](#Intro)
- [Создание](#Create)
- [Получение значения](#GetValue)
- [Обработка](#Handling)
- [Итог](#Summary)

## <a name="Intro"></a> Введение
С давних пор одной из самых частых проблем является Null Pointer Exception, т.е. когда мы ожидаем объект, а приходит пустота. И вот в Java 8 появилось средство, которое призвано облегчить работу с возможностью получения NPE и корректной обработкой null вместо объекта. Более того, в механизм Optional добавлена тесная связь с функциональными новшествами из Java 8.
Подробнее: [Guide To Java 8 Optional](http://www.baeldung.com/java-optional)

## <a name="Create"></a> Создание Optional
Optional не является функциональным интерфейсом, это обычный класс.
Поэтому, живёт он в java.util и для работы требуется выполнить импорт:
```java
import java.util.Optional;
```
Для создания Optional необходимо воспользоваться статическими методами.
Например:
```java
Optional<String> firstName = Optional.of(object);
```
Стоит сразу учесть, что если object будет null - мы получим NullPointerException.
Если мы не хотим такое поведение, то стоит использовать offNullable:
```java
Optional<String> firstName = Optional.ofNullable(object);
```
Так же можно вернуть пустой Optional:
```java
Optional<String> firstName = Optional.empty();
```

## <a name="GetValue"></a> Получение значения
Получить значение из Optional можно несколькими способами, в зависимости от того, какого поведения мы хотим достичь.
Самым простым вариантом является метод **get**:
```java
Optional<String> firstName = Optional.of("TestString");
System.out.println(firstName.get());
```
Всё хорошо, но только пока у нас есть значение.
Если у нас **Optional.empty()**, мы получим ```java.util.NoSuchElementException```.
Если у нас **Optional.of(null)**, будет ```java.lang.NullPointerException```.

Самый близкий к традициям способ - обработка через if.
Для этого есть метод **isPresent** - т.е. дословно "представлен ли":
```java
Optional<String> text = Optional.of("Text");
if (text.isPresent()) {
	System.out.println(text.get());
}
```

Так же мы можем указать, что возвращать в случае, если значение не задано.
Самый простой способ - заменить объект объектом.
Для этого есть метод **orElse**:
```java
Optional<String> text = Optional.ofNullable(null);
System.out.println(text.orElse("Null detected!"));
```
Кроме этого, мы можем указать функцию, результат которой мы хотим вернуть в том случае, если значения нет. То есть входного параметра нет у функции, есть только результат. Такими характеристиками обладает **Supplier**, поэтому:
```java
Optional<String> text = Optional.empty();
Supplier<String> emptyResultSupplier  = () -> "Empty Result";
System.out.println(text.orElseGet(emptyResultSupplier));
```
**orElseGet** потому так и называется - или вызови get, т.к. у Supplier только такой метод и есть.

Иногда может потребоваться выполнить какое-то действие, если значение есть (т.е. не null и не empty). В таком случае нам предлагают использовать **ifPresent**, т.е. "если представлен":
```java
Optional<String> text = Optional.of("Text for consumer");
Consumer<String> consumer = str -> System.out.println(str);
text.ifPresent(consumer);
```
Действие не возвращает результат, данный метод является void. Следовательно, мы будем использовать **Consumer**.

Интересно, что у нас есть возможность при незаданном значении предоставить исключение для того, чтобы оно было брошено. Для этого так же будет использован Supplier.
Для такого предусмотрен метод **orElseThrow**:
```java
Optional<String> text = Optional.ofNullable(null);
text.orElseThrow(() -> new IllegalStateException("Exception от Supplier")
```

## <a name="Handling"></a> Обработка
Кроме выше указанного, есть возможность указать для Optional дополнительную обработку.
Самым простым вариантом является добавление фильтра.
Вот пример с методом **filter**:
```java
// Предикат: символ является буквой (true = буква, а не цифра)
IntPredicate isAlphabetic = i -> Character.isAlphabetic(i);
// Предикат: ни один из символов в строке не является isAlphabetic
Predicate<String> isDigital = s -> s.chars().noneMatch(isAlphabetic);
System.out.println(string.filter(isDigital).orElse("is a text"));
```
Метод **map** используется для маппинга одного объекта на другой.
По сути это просто преобразование одного значения в Optional в другой:
```java
Optional<Integer> intValue = Optional.ofNullable(123);
// map используется для преобразования значения Optional, если оно есть.
String text = intValue.map(e -> String.valueOf(e)).orElse("");
System.out.println(text);
```

Есть ещё возможность использовать **flatMap**.
Случай когда может пригодиться: Когда в Optional у нас лежит Optional:
```java
Optional<String> word = Optional.of("apple");
Optional<Optional<String>> ofOptional = word.map(s -> Optional.of(s.toUpperCase()));
Optional<String> upperCased = word.flatMap(s -> Optional.of(s.toUpperCase()));
```
Как видно, когда мы выполняем маппинг в Optional, map возвращает Optional optional'ов.
Если же мы используем flatMap, то он умнее и возвращает Optional для типа.

## <a name="Summary"></a> Итог
Optional позволяет более гибко обработывать отсутствие значения.
Optional имеет статические методы для создания экземпляра Optional.
Optional имеет следующие методы создания:
- Optional.empty() - создание пустого Optional
- Optional.of(object) - создание Optional, не содержащего null
- Optional.ofNullable(object) - создание Optional, который может содержать null

Мы можем узнать, есть ли значение у Optional при помощи метода ```isPresent()```.

Есть несколько способов получить значение из Optional:
- optional.get() - получить значение без какой либо обработки отсутствия значения
- optional.orElse(object) - получить значение, а если его нет - вернуть объект
- optional.orElseGet(supplier) - получить значение, если нет - взять его из supplier
- optional.orElseThrow(supplier) - получить значение или throw исключение от supplier
- ifPresent(consumer) - если значение есть - передать его в consumer или бездействие

Так же Optional позволяет выполнять дополнительную обработку, если значение есть:
- optional.filter(predicate) - возвращает новый Optional.
Если предикат вернул true и optional не пуст - результирующий optional будет содержать значение
- optional.map(function) - выполнить преобразование значения из optional, если есть
- optional.flatMap(function) - выполнить "умное" преобразование optional optional'ов

Как видно, все преобразования возвращают тоже Optional. empty или со значением.