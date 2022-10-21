# MonitorController

监控 Controller，在线用户，手动踢出用户等功能

```java
package com.xkcoding.rbac.security.controller;

import cn.hutool.core.collection.CollUtil;
import com.xkcoding.rbac.security.common.ApiResponse;
import com.xkcoding.rbac.security.common.PageResult;
import com.xkcoding.rbac.security.common.Status;
import com.xkcoding.rbac.security.exception.SecurityException;
import com.xkcoding.rbac.security.payload.PageCondition;
import com.xkcoding.rbac.security.service.MonitorService;
import com.xkcoding.rbac.security.util.PageUtil;
import com.xkcoding.rbac.security.util.SecurityUtil;
import com.xkcoding.rbac.security.vo.OnlineUser;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * <p>
 * 监控 Controller，在线用户，手动踢出用户等功能
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-11 20:55
 */
@Slf4j
@RestController
@RequestMapping("/api/monitor")
public class MonitorController {
    @Autowired
    private MonitorService monitorService;

    /**
     * 在线用户列表
     *
     * @param pageCondition 分页参数
     */
    @GetMapping("/online/user")
    public ApiResponse onlineUser(PageCondition pageCondition) {
        //传入VO分页对象，包括当前页码和每页条数属性
        PageUtil.checkPageCondition(pageCondition, PageCondition.class);
        //根据分页参数获取在线用户列表，返回封装的在线用户分页信息
        PageResult<OnlineUser> pageResult = monitorService.onlineUser(pageCondition);
        return ApiResponse.ofSuccess(pageResult);
    }

    /**
     * 批量踢出在线用户
     *
     * @param names 用户名列表
     */
    @DeleteMapping("/online/user/kickout")
    public ApiResponse kickoutOnlineUser(@RequestBody List<String> names) {
        //根据用户名批量踢出在线用户
        //空或者为当前用户则抛出异常
        if (CollUtil.isEmpty(names)) {
            throw new SecurityException(Status.PARAM_NOT_NULL);
        }
        if (names.contains(SecurityUtil.getCurrentUsername())) {
            throw new SecurityException(Status.KICKOUT_SELF);
        }
        //执行踢出操作，表现为清除redis中的jwt信息
        monitorService.kickout(names);
        return ApiResponse.ofSuccess();
    }
}
```