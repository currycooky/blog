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