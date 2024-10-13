日期：2024-10-11
# 文档说明
## 实验要求
题目: 实现一个登录验证过滤器
目标: 创建一个 Servlet的 过滤器,用于验证用户是否已登录。对于未登录的用户,将其重定向到登录页面。
要求: 
1. 创建一个名为 LoginFilter 的类, 实现 javax.servlet.Filter 接口。
2. 使用 @WebFilter 注解配置过滤器,使其应用于所有 URL 路径 ("/*")。
3. 在 doFilter 方法中实现以下逻辑: 
  a. 检查当前请求是否是对登录页面、注册页面或公共资源的请求。如果是,则允许请求通过。 
  b. 如果不是上述情况,检查用户的 session 中是否存在表示已登录的属性(如 "user" 属性)。
  c. 如果用户已登录,允许请求继续。 
  d. 如果用户未登录,将请求重定向到登录页面。
4. 创建一个排除列表,包含不需要登录就能访问的路径(如 "/login", "/register", "/public")。
5. 实现一个方法来检查当前请求路径是否在排除列表中。
6. 添加适当的注释,解释代码的主要部分。

## 主要组件
### LoginFilter类实现逻辑
使用 @WebFilter 注解配置过滤器，适用于所有 URL 路径。
在 doFilter 方法中实现以下逻辑：
检查请求是否为登录页面、注册页面或公共资源，如果是，则允许请求通过。
检查用户 session 中是否存在已登录的属性（如 "user"）。
如果用户已登录，允许请求继续。
如果用户未登录，重定向到登录页面。
#### LoginFilter类

```java
package org.example;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;

@WebFilter(filterName = "LoginFilter",urlPatterns = "/*")
public class LoginFilter implements Filter {
    // 排除列表，不需要登录就能访问的路径
    private static final List<String> excludePaths = Arrays.asList("/","/login", "/register", "/index",
            "/index.jsp", "/login.jsp", "/register.jsp");

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        HttpSession session = request.getSession();
        // 获取请求的URI
        String requestURI = request.getRequestURI();
        System.out.println("URI：");
        System.out.println(requestURI);

        // 检查请求是否在排除列表中
        if (isExcluded(requestURI)) {
            System.out.println("路径：" + requestURI + "已放行");
            chain.doFilter(request, response);
        } else {
            // 检查用户是否已登录
            if (session != null && session.getAttribute("user") != null) {
                // 用户已登录，继续请求
                chain.doFilter(request, response);
            } else {
                // 用户未登录，重定向到登录页面
                response.sendRedirect(request.getContextPath() + "/login.jsp");
            }
        }
    }

    @Override
    public void destroy() {
        //Filter.super.destroy();
        // 在过滤器销毁时执行清理工作
    }
    /**
     * 检查当前请求路径是否在排除列表中
     * @param requestURI 当前请求的URI
     * @return 如果在排除列表中返回true，否则返回false
     */
    private boolean isExcluded(String requestURI) {
        /*for (String path : excludePaths) {
            if (requestURI.startsWith(path)) {
                return true;
            }
        }*/
        if (excludePaths.contains(requestURI)) {
            return true;
        }
        return false;
    }
}


```

