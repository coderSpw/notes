# SpringMVC源码解析

## 1. 初始化化九大组件

- MultipartResolver：文件上传解析器

- LocaleResolver：国际化解析器

- ThemeResolver：模板解析器

- HandlerMapping：处理器映射器

- HandlerAdapter：处理器适配器

- ViewResolver：视图解析器

- RequestToViewNameTranslator：请求视图名解释器

- HandlerExceptionResolver：处理器异常解析器

- FlashMapManger：重定向参数管理器

  ![image-20220516005117651](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005117651.png)

## 2. 执行流程源码解析

1. 执行DispatcherServlet类的doService方法，调用doDispatch方法；

2. 判断当前请求是否为MultiRequest请求，是的话进行强转；

3. 执行getHandler方法获取处理器执行链，通过通过request请求找到对应的handlerMapping处理器映射器，根据处理器映射器获取处理器执行链；

   ![image-20220516005212700](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005212700.png)

4. 执行getAdapter方法根据处理器获取对应的handlerAdapter处理器适配器；

   ![image-20220516005239552](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005239552.png)

5. 执行applyPreHandle方法，正向遍历拦截器，调用拦截器的preHandle方法；

   ![image-20220516010329825](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516010329825.png)

6. 执行handle方法，调用处理器适配器的handle方法获取ModelAndView对象；

   ![image-20220516005702305](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005702305.png)

7. 执行applyDefaultViewName方法，如果modelAndView中视图不存在，使用RequestToViewNameTranslator从请求中获取视图名赋值到modelAndView对象中；

   ![image-20220516005631562](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005631562.png)

8. 执行applyPostHandle方法，反向遍历拦截器，调用拦截器的postHandle方法；

   ![image-20220516010355679](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516010355679.png)

9. 执行processDispatchResult方法， 内部调用render方法；

   ![image-20220516005753645](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005753645.png)

10. 执行resolveViewName方法，从modelAndView对象中获取viewName视图名，根据视图名和locale，利用视图解析器生成View对象；

    ![image-20220516005834838](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005834838.png)

    ![image-20220516005850438](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005850438.png)

11. 调用view对象的render方法将model数据渲染到view上；

    ![image-20220516005918936](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220516005918936.png)

12. 将view返回给中央控制器，调用servlet的doDispatch方法响应给浏览器；
