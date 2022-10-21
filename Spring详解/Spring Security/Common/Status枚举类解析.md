# Status枚举类解析

> 枚举类型的常数数组获取 Status[] statuses = Status.values();  枚举类.value()方法

该枚举类主要用于通用状态码封装

属性封装：

- 状态码：code 类型：Integer
- 返回信息：message 类型：String

构造方法封装：

`Status(Integer code, String message)`

剩下的就是通用状态码常数枚举

```java
package com.xkcoding.rbac.security.common;

import lombok.Getter;

/**
 * <p>
 * 通用状态码
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-07 14:31
 */
@Getter
public enum Status implements IStatus {
    /**
     * 操作成功！
     */
    SUCCESS(200, "操作成功！"),

    /**
     * 操作异常！
     */
    ERROR(500, "操作异常！"),

    /**
     * 退出成功！
     */
    LOGOUT(200, "退出成功！"),

    /**
     * 请先登录！
     */
    UNAUTHORIZED(401, "请先登录！"),

    /**
     * 暂无权限访问！
     */
    ACCESS_DENIED(403, "权限不足！"),

    /**
     * 请求不存在！
     */
    REQUEST_NOT_FOUND(404, "请求不存在！"),

    /**
     * 请求方式不支持！
     */
    HTTP_BAD_METHOD(405, "请求方式不支持！"),

    /**
     * 请求异常！
     */
    BAD_REQUEST(400, "请求异常！"),

    /**
     * 参数不匹配！
     */
    PARAM_NOT_MATCH(400, "参数不匹配！"),

    /**
     * 参数不能为空！
     */
    PARAM_NOT_NULL(400, "参数不能为空！"),

    /**
     * 当前用户已被锁定，请联系管理员解锁！
     */
    USER_DISABLED(403, "当前用户已被锁定，请联系管理员解锁！"),

    /**
     * 用户名或密码错误！
     */
    USERNAME_PASSWORD_ERROR(5001, "用户名或密码错误！"),

    /**
     * token 已过期，请重新登录！
     */
    TOKEN_EXPIRED(5002, "token 已过期，请重新登录！"),

    /**
     * token 解析失败，请尝试重新登录！
     */
    TOKEN_PARSE_ERROR(5002, "token 解析失败，请尝试重新登录！"),

    /**
     * 当前用户已在别处登录，请尝试更改密码或重新登录！
     */
    TOKEN_OUT_OF_CTRL(5003, "当前用户已在别处登录，请尝试更改密码或重新登录！"),

    /**
     * 无法手动踢出自己，请尝试退出登录操作！
     */
    KICKOUT_SELF(5004, "无法手动踢出自己，请尝试退出登录操作！");

    /**
     * 状态码
     */
    private Integer code;

    /**
     * 返回信息
     */
    private String message;

    Status(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    public static Status fromCode(Integer code) {
        Status[] statuses = Status.values();
        //遍历获取状态码对应的枚举常数，如果获取到则返回对应枚举。
        for (Status status : statuses) {
            if (status.getCode().equals(code)) {
                return status;
            }
        }
        return SUCCESS;
    }

    @Override
    public String toString() {
        // hutool-all 第三方工具类中String类的方法，类似于Log.info("{}",context);
        return String.format(" Status:{code=%s, message=%s} ", getCode(), getMessage());
    }

}

```



## 枚举接口声明

```java
package com.xkcoding.rbac.security.common;

/**
 * <p>
 * REST API 错误码接口
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-07 14:35
 */
public interface IStatus {

    /**
     * 状态码
     *
     * @return 状态码
     */
    Integer getCode();

    /**
     * 返回信息
     *
     * @return 返回信息
     */
    String getMessage();

}

```

