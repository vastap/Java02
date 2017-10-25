# Lesson 04: Spliterator

## <a name="toc"></a> Содержание:
- [Введение](#Intro)
- [Создание](#Create)
- [Определение](#Define)
- [Характеристики](#characteristics)
- [Размер](#Size)
- [Действия](#Actions)
- [Разделение](#Split)
- [Использование](#Usage)
- [Материалы](#Links)

## <a name="Intro"></a> Введение
Ещё в Java 1.2 был введён интерфейс java.util.Iterator.
Итератор имеет довольно простой контракт: hasNext(), next(), remove().

Начиная с версии 1.5 вводится интерфейс Iterable, который позволяет использовать многие вещи (коллекции, например) в цикле for-each, который по сути является "синтаксическим сахаром" и скрывает внутри всё тот же итератор.

В Java 8 в Iterable появился метод **forEach**.
Интересный обзор: [Guide to the Java 8 forEach](http://www.baeldung.com/foreach-java).

Начиная с Java 8 так же вводится новое понятие - Stream. Которое в том числе позволяет так же обходить элементы через **forEach**. И тут возникла задача иметь возможность итерироваться по набору элементов многопоточно. Итератор тут уже просто так не подойдёт. Поэтому, ввели новое понятие - **Spliterator** (Iterator + split).
Подробнее см. "[Java 8 Stream API, часть шестая: собственный поток (на самом деле нет)](https://easyjava.ru/java/java-8-stream-api-chast-shestaya-sobstvennyj-potok-na-samom-dele-net/)".

## <a name="Create"></a> Создание
Чтобы создать Stream с использованием сплитератора необходимо:
```java
java.util.stream.StreamSupport.stream(spliterator, true);
```
Второй параметр указывает, нужно ли выполнять стрим параллельно.

## Написание сплиттератора
Создать сплитератор можно двумя способами:
- implements Spliterator
- extends Spliterators.AbstractSpliterator или любого другого на выбор (например, Spliterators.AbstractIntSpliterator)
При наследовании так же понадобится создать конструктор, соответствующий конструктору в super.

При написании сплиттератора необходимо реализовать следующие методы:
- boolean tryAdvance(Consumer action)
- Spliterator trySplit()
- long estimateSize()
- int characteristics()

## <a name="Define"></a> Определение
Определим наш будущий сплитератор:
```java
public class TestSpliterator implements Spliterator<String> {
	private int firstPosition;
	private int lastPosition;
	private final String data[];

	public TestSpliterator(String[] data) {
		this.data = data;
		this.firstPosition = 0;
		this.lastPosition = data.length - 1;
	}

	public TestSpliterator(String[] data, int firstPosition, int lastPosition) {
		this.firstPosition = firstPosition;
		this.lastPosition = lastPosition;
		this.data = data;
	}

}
```

## <a name="characteristics"></a> Характеристики
Метод characteristics() возвращает характеристики в виде битовой маски.
Пример:
```java
@Override
public int characteristics() {
	return IMMUTABLE | SIZED | SUBSIZED;
}
```
Это всё константы из **java.util.Spliterator**.
Вот их смысл:
- ORDERED — порядок данных имеет значение.
Отсутствие этой характеристики автоматически переведёт параллельный поток в неупорядоченный режим, благодаря чему он сможет работать быстрее.
- DISTINCT — элементы заведомо уникальны (например, SET)
C ней операция distinct() на потоке выполняться не будет
- SORTED — элементы отсортированы (например, TreeSet)
C ней потоковая операция sorted() может быть пропущена
- SIZED — известно точное количество элементов сплитератора.
Такую характеристику возвращают сплитераторы всех коллекций.
После некоторых потоковых операций (например, map() или sorted()) она сохраняется, а после других (скажем, filter() или distinct()) — теряется.
Она полезна для сортировки или, скажем, операции toArray(): можно заранее выделить массив нужного размера, а не гадать, сколько элементов понадобится.
- SUBSIZED — известно, что все дочерние сплитераторы также будут знать свой размер.
Эту характеристику возвращает сплитератор от ArrayList, потому что при делении он просто разбивает диапазон значений на два диапазона известной длины. А вот HashSet её не вернёт, потому что он разбивает хэш-таблицу, для которой не известно, сколько содержится элементов в каждой половине. Соответственно дочерние сплитераторы уже не будут возвращать и SIZED.
- NONNULL — известно, что среди элементов нет null
Например, её возвращают все сплитераторы, созданные на примитивных типах.
- IMMUTABLE — известно, что источник данных в процессе обхода не может измениться
Сплитераторы от обычных коллекций такую характеристику не возвращают, но её выдаёт, например, сплитератор от Collections.singletonList(), потому что этот список изменить нельзя.
- CONCURRENT — сплитератор остаётся рабочим после любых изменений источника.
Такую характеристику сообщают сплитераторы коллекций из java.util.concurrent.
Если сплитератор не имеет характеристик IMMUTABLE и CONCURRENT, то хорошо бы заставить его работать в fail-fast режиме, чтобы он кидал ConcurrentModificationException, если заметит, что источник изменился.

## <a name="Size"></a> Размер
Сплитератор может знать свой размер.
Для этого можно добавить необходимые переменные:
```java
private int firstPosition;
private int lastPosition;
```

а так же добавить реализации методов:
```java
@Override
public long estimateSize() {
	return lastPosition- firstPosition;
}

@Override
public long getExactSizeIfKnown() {
	return estimateSize();
}
```
Если размер нельзя было бы вычислить, то необходимо было бы вернуть Long.MAX_VALUE.

## <a name="Actions"></a> Действия
Для выполнения непосредственно полезной работы необходимо реализовать метод
```java
@Override
public boolean tryAdvance(Consumer<? super String> action) {
	if (firstPosition <= lastPosition) {
		firstPosition++;
		action.accept(data[firstPosition]);
		return true;
	} else {
		return false;
	}
}
```

tryAdvance() похож на next() обычного Iterator. Он проверяет, существует ли следующий элемент и, если существует, применяет на него переданную функцию и возвращает true. В случае, если элементов больше не осталось, метод должен вернуть false.

Так же необходимо реализовать метод **forEachRemaining**
```java
@Override
public void forEachRemaining(Consumer<? super String> action) {
	for (;firstPosition <= lastPosition; firstPosition++) {
    	action.accept(data[firstPosition]);
	}
}
```

forEachRemaining() это такой групповой tryAdvance() который применяет переданную функцию ко всем оставшимся элементам Spliterator. Интересно, что результат estimateSize() должен быть равен числу элементов, с которым столкнётся forEachRemaining() в случае SIZED Spliterator.

## <a name="Split"></a> Разделение
Далее необходимо указать, каким образом будут делиться наши сплитераторы:
```Java
@Override
public Spliterator<String> trySplit() {
	int half = (lastPosition - firstPosition)/2;
	if (half<=1) {
		//Not enough data to split
		return null;
	}
	int f = firstPosition;
	int l = firstPosition + half;

	firstPosition = firstPosition + half +1;
	return new MultilineSpliterator(f, l);
}
```

Стоит помнить, что:
- старый Spliterator и новорождённый Spliterator никогда не должны иметь возможности вдвоём обработать какой-либо элемент. Это особенно важно учитывать для граничных элементов.
- estimateSize() старого Spliterator после trySplit() должен возвращать оставшееся после разделения число элементов
- операция разделения должна выполняться быстро, желательно чтобы её сложно была O(1) или близкой к тому.

## <a name="Usage"></a> Использование
Для проверки работы можно написать тест:
```java
@Test
public void testAppHasAGreeting() {
	String[] source = "abc\nbca\ncgh\njdsfs\nfsdfsd\nsdfdsf\n".split("\n");
    long count = StreamSupport.stream(new TestSpliterator(source), true).count();
    assertEquals(source.length, count);
}
```
Как видно, тест отрабатывает без ошибок. Значения сошлись - следовательно, работает корректно.

## <a name="Links"></a> Материалы
Использована статья: "[Java 8 Stream API, часть шестая: собственный поток (на самом деле нет)](https://easyjava.ru/java/java-8-stream-api-chast-shestaya-sobstvennyj-potok-na-samom-dele-net/)"

А так же статья: "[Пишем свой Spliterator](https://habrahabr.ru/post/256905/)".