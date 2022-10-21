# CustomConfig

登录用户配置类，目前此类只添加了登录用户不需要拦截的地址；可能后续还有其它配置可以添加在此配置类中。

> @ConfigurationProperties(prefix = "custom.config")  元数据生成注解

```java
package com.xkcoding.rbac.security.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * <p>
 * 自定义配置
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-13 10:56
 */
@ConfigurationProperties(prefix = "custom.config")
@Data
public class CustomConfig {
    /**
     * 不需要拦截的地址,拦截地址配置类
     */
    private IgnoreConfig ignores;
}

```

