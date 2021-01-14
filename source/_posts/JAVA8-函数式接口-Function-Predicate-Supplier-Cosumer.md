---
title: JAVA8 函数式接口 Function/Predicate/Supplier/Consumer
date: 2021-01-14 15:27:36
tags:
- Java
category: Java小技巧
---


函数式接口是为方便Lambda表达而诞生的新型技术点。这里从43个接口中记录了一些常用的接口。


## 1. Consumer - 消费者

   | 接口 | 详情 |
   |  ----      | ---- |
   | Consumer< T >  | 提供一个T类型的输入参数，不返回执行结果 |
   | BiConsumer<T,U> |提供两个自定义类型的输入参数，不返回执行结果 |
   | DoubleConsumer | 表示接受单个double值参数，但不返回结果的操作 |
   | IntConsumer | 表示接受单个int值参数，但不返回结果的操作 | 
   | LongConsumer | 表示接受单个long值参数，但不返回结果的操作 | 
   | ObjDoubleConsumer< T > |表示接受object值和double值，但是不返回任何操作结果 |
   | ObjIntConsumer< T > | 表示接受object值和int值，但是不返回任何操作结 |
   | ObjLongConsumer< T > | 表示接受object值和long值，但是不返回任何操作结果 |

   Consumer<T>
   提供一个T类型的输入参数，不返回执行结果
    
```JAVA
    //对给定的参数执行操作
    void accept(T t)

    //返回一个组合函数，after将会在该函数执行之后应用
    default Consumer<T> andThen(Consumer<? super T> after)


    StringBuilder sb = new StringBuilder("Hello ");
    Consumer<StringBuilder> consumer = (str) -> str.append("Jack!");
    consumer.accept(sb);
    System.out.println(sb.toString());	// Hello Jack!

    Consumer<StringBuilder> consumer1 = (str) -> str.append(" Bob!");
    consumer.andThen(consumer1).accept(sb);

```

## 2. Supplier - 供应者


| 接口 | 详情 |
|  ----      | ---- |
|Supplier< T > | 不提供输入参数，但是返回结果的函数 | 
| BooleanSupplier | 不提供输入参数，但是返回boolean结果的函数 |
| DoubleSupplier | 不提供输入参数，但是返回double结果的函数 | 
| IntSupplier | 不提供输入参数，但是返回int结果的函数 | 
| LongSupplier | 不提供输入参数，但是返回long结果的函数 |

```JAVA
//获取结果值
T  get()	

Supplier<String> supplier = () -> "Hello Jack!";
System.out.println(supplier.get()); // Hello Jack!
```

>CompletableFuture 类提供了异步多线程池的方法，其中充分利用了Consumer和Supplier等函数式接口。
>执行之后可以接thenApply，thenRun，thenAccept来处理
> 
>1. Function<? super T,? extends U>
> 2. Runnable
> 3. Consumer<? super T>
```Java
    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the given executor with the value obtained
     * by calling the given Supplier.
     *
     * @param supplier a function returning the value to be used
     * to complete the returned CompletableFuture
     * @param executor the executor to use for asynchronous execution
     * @param <U> the function's return type
     * @return the new CompletableFuture
     */
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }
```

## 3. Predicate - 断言



| 接口 | 详情 |
|  ----      | ---- |
|Predicate< T > | 对给定的输入参数执行操作，返回一个boolean类型的结果（布尔值函数）|
|BiPredicate<T,U> | 对给定的两个输入参数执行操作，返回一个boolean类型的结果（布尔值函数）
|DoublePredicate | 对给定的double参数执行操作，返回一个boolean类型的结果（布尔值函数）
|IntPredicate | 对给定的int输入参数执行操作，返回一个boolean类型的结果（布尔值函数）
|LongPredicate | 对给定的long参数执行操作，返回一个boolean类型的结果（布尔值函数）

| Predicate方法 | 详情 |
|  ----      | ---- |
|boolean  test(T t) | 根据给定的参数进行判断
| Predicate< T > and(Predicate<? super T> other)| 返回一个组合判断，将other以短路并且的方式加入到函数的判断中
| Predicate< T > or(Predicate<? super T> other) | 返回一个组合判断，将other以短路或的方式加入到函数的判断中
| Predicate< T > negate() | 将函数的判断取反


```java

Predicate<Integer> predicate = number -> number != 0;
System.out.println(predicate.test(10));    //true
        
predicate = predicate.and(number -> number >= 10);
System.out.println(predicate.test(10));    //true
```

## 3. Function-功能


| 接口 | 详情 |
|  ----      | ---- |
|Function<T,R>|接收一个参数并返回结果的函数
|BiFunction<T,U,R>|接受两个参数并返回结果的函数
|DoubleFunction< R >|接收一个double类型的参数并返回结果的函数
|DoubleToIntFunction|接收一个double类型的参数并返回int结果的函数
|DoubleToLongFunction|接收一个double类型的参数并返回long结果的函数
|IntFunction< R >|接收一个int类型的参数并返回结果的函数
|IntToDoubleFunction|接收一个int类型的参数并返回double结果的函数
|IntToLongFunction|接收一个int类型的参数并返回long结果的函数
|LongFunction< R >|接收一个long类型的参数并返回结果的函数
|LongToDoubleFunction|接收一个long类型的参数并返回double结果的函数
|LongToIntFunction|接收一个long类型的参数并返回int结果的函数
|ToDoubleBiFunction<T,U>|接收两个参数并返回double结果的函数
|ToDoubleFunction< T >|接收一个参数并返回double结果的函数
|ToIntBiFunction<T,U>|接收两个参数并返回int结果的函数
|ToIntFunction< T >|接收一个参数并返回int结果的函数
|ToLongBiFunction<T,U>|接收两个参数并返回long结果的函数
|ToLongFunction< T >|接收一个参数并返回long结果的函数

| Function<T,R>方法 | 详情 |
|  ----      | ---- |
|R  apply(T t) | 将此参数应用到函数中
|Function<T, R>  andThen(Function<? super R,? extends V> after)	| 返回一个组合函数，该函数结果应用到after函数中
|Function<T, R>  compose(Function<? super V,? extends T> before) | 返回一个组合函数，首先将入参应用到before函数，再将before函数结果应用到该函数中

```java

Function<String, String> function = a -> a + " Jack!";
System.out.println(function.apply("Hello")); // Hello Jack!
        

Function<String, String> function = a -> a + " Jack!";
Function<String, String> function1 = a -> a + " Bob!";
String greet = function.andThen(function1).apply("Hello");
System.out.println(greet); // Hello Jack! Bob!
        

Function<String, String> function = a -> a + " Jack!";
Function<String, String> function1 = a -> a + " Bob!";
String greet = function.compose(function1).apply("Hello");
System.out.println(greet); // Hello Bob! Jack!
```

## 4. Operator-操作员


| 接口 | 详情 |
|  ----      | ---- |
|UnaryOperator< T >|提供单个参数，并且返回一个与输入参数类型一致的结果
|BinaryOperator< T >|提供两个参数，并且返回结果与输入参数类型一致的结果
|DoubleBinaryOperator|提供两个double参数并且返回double结果
|DoubleUnaryOperator|提供单个double参数并且返回double结果
|IntBinaryOperator|提供两个int参数并且返回int结果
|IntUnaryOperator|提供单个int参数并且返回int结果
|LongBinaryOperator|提供两个long参数并且返回long结果
|LongUnaryOperator|提供单个long参数并且返回long结果


| UnaryOperator< T >方法 | 详情 |
|  ----      | ---- |
|T apply(T t) | 将给定参数应用到函数中
|Function<T, R>  andThen(Function<? super R,? extends V> after)	| 返回一个组合函数，该函数结果应用到after函数中
|Function<T, R>  compose(Function<? super V,? extends T> before) |返回一个组合函数，首先将入参应用到before函数，再将before函数结果应用到该函数中


```java

UnaryOperator<String> unaryOperator = greet -> greet + " Bob!";
System.out.println(unaryOperator.apply("Hello")); // Hello Jack!
        

UnaryOperator<String> unaryOperator1 = greet -> greet + " Jack!";
String greet = unaryOperator.andThen(unaryOperator1).apply("Hello");
System.out.println(greet); // Hello Bob! Jack!
        

String greet = unaryOperator.compose(unaryOperator1).apply("Hello");
System.out.println(greet); // Hello Jack! Bob!
```

## 自己的TIPS

这些接口中，Predicate可以返回Boolean值，适合和Stream中的filter配合使用。Function和Operator在很多情况下可以替换。
下面是一个Predicate和Function组合使用，之后放在filter里面过滤的实例，目的是用getter获取属性之后根据属性去重：

```java
private <T> Predicate <T> distinctByKey(Function<? super, T ?> keyExtractor){
    Map<Obejct, Object> map = new ConcurrentHashMap<>(8);
    return t->map.putifAbsent(keyExtractor.apply(t), Boolean.True) == null;
}
```
private后面的<T>声明这是一个泛型方法（泛型类里的不是泛型方法），这个T就是方法里可以使用的泛型，
常见的标识如T,E,K,V等。

这个方法返回了一个带判断的Lambda表达式，其中t会带入filter中的值。


 