# Lesson 05: CompletableFeature

## <a name="toc"></a> Содержание:
- [Runnable](#Runnable)
- [Callable и Future](#CallableAndFuture)
- [CompletableFuture](#CompletableFuture)

## <a name="Runnable"></a> Runnable
С самого начала развития языка Java в него закладывалась многопоточность. Наверно, каждый программист Java знает про **Runnable**.
Это простой интерфейс предназначенный для того, чтобы показать, что класс, его реализующий, предназначается для выполнения в потоке. Предоставляет метод один единственный **run**, который и будет вызван. Сам Runnable не является ни в коей мере новым потоком. Поэтому, если просто вызывать у Runnable метод run - выполнятся он будет в том же потоке, в котором его вызвали.
Пример простого Runnable, выводящего название потока:
```java
Runnable runnable = new Runnable() {
	@Override
    public void run() {
    	System.out.println("Thread name is " + Thread.currentThread().getName());
    }
};
```
Соответственно, его необходимо передать в поток:
```java
Thread newThread = new Thread(runnable);
newThread.start();
```
Метод start является synchronized, добавляет в группу запускаемый поток. Вызывается внутренний native метод старт. В потоке есть свой метод **run**, который по умолчанию вызывает у target(т.е. как раз у Runnable) метод **run**. Пример:
```java
Thread newThread = new Thread(runnable);
newThread.start();
```
И если мы попробуем написать тест, проверяющий что имя потока в Runnable отличается от имени потока, в котором выполняется тест, то мы столкнёмся с несколькими проблемами:
- Внутренний класс не может использовать не final / effective final переменные.
- Придётся сделать некий final контейнер, что является не слишком красивым решением
- Не слишком читабельно и удобно

## <a name="CallableAndFuture"></a> Callable и Future
И в Java 1.5 добавили интерфейс Callable, с методом сall.
Он аналогичен Runnable, т.е. предназначен для выполнения в потоке, но не является void, а возвращает результат.
Так же добавили интерфейс Future, который представляет результат асинхронного выполнения. Т.е. представляет результат, который будет получен когда-то в будущем.

Но, как мы помним, интерфейс - это контракт. А нам нужны реализации. Поэтому, добрые разработчики Java нам предоставили класс FutureTask - который и является представлением задачи (Task) в виде Runnable или Callable, которая будет выполнена в будущем, т.е. во Future.
```java
// Что-то, что предполагается выполнять в потоке
Callable<String> callable = new Callable<String>() {
	@Override
    public String call() throws Exception {
    	return Thread.currentThread().getName();
    }
};

// Результат (так же Runnable, что позволяет отправить его в поток)
FutureTask<String> future = new FutureTask<String>(callable);
new Thread(future).start();
try {
	System.out.println(future.get());
} catch (InterruptedException | ExecutionException e) {
	e.printStackTrace();
}
```

Можно сделать ещё и с применением ExecutorService:
```java
ExecutorService pool = Executors.newFixedThreadPool(3);
Callable<String> callable = new Callable<String>() {
	@Override
    public String call() throws Exception {
    	return Thread.currentThread().getName();
    }
};
Future<String> future = pool.submit(callable);
try {
	System.out.println(future.get());
} catch (InterruptedException | ExecutionException e) {
	e.printStackTrace();
}
```
Подробнее можно так же прочитать здесь:
"[Guide to java.util.concurrent.Future](http://www.baeldung.com/java-future)"

"[Многопоточное программирование в Java 8. Часть первая. Параллельное выполнение кода с помощью потоков](https://tproger.ru/translations/java8-concurrency-tutorial-1/)"

"[How to use Future and FutureTask in Java Concurrency with Example](http://javarevisited.blogspot.ru/2015/01/how-to-use-future-and-futuretask-in-Java.html)"

## <a name="CompletableFuture"></a> CompletableFuture

Материал: [Guide To CompletableFuture](http://www.baeldung.com/java-completablefuture)

