# BaseException解析

该类主要用于运行时异常的异常信息返回封装。

和ApiResponse的封装参数统一都是

- 状态码：code  类型：Integer
- 返回内容： message 类型：String
- 返回数据：data  类型：Object

构造方法的重载：

- `public BaseException(Status status)`入参为枚举类型的Status枚举类，详细参照[Status枚举类解析](C:\Users\汤琛\Desktop\学习资料\Spring详解\Spring Security\Common\Status枚举类解析.md)
- `public BaseException(Status status, Object data)`
- `public BaseException(Integer code, String message)`
- `public BaseException(Integer code, String message, Object data)`

```java
package com.xkcoding.rbac.security.common;

import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * <p>
 * 异常基类
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-07 14:57
 */
@EqualsAndHashCode(callSuper = true)
@Data
public class BaseException extends RuntimeException {
    private Integer code;
    private String message;
    private Object data;

    public BaseException(Status status) {
        super(status.getMessage());
        this.code = status.getCode();
        this.message = status.getMessage();
    }

    public BaseException(Status status, Object data) {
        this(status);
        this.data = data;
    }

    public BaseException(Integer code, String message) {
        super(message);
        this.code = code;
        this.message = message;
    }

    public BaseException(Integer code, String message, Object data) {
        this(code, message);
        this.data = data;
    }
}

```

