# CustomUserDetailsService

自定义UserDetails查询类

```java
package com.xkcoding.rbac.security.service;

import com.xkcoding.rbac.security.model.Permission;
import com.xkcoding.rbac.security.model.Role;
import com.xkcoding.rbac.security.model.User;
import com.xkcoding.rbac.security.repository.PermissionDao;
import com.xkcoding.rbac.security.repository.RoleDao;
import com.xkcoding.rbac.security.repository.UserDao;
import com.xkcoding.rbac.security.vo.UserPrincipal;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

/**
 * <p>
 * 自定义UserDetails查询
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-10 10:29
 */
@Service
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private UserDao userDao;

    @Autowired
    private RoleDao roleDao;

    @Autowired
    private PermissionDao permissionDao;

    @Override
    public UserDetails loadUserByUsername(String usernameOrEmailOrPhone) throws UsernameNotFoundException {
        //JPA根据用户名或者email或者phone查询是否存在User对象，返回的对象是Optional一个容器对象，通过orElseThrow方法进行空指针异常信息处理
        User user = userDao.findByUsernameOrEmailOrPhone(usernameOrEmailOrPhone, usernameOrEmailOrPhone, usernameOrEmailOrPhone).orElseThrow(() -> new UsernameNotFoundException("未找到用户信息 : " + usernameOrEmailOrPhone));
        //通过用户id查询用户绑定的角色
        List<Role> roles = roleDao.selectByUserId(user.getId());
        //拿到用户所有的角色id集合
        List<Long> roleIds = roles.stream().map(Role::getId).collect(Collectors.toList());
        //通过角色id查询权限，拿到所有的权限集合
        List<Permission> permissions = permissionDao.selectByRoleIdList(roleIds);
        //返回该登录对象的vo对象
        return UserPrincipal.create(user, roles, permissions);
    }
}
```