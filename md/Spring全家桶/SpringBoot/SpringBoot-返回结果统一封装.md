# SpringBoot-返回结果统一封装

## 为什么要对SpringBoot返回结果进行统一封装？

目前主流开发方式都是前后端分离的，在这种大背景下，对 SpringBoot 应用程序中的返回结果进行统一封装是一种良好的实践，因为它有助于提高代码的可维护性、可读性和可扩展性，以及前后端协同开发的顺利进行。以下是一些主要理由：

- 规范的接口定义：规范的接口定义确保前后端开发团队有共同的理解，使得 API 的使用和响应方式清晰可见。这有助于消除误解，减少沟通问题，提高项目的整体可维护性。
- 统一的异常处理：规范的接口定义确保了一致的异常处理方式，不论是数据验证异常、业务逻辑异常还是服务器异常，都有统一的异常响应结构。
  - 使前端开发团队更容易处理各种异常情况，减少了开发时间和复杂的异常处理逻辑，提高了前端应用程序的可靠性，同时还改善了用户体验，提供更精确的错误信息，有助于用户更好地理解问题。
  - 同时也简化了后端开发人员的故障排除过程，因为他们能够快速定位和解决问题，而不必处理各种不同的异常情况。
- 可扩展性：规范的接口定义允许你轻松地扩展响应格式，以适应应用程序的增长和需求变化。这意味着你可以添加新的元数据或更改现有属性，而不必修改所有的控制器方法。
- 安全性：规范的接口定义有助于确保敏感数据不会不小心泄露到前端，从而提高应用程序的安全性。
- 易于测试：规范的接口定义使得单元测试和集成测试更加容易。前端和后端开发人员都可以预测API的响应结构，从而更容易地进行测试。

综上所述，规范的接口定义在前后端分离开发中起着关键作用，帮助实现协同开发、可维护性和规范的 API 使用。这有助于确保前后端团队能够有效地协同工作，同时满足应用程序的可扩展性、安全性和性能需求。这是一种良好的开发实践，特别是在大型应用程序中，可以有效管理和维护不断增长的代码库。

## 实现步骤

### 响应码接口

```java
public interface IResultCode {
    Integer getCode();

    String getMsg();
}
```

### 响应码枚举

```java
public enum ResultCode implements IResultCode {
    SUCCESS(200, "请求成功"),
    SYSTEM_ERROR(500, "系统内部错误"),
    PARAM_TYPE_ERROR(400, "参数类型不匹配"),
    NOT_FOUND(404, "资源不存在");


    private final Integer code;
    private final String msg;

    ResultCode(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    @Override
    public Integer getCode() {
        return code;
    }

    @Override
    public String getMsg() {
        return msg;
    }
}
```

### 返回结果类

```java
@Data
@Builder
public class Result<T> {
    private Boolean success;
    private Integer code;
    private String msg;
    private T data;

    public static <T> Result<T> success(T data) {
        return Result.<T>builder()
                .success(true)
                .code(ResultCode.SUCCESS.getCode())
                .msg(ResultCode.SUCCESS.getMsg())
                .data(data)
                .build();
    }

    public static <T> Result<T> fail(Integer code, String msg) {
        return Result.<T>builder().success(false).code(code).msg(msg).build();
    }

    public static <T> Result<T> fail(IResultCode resultCode) {
        return Result.<T>builder()
                .success(false)
                .code(resultCode.getCode())
                .msg(resultCode.getMsg())
                .build();
    }
}
```

- success：表示请求是否成功的布尔值；
- code：表示响应的状态码，通常为6位，例如 A 业务：100000-199999，B 业务：200000-299999，以此类推，不同的数字开头代表不同的业务；
- msg：包含响应消息或错误消息的字符串，用于展示给用户看的；
- data：用于封装响应数据的泛型对象；

至此咱们可以先写一个 `TestController` 类进行测试一下，如下所示：

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("/success")
    public Result<Map<String, Object>> success() {
        Integer age = 18;
        String name = "小让";
        Map<String, Object> map = new HashMap<>();
        map.put("name", name);
        map.put("age", age);
        return Result.success(map);
    }

    @GetMapping("/error")
    public Result error() {
        return Result.fail(ResultCode.SYSTEM_ERROR);
    }
}
```

首先访问一下 http://localhost:9527/test/success ，测试结果如下所示：<br />![image-20231012184022037](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310121840217.png)

再访问一下 http://localhost:9527/test/error ，测试结果如下所示：<br />![image-20231012184212564](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310121842612.png)

### 响应拦截器

虽然上面的代码可以实现功能，不过较为繁琐，对于每一个接口都需要使用 `Result` 中的 `success()` 或 `fail()` 方法对结果进行包装，重复劳动，所以需要对代码进行优化，那么该如何优化呢？此时就需要借助 SpringBoot 中的 `ResponseBodyAdvice` 拦截器。

> [!TIP]
>
> `ResponseBodyAdvice` 是 Spring Framework中的一个接口，它允许你全局性地修改控制器方法的响应体（Response Body）。通常，你可以使用这个接口来统一处理和修改控制器方法的响应，例如**将响应结果封装为统一的数据结构**，添加通用的响应头，对数据进行加工，实现国际化支持等。这有助于实现全局的响应定制和增加代码的可维护性。
>
> 具体实现步骤如下所示：
>
> 1. 创建一个实现 `ResponseBodyAdvice` 接口的类，类上需要标注 `RestControllerAdvice` 或 `ControllerAdvice` 注解，并且指定应用的包路径；
>
>    ```java
>    import org.springframework.web.bind.annotation.RestControllerAdvice;
>    import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;
>    
>    @RestControllerAdvice
>    public class CustomResponseBodyAdvice implements ResponseBodyAdvice<Object> {
>        // 实现ResponseBodyAdvice接口中的方法
>    }
>    ```
>
> 2. 实现 `supports()` 方法：该方法用于确定是否应用 `beforeBodyWrite()` 方法来处理响应。你可以在该方法中编写逻辑以决定是否应该对特定响应进行处理。例如，你可以根据控制器方法的返回类型或其他条件来决定是否应该处理响应。
>
>    ```java
>    @Override
>    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
>        // 根据条件判断是否应该处理响应
>        // 返回true表示应该处理，返回false表示不处理
>    }
>    ```
>
> 3. 实现`beforeBodyWrite()`方法：该方法在控制器方法的响应体写入到 HTTP 响应之前被调用。在这里，你可以修改响应体、添加通用数据、进行国际化处理或其他自定义操作。
>
>    ```java
>    @Override
>    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
>            Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
>        // 修改响应体、添加通用数据、进行国际化处理等操作
>        // 返回修改后的响应体
>    }
>    ```

```java
@RestControllerAdvice
public class ResponseInterceptor implements ResponseBodyAdvice<Object> {
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @SneakyThrows
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if (body == null) {
            return Result.success();
        }
        if (body instanceof Result) {
            return body;
        }
        return Result.success(body);
    }
}
```

增加响应拦截器后，可以将原来 `TestController` 中的方法修改至如下所示：

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("/success")
    public Map<String, Object> success() {
        Integer age = 18;
        String name = "小让";
        Map<String, Object> map = new HashMap<>();
        map.put("name", name);
        map.put("age", age);
        return map;
    }

    @GetMapping("/error")
    public Result error() {
        return Result.fail(ResultCode.SYSTEM_ERROR);
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

为了演示一种特殊情况，当方法返回值为 `String` 类型时，即 `hello()` 方法那种情况，如果 `ResponseInterceptor` 响应拦截器不作任何改动，此时访问 http://localhost:9527/test/hello ，会抛出如下所示异常！<br />![image-20231012193013855](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310121930930.png)

因此针对这一特殊情况， `ResponseInterceptor` 响应拦截器中的 `beforeBodyWrite()` 方法需要添加额外分支情况，如下所示：

```java
@RestControllerAdvice(basePackages = "fun.xiaorang.result.controller")
public class ResponseInterceptor implements ResponseBodyAdvice<Object> {
    private final ObjectMapper objectMapper;

    public ResponseInterceptor(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @SneakyThrows
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if (body == null) {
            return Result.success();
        }
        if (body instanceof Result) {
            return body;
        }
        if (body instanceof String) {
            response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
            return objectMapper.writeValueAsString(Result.success(body));
        }
        return Result.success(body);
    }
}
```

### 自定义业务异常类

继承自`RuntimeException`或其子类，以便将应用程序特定的异常与标准异常区分开来。

```java
public class BusinessException extends RuntimeException {
    private final Integer code;
    private final String msg;

    public BusinessException(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public BusinessException(IResultCode resultCode) {
        this.code = resultCode.getCode();
        this.msg = resultCode.getMsg();
    }

    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

### 全局异常处理器

> [!TIP]
>
> 全局异常处理器是 SpringBoot 应用程序中的一个组件，**用于捕获和处理应用程序中发生的未被捕获的异常**，以确保应用程序的稳定性和用户友好性。全局异常处理器通常用于**统一处理异常**，**返回一致的错误响应**，或执行其他与异常处理相关的操作。
>
> 具体实现步骤如下所示：
>
> 1. 创建全局异常处理器类，通常标注 `@RestControllerAdvice` 注解，以标识该类是一个全局异常处理器。
>
>    ```java
>    import org.springframework.web.bind.annotation.RestControllerAdvice;
>    import org.springframework.web.bind.annotation.ExceptionHandler;
>    
>    @RestControllerAdvice
>    public class GlobalExceptionHandler {
>        // 异常处理方法将在此类中定义
>    }
>    ```
>
> 2. 定义异常处理方法：在全局异常处理器类中，定义一个或多个异常处理方法，通常使用 `@ExceptionHandler` 注解来标识要处理的异常类型。在这些方法中，你可以捕获异常、执行自定义处理逻辑，以及返回适当的错误响应。
>
>    ```java
>    @ExceptionHandler(Exception.class)
>    public ResponseEntity<Result<Void>> handleException(Exception e) {
>        // 捕获异常，执行处理逻辑，返回错误响应
>    }
>    ```

如下所示，列举出日常开发中比较常见的几种异常捕获方法，当以上都不满足时，存在最终的兜底方案，那就是捕获 `Exception` 异常！

> [!TIP]
>
> 当访问一个不存在的资源时，服务器会抛出一个 Not Found 的错误，如果想捕获该 `NoHandlerFoundException` 异常，除了在全局异常处理器类中增加一个异常处理方法之外，还需要在 `application.yml` 配置文件中添加如下配置，这样才能捕获到 `NoHandlerFoundException` 异常！
>
> ```yaml
> spring:
>       mvc:
>          throw-exception-if-no-handler-found: true
>       resources:
>          add-mappings: false
>    ```

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<Result<Void>> handleException(MethodArgumentTypeMismatchException e) {
        log.error("参数类型不匹配异常信息，异常堆栈信息：{}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(Result.fail(ResultCode.PARAM_TYPE_ERROR));
    }

    @ExceptionHandler(ArithmeticException.class)
    public ResponseEntity<Result<Void>> handleException(ArithmeticException e) {
        log.error("算术异常信息，异常堆栈信息：{}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Result.fail(ResultCode.SYSTEM_ERROR));
    }
    
    @ExceptionHandler(NoHandlerFoundException.class)
    public ResponseEntity<Result<Void>> handleException(NoHandlerFoundException e) {
        log.error("404异常信息，异常堆栈信息：{}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(Result.fail(ResultCode.NOT_FOUND));
    }

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<Result<Void>> handleException(BusinessException e) {
        log.error("业务异常信息，异常堆栈信息：{}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Result.fail(e.getCode(), e.getMsg()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Result<Void>> handleException(Exception e) {
        log.error("全局异常信息，异常堆栈信息：{}", e.getMessage(), e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Result.fail(ResultCode.SYSTEM_ERROR));
    }
}
```

现在，修改一下 `TestController` 类中的 `error()` 方法，让其真正抛出异常，检验一下咱们的全局异常处理器是否能处理抛出的异常。改动后的代码如下所示：

```java
@GetMapping("/error")
public Result error() {
    int i = 1 / 0;
    return Result.fail(ResultCode.SYSTEM_ERROR);
}
```

根据咱们的经验可知，该方法会抛出 `ArithmeticException` 异常！访问一下  http://localhost:9527/test/error ，控制台打印异常信息：<br />![image-20231013000302829](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310130003899.png)返回结果如下所示：<br />![image-20231013001923138](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310130019196.png)

对于 `MethodArgumentTypeMismatchException` 参数类型不匹配的异常，咱们在 `TestController` 中增加如下方法用于测试，

```java
@GetMapping("/error2")
public Integer error2(Integer age) {
    return age;
}
```

当咱们访问 http://localhost:9527/test/error2?age=10 时，该方法并不会抛出异常，而访问 http://localhost:9527/test/error2?age=cc 时，由于 `String` 类型无法强转为 `Integer` 类型，此时就会抛出 `MethodArgumentTypeMismatchException` 参数类型不匹配的异常，控制台打印如下异常信息：<br />![image-20231013003232006](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310130032091.png)返回结果如下所示：<br />![image-20231013003513618](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202310130035685.png)

至此，咱们的全局异常处理器已经生效，能够捕获和处理程序中抛出的异常，并且返回封装好的结果类！

