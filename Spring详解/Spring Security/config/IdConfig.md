# IdConfig

该类是雪花算法生成主键的配置类---雪花算法生成ID主要适用场景为分布式ID生成

```java
package com.xkcoding.rbac.security.config;

import cn.hutool.core.lang.Snowflake;
import cn.hutool.core.util.IdUtil;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * <p>
 * 雪花主键生成器
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-10 11:28
 */
@Configuration
public class IdConfig {
    /**
     * 雪花生成器
     */
    @Bean
    public Snowflake snowflake() {
        return IdUtil.createSnowflake(1, 1);
    }
}
```