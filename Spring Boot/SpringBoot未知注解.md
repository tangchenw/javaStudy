# SpringBoot陌生注解

### @SuppressWarnings(value = "unchecked")

跳过泛型检查，好像没啥用就少了开发环境的警告。

###  @ControllerAdvice

这个类是为那些声明了（`@ExceptionHandler`、`@InitBinder` 或 `@ModelAttribute`注解修饰的）方法的类而提供的专业化的`@Component` , 以供多个 `Controller`类所共享。

`@ControllerAdvice` 配合 `@ExceptionHandler` 实现全局异常处理

### @ExceptionHandler(value = Exception.class)

用于在特定的处理器类、方法中处理异常的注解

接收Throwable类作为参数，我们知道Throwable是所有异常的父类，所以说，可以自行指定所有异常

比如在**方法**上加：`@ExceptionHandler(IllegalArgumentException.class)`，则表明此方法处理

`IllegalArgumentException` 类型的异常，如果参数为空，将默认为方法参数列表中列出的任何异常（方法抛出什么异常都接得住）。
