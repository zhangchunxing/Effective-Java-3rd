# 目录

* [意图](README.md)
* [引言](chapter-01/Introduction.md)

* [创建和销毁对象](chapter-02/README.md)
  * [考虑静态工厂方法代替构造方法](chapter-02/Item1.md)

  * [当遇到许多构造方法参数时，考虑构建器](chapter-02/Item2.md)

  * [强制对单例属性使用私有构造方法或是枚举类型](chapter-02/Item3.md)

  * [使用私有构造方法来阻止类的实例化](chapter-02/Item4.md)

  * [优先使用依赖注入而非硬编码资源的关联关系](chapter-02/Item5.md)

  * [避免创建不必要的对象](chapter-02/Item6.md)

  * [消除废弃的对象引用](chapter-02/Item7.md)

  * [避免使用终结器与清理器](chapter-02/Item8.md)

  * [优先使用try-with-resources而非try-finally](chapter-02/Item9.md)


- [类和接口](chapter-04/README.md)
  - [在公有类中，请使用访问器方法而非公有字段](chapter-04/Item16.md)
  - [减少可变性](chapter-04/Item17.md)
  - [优先使用组合而非继承](chapter-04/Item18.md)
  - [优先选择接口而不是抽象类](chapter-04/Item20.md)
  - [针对后代来设计接口](chapter-04/Item21.md)
  - [使用接口只用来定义类型](chapter-04/Item22.md)
  - [在一个源文件中只定义一个顶层类](chapter-4/Item25.md)

- [泛型](chapter-05/README.md)
  - [消除未检查的警告](chapter-05/Item27.md)
  - [优先选择列表而非数组](chapter-05/Item28.md)
  - [优先考虑泛型类型](chapter-05/Item29.md)
  - [优先考虑泛型方法](chapter-05/Item30.md)

- [枚举和注解](chapter-06/README.md)
  - [使用实例字段而非序数](chapter-06/Item35.md)
  - [使用EnumSet代替位字段](chapter-06/Item36.md)
  - [始终如一地使用Override注解](chapter-06/Item40.md)
  - [使用标记接口来定义类型](chapter-06/Item41.md)
  
- [Lambda和Stream](chapter-07/README.md)
  - [优先选择lambdas而非匿名类](chapter-07/Item42.md)
  - [优先选择方法引用而非lambdas](chapter-07/Item43.md)
  - [优先选择标准的函数式接⼝](chapter-07/Item44.md)

- [方法](chapter-08/README.md)
  - [谨慎设计方法签名](chapter-08/Item51.md)
  - [谨慎使用可变参数](chapter-08/Item53.md)
  - [返回空集合或是数组而非null](chapter-08/Item54.md)

- [通用程序设计](chapter-09/README.md)
  - [最小化局部变量的作用域](chapter-09/Item57.md)
  - [优先选择for-each循环而非传统的for循环](chapter-09/Item58.md)
  - [在需要精确答案的情况下请避免使用float与double](chapter-09/Item60.md)
  - [优先选择原生类型而非包装类型](chapter-09/Item61.md)
  - [注意字符串拼接的性能](chapter-09/Item63.md)
  - [通过接口来引用对象](chapter-09/Item64.md)
  - [优先选择接口而非反射](chapter-09/Item65.md)
  - [审慎地使用本地方法](chapter-09/Item66.md)

- [异常](chapter-10/README.md)
  - [只在异常条件下使用异常](chapter-10/Item69.md)
  - [可恢复条件下使用受检异常，程序错误下使用运行时异常](chapter-10/Item70.md)
  - [避免受检异常的不必要使用](chapter-10/Item71.md)
  
- [并发](chapter-11/README.md)
  - [对共享可变数据的同步访问](chapter-11/Item78.md)
  - [优先使用执行器，任务和流，而不是线程](chapter-11/Item80.md)
