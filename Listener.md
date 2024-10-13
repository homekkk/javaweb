# 实验步骤
1.创建 RequestLoggingListener 类：
2.实现 ServletRequestListener 接口。
3.在 requestInitialized 方法中记录请求的详细信息，并将其存储为请求的属性。
创建测试 Servlet TestLoggingServlet：
4.在 Servlet 中获取存储的日志信息。
将日志信息通过 JavaScript 输出到浏览器的控制台。

## RequestLoggingListener类

```java
package org.example;

import jakarta.servlet.ServletRequestEvent;
import jakarta.servlet.ServletRequestListener;
import jakarta.servlet.annotation.WebListener;
import jakarta.servlet.http.HttpServletRequest;

import java.util.HashMap;
import java.util.Map;
@WebListener
public class RequestLoggingListener implements ServletRequestListener{

    private static final Map<String, Long> requestStartTimes = new HashMap<>();
    private MySessionListener mySessionListener = new MySessionListener();

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        // 记录请求开始处理时间
        long startTime = System.currentTimeMillis();
        requestStartTimes.put(request.getRequestURI(), startTime);
        mySessionListener.addRequestCount(sre);

        // 记录请求开始的日志
        logRequest(request, "Request started");
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("本次请求处理完成");
        mySessionListener.deleteRequestCount(sre);
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        String requestURI = request.getRequestURI();
        Long startTime = requestStartTimes.remove(requestURI);
        if (startTime != null) {
            long endTime = System.currentTimeMillis();
            long processingTime = endTime - startTime;

            // 记录请求结束的日志
            logRequest(request, "Request finished", processingTime);
        }
    }

    private void logRequest(HttpServletRequest request, String status) {

        logRequest(request, status, -1);
    }

    private void logRequest(HttpServletRequest request, String status, long processingTime) {
        StringBuilder log = new StringBuilder();
        log.append("[").append(request.getRequestURL().toString()).append("] \n ");
        log.append("Client IP: ").append(request.getRemoteAddr()).append(" \n ");
        log.append("Method: ").append(request.getMethod()).append(" \n ");
        log.append("URI: ").append(request.getRequestURI()).append(" \n ");
        log.append("QueryString: ").append(request.getQueryString() != null ? request.getQueryString() : "").append(" \n ");
        log.append("User-Agent: ").append(request.getHeader("User-Agent")).append(" \n ");
        log.append("Status: ").append(status);
        if (processingTime != -1) {
            log.append(" \n Processing Time: ").append(processingTime).append(" ms");
        }
        log.append("\n");

        // 这里可以将日志输出到控制台，或者写入到日志文件
        System.out.println("==========================Log-Start======================");
        System.out.println(log.toString());
        System.out.println("==========================Log-End======================");
    }
}
```