# PageResult解析

该类主要用于通用分页参数返回

封装属性说明：

- 当前页数据 rows 类型：List<T> 返回对象List集合
- 总条数 total 类型：long

封装方法说明：
`public static <T> PageResult of(List<T> rows, Long total)`返回对应参数的PageResult对象

```java
package com.xkcoding.rbac.security.common;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.List;

/**
 * <p>
 * 通用分页参数返回
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-11 20:26
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class PageResult<T> implements Serializable {
    private static final long serialVersionUID = 3420391142991247367L;

    /**
     * 当前页数据
     */
    private List<T> rows;

    /**
     * 总条数
     */
    private Long total;

    public static <T> PageResult of(List<T> rows, Long total) {
        return new PageResult<>(rows, total);
    }
}

```

