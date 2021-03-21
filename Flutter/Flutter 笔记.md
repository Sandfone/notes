# Flutter 笔记

## Dart

### Keywords

- factory

  #### Factory constructors

  Use the `factory` keyword when implementing a constructor that doesn’t always create a new instance of its class. For example, a factory constructor might return an instance from a cache, or it might return an instance of a subtype.

  The following example demonstrates a factory constructor returning objects from a cache:

  ```dart
  class Logger {
    final String name;
    bool mute = false;
  
    // _cache is library-private, thanks to
    // the _ in front of its name.
    static final Map<String, Logger> _cache =
        <String, Logger>{};
  
    factory Logger(String name) {
      return _cache.putIfAbsent(
          name, () => Logger._internal(name));
    }
  
    Logger._internal(this.name);
  
    void log(String msg) {
      if (!mute) print(msg);
    }
  }
  ```

  > **Note:** Factory constructors have no access to `this`.

  Invoke a factory constructor just like you would any other constructor:

  ```dart
  var logger = Logger('UI');
  logger.log('Button clicked');
  ```



### Operate

- `?:`  三目运算符
- `??`  
- `?.`
- `??=`  simply means “If left hand side is null, carry out assignment”. This will only assign a value if the variable is null.
- `..`  Cascade notation
- `~/`  除数取整
- `...`  Spread operate 将一个 List 中的所有数据解包到另一个 List 中。



## Flutter

### pubspec.yaml

- dependencies 和 dev_dependencies 的区别

  dependencies 的依赖包将作为APP的源码的一部分参与编译，生成最终的安装包。而 dev_dependencies 的依赖包只是作为开发阶段的一些工具包，主要是用于帮助我们提高开发、测试效率，比如flutter的自动化测试包等。