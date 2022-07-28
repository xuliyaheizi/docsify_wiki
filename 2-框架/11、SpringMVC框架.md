# SpringMVC

## SpringMVC常用的注解

1. @EnableWebMvc：在配置类中开启Web MVC的配置支持。

2. @Controller

3. @RequestMapping：用于映射web请求，包括访问路径和参数。

4. @ResponseBody：支持将返回值放到response内，而不是一个页面，通常用户返回json数据。

5. @RequestBody：允许request的参数在request体中，而不是在直接连接的地址后面。（放在参数前）

6. @PathVariable：用于接收路径参数，比如@RequestMapping(“/hello/{name}”)声明的路径，将注解放在参数前，即可获取该值，通常作为Restful的接口实现方法。

7. @RestController：该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。

8. @ControllerAdvice

   - 全局异常处理
   - 全局数据绑定
   - 全局数据预处理

9. @ExceptionHandler：用于全局处理控制器里的异常。

10. @InitBinder：用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。

11. @ModelAttribute

    1. @ModelAttribute注释方法 

       如果把@ModelAttribute放在方法的注解上时，代表的是：该Controller的所有方法在调用前，先执行此@ModelAttribute方法。可以把这个@ModelAttribute特性，应用在BaseController当中，所有的Controller继承BaseController，即可实现在调用Controller时，先执行@ModelAttribute方法。比如权限的验证（也可以使用Interceptor）等。

    2. @ModelAttribute注释一个方法的参数 

       当作为方法的参数使用，指示的参数应该从模型中检索。如果不存在，它应该首先实例化，然后添加到模型中，一旦出现在模型中，参数字段应该从具有匹配名称的所有请求参数中填充。