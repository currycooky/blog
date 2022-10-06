# Java实战

## Stream

### FlatMap

给定数组[1, 2, 3]和[3, 4]，返回所有的数对

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> pairs =
        numbers1.stream()
                .flatMap(i -> numbers2.stream()
                        .map(j -> new int[]{i, j})
                )
                .collect(toList());
pairs.forEach(e -> System.out.println(Arrays.toString(e)));
```

### Reduce

可以用于求和等操作

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
System.out.println(numbers1.stream().reduce(1, (a, b) -> a * b).longValue());
```

> 结果为6
> `1`表示初始值

统计一个列表中有多少元素

```java
List<String> menus = Arrays.asList("a", "b", "c", "d");
int i = menus.stream().map(e -> 1).reduce(0, Integer::sum).intValue();
Assertions.assertEquals(i, 4);

// 也可以用count()替代
long j = menus.stream().count();
Assertions.assertEquals(j, 4);
```

### IntStream

```java
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
System.out.println(Arrays.toString(IntStream.range(1, 10).toArray()));
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
System.out.println(Arrays.toString(IntStream.rangeClosed(1, 10).toArray()));
```

## Optional
使用Optional来替换掉多次get

```java
String name = person.getCar().getInsurman().getName();

if (person.getCar() != null) {
        if (person.getCar().getInsurman() != null) {
                return person.getCar().getInsurman().getName();
        }
}
```
正常像这种连续的get方法，都需要我们去判断是否为空，避免出现空指针，而那样也会导致代码臃肿，不易读，通过使用Optional，可以有效的解决这种问题：
```java
// Optional<Person> person;
String name = person.flatMap(Person::getCar).flatMap(Car::getInsurman).map(Insurman::getName).orElse("Unknown");
```

虽然代码数量看着差不多，但是更加优雅，也更加易读

## 接口

如果一个类使用相同的函数签名从多个地方继承了方法，通过以下三条规则进行判断：

1. 类中的方法优先级最高，高于接口的默认方法；
2. 最具体实现的默认方法的接口更高，比如B继承A，B就比A更具体，其默认方法的优先级也就更高
3. 显示覆盖和调用期望的方法，显示地选择哪一个默认方法的实现