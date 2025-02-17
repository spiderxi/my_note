# Tomcat简介
## 1. Servlet规范
*讲一下javax.Servlet接口生命周期涉及的Hook方法?*
```
1️⃣ init(): Servlet被容器初始化时调用
2️⃣ service(): 处理请求时调用
3️⃣ destroy(): Servlet被容器销毁时调用
```

*Servlet容器有哪两种扩展机制?*
```
🌟 Filter#doFilter(): 处理请求时调用
🌟 Listener: 监听Servlet容器内部的事件
```
