### 1、Spring  MVC的原理？

1. 客户端（浏览器）发送请求，直接请求到DispatcherServlet。
2. DispatcherServlet根据请求信息调用HandlerMapping，解析请求对应的Handler
3. 解析到对应的Handler后，开始由HandlerAdapter适配器处理
4. HandlerAdapter会根据Handler来调用真正的处理器开处理请求，并处理相应的业务逻辑
5. 处理器处理完业务后，会返回一个ModelAndView对象，Model是返回的数据对象，View是个逻辑上的View
6. ViewResolver会根据逻辑View查找实际的View
7. DispaterServlet把返回的Model传给View
8. 通过View返回给请求者（浏览器）