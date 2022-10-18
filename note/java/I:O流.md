# I/O流

## File

File中提供了很多方法，但是没有直接提供一些基本功能，比如读取文件内容的操作。

```java
File file = new File("/file/path");
```

## 流

基于FileInputStream简单的读取文件

```java
try (InputStream inputStream = new FileInputStream("filepath")) {
    byte[] bytes = new byte[4096];
    while (inputStream.read(bytes) > 0) {
        // do something
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

## Reader和Writer类

把抽象从字节提升到字符，写出的代码更易于理解和阅读，同时也可以减少一部分代码量。

```java
try (BufferedReader bufferedReader = new BufferedReader(new FileReader("/Users/curry/Documents/local-code/java-demo/build.gradle"))) {
    String line = "";
    while ((line = bufferedReader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

```java
List<String> strings = Files.readAllLines(Paths.get("filepath"));
try (FileWriter fileWriter = new FileWriter("writepath")) {
    strings.forEach(e -> {
        try {
            fileWriter.write(e + "\n");
        } catch (IOException ex) {
            throw new RuntimeException(ex);
        }
    });
} catch (IOException e) {
    e.printStackTrace();
}
```

## NIO中的通道和缓冲区

### ButeBuffer
“直接缓冲区”，只要可能就会绕过Java堆内存，在本地内存中分配，而不是在标准的Java堆内存中。

直接通过`ByteBuffer.allocateDirect(65536)`创建对象。

缓存区这种抽象只存在于内存中，如果想影响外部世界，需要使用Channel对象。
```java
FileInputStream fis = getSomeStream();

try (FileChannel channel = fis.getChannel()) {
    ByteBuffer buffer = ByteBuffer.allocateDirect(16 * 1024 * 1024);
    while(channel.read(buffer) != -1 || buffer.position() > 0) {
        buffer.compact();
    }
} catch (IOException e) {
    // error
}
```

## 异步I/O
**基于Future接口的方式**
```java
try (AsynchronousFileChannel channel =
         AsynchronousFileChannel.open(Paths.get("input.txt"))) {
  ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024 * 100);
  Future<Integer> result = channel.read(buffer, 0);

  while(!result.isDone()) {
    // 做些其他有用的操作……
  }

  System.out.println("Bytes read: " + result.get());
}
```

**基于回调的方式**
基于`CompletionHandler`接口实现。接口定义了`completed()`和`failed()`，分别在操作成功和失败时调用。
```java
CompletionHandler<Integer,Object> h =
  new CompletionHandler() {
  public void completed(Integer written, Object o) {
    System.out.println("Bytes written: " + written);
  }

  public void failed(Throwable x, Object o) {
    System.out.println("Asynch write failed: "+ x.getMessage());
  }
};

try (AsynchronousFileChannel channel =
       AsynchronousFileChannel.open(Paths.get("primes.txt"),
          StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {

  channel.write(buffy, 0, null, h);
  Thread.sleep(1000); // 必须这么做，防止退出太快
}
```