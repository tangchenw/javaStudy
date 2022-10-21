# SecurityException

全局异常

> 所有的自定义异常都建议继承BaseException，而不要直接返回BaseException，BaseException用于处理运行时异常（捕获型）

```java
package com.xkcoding.rbac.security.exception;

import com.xkcoding.rbac.security.common.BaseException;
import com.xkcoding.rbac.security.common.Status;
import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * <p>
 * 全局异常
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-10 17:24
 */
@EqualsAndHashCode(callSuper = true)
@Data
public class SecurityException extends BaseException {
    public SecurityException(Status status) {
        super(status);
    }

    public SecurityException(Status status, Object data) {
        super(status, data);
    }

    public SecurityException(Integer code, String message) {
        super(code, message);
    }

    public SecurityException(Integer code, String message, Object data) {
        super(code, message, data);
    }
}
```