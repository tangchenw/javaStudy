# JwtAuthenticationFilter

此类作为Jwt 认证过滤器类，可能单独放在filter目录下更好，但是暂时放在config目录下

```java
package com.xkcoding.rbac.security.config;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import com.google.common.collect.Sets;
import com.xkcoding.rbac.security.common.Status;
import com.xkcoding.rbac.security.exception.SecurityException;
import com.xkcoding.rbac.security.service.CustomUserDetailsService;
import com.xkcoding.rbac.security.util.JwtUtil;
import com.xkcoding.rbac.security.util.ResponseUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Set;

/**
 * <p>
 * Jwt 认证过滤器
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2018-12-10 15:15
 */
@Component
@Slf4j
//OncePerRequestFilter是springboot的过滤器抽象类，作用是被继承实现并在请求时只执行一次过滤。
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    //实现了UserDetailsService接口重写其中loadUserByUsername从而自定义UserDetails查询的类
    @Autowired
    private CustomUserDetailsService customUserDetailsService;
    //JWT工具类注入，里面有创建JWT串，解析以及根据jwt获取用户名等等方法
    @Autowired
    private JwtUtil jwtUtil;
    //用户登录配置类，目前只有一个拦截请求配置。
    @Autowired
    private CustomConfig customConfig;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //如果请求的对象是需要忽略的，不需要权限拦截，则直接过滤，返回。
        if (checkIgnores(request)) {
            filterChain.doFilter(request, response);
            return;
        }
        //否则从 request 的 header 中获取 JWT
        String jwt = jwtUtil.getJwtFromRequest(request);
        //如果jwt存在
        if (StrUtil.isNotBlank(jwt)) {
            try {
                //根据jwt获取登录名
                String username = jwtUtil.getUsernameFromJWT(jwt);
                //拿到登录的vo对象
                UserDetails userDetails = customUserDetailsService.loadUserByUsername(username);
                //验证权限
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                //设置Authentication认证的details
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                //security上下文对象设置认证。
                SecurityContextHolder.getContext().setAuthentication(authentication);
                //执行过滤链
                filterChain.doFilter(request, response);
            } catch (SecurityException e) {
                ResponseUtil.renderJson(response, e);
            }
        } else {
            //jwt不存在则返回响应信息，请先登录
            ResponseUtil.renderJson(response, Status.UNAUTHORIZED, null);
        }

    }

    /**
     * 请求是否不需要进行权限拦截
     *
     * @param request 当前请求
     * @return true - 忽略，false - 不忽略
     */
    private boolean checkIgnores(HttpServletRequest request) {
        //通过request请求对象获取请求方法
        String method = request.getMethod();

        //请求方法不存在则初始化为get请求枚举值
        HttpMethod httpMethod = HttpMethod.resolve(method);
        if (ObjectUtil.isNull(httpMethod)) {
            httpMethod = HttpMethod.GET;
        }

        //set集合，无序唯一，存储需要过滤的请求类型
        Set<String> ignores = Sets.newHashSet();

        //判断请求方法枚举值
        switch (httpMethod) {
            case GET:
                ignores.addAll(customConfig.getIgnores().getGet());
                break;
            case PUT:
                ignores.addAll(customConfig.getIgnores().getPut());
                break;
            case HEAD:
                ignores.addAll(customConfig.getIgnores().getHead());
                break;
            case POST:
                ignores.addAll(customConfig.getIgnores().getPost());
                break;
            case PATCH:
                ignores.addAll(customConfig.getIgnores().getPatch());
                break;
            case TRACE:
                ignores.addAll(customConfig.getIgnores().getTrace());
                break;
            case DELETE:
                ignores.addAll(customConfig.getIgnores().getDelete());
                break;
            case OPTIONS:
                ignores.addAll(customConfig.getIgnores().getOptions());
                break;
            default:
                break;
        }
        //添加需要忽略的请求url
        ignores.addAll(customConfig.getIgnores().getPattern());
        //存在需要忽略的请求url或者请求方法，则遍历一致返回true，不一致返回false
        if (CollUtil.isNotEmpty(ignores)) {
            for (String ignore : ignores) {
                AntPathRequestMatcher matcher = new AntPathRequestMatcher(ignore, method);
                if (matcher.matches(request)) {
                    return true;
                }
            }
        }

        return false;
    }

}

```

