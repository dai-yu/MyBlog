# SpringBoot 学习笔记6 - 返回统一的json格式



## 1. 创建返回对象泛型类

统一对象类： `Result.java`

```java
package com.example.demo.entity.common;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 21:38
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description 统一返回json格式
 */
public class Result<T> {

    /**
     * 错误码
     */
    private Integer code;

    /**
     * 提示信息
     */
    private String msg;

    /**
     * 数据
     */
    private ResultBean<T> result;

    // 省略 getter 和 setter 方法
}
```

其中，`ResultBean`的定义为：

`ResultBean.java`

```java
package com.example.demo.entity.common;

import java.util.List;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 22:51
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
public class ResultBean<T> {

    private Integer total;
    private List<T> data;

    // 省略 getter 和 setter 方法
}
```

## 2. 创建异常类

错误定义枚举：`ErrorEnum.java`

```java
package com.example.demo.enums;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 22:35
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
public enum ErrorEnum {

    BOOK_NAME_NULL_ERROR(10001, "book name is null."),
    HELLO_NAME_NULL_ERROR(20001, "hi name is null.")
    ;


    Integer errorCode;
    String errorMessage;

    ErrorEnum(Integer errorCode, String errorMessage) {
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }

    public Integer getErrorCode() {
        return errorCode;
    }

    public String getErrorMessage() {
        return errorMessage;
    }
}
```

异常类：`DemoException.java`

```java
package com.example.demo.exception;

import com.example.demo.enums.ErrorEnum;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 22:31
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description 统一异常
 */
public class DemoException extends RuntimeException {

    private final ErrorEnum errorEnum;

    public DemoException(ErrorEnum errorEnum) {
        this.errorEnum = errorEnum;
    }

    public ErrorEnum getErrorEnum() {
        return errorEnum;
    }
}
```

## 3. 创建工具类

创建工具类 `ResultUtils.java` ，进行封装返回成功信息，异常信息。

```java
package com.example.demo.utils;

import com.example.demo.entity.common.Result;
import com.example.demo.entity.common.ResultBean;
import com.example.demo.enums.ErrorEnum;
import com.example.demo.exception.DemoException;

import java.util.ArrayList;
import java.util.List;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 22:19
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
public class ResultUtils {

    /**
     * 成功返回
     *
     * @param data 返回数据
     * @param <T>  数据类型
     * @return 统一的返回值
     */
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(0);
        result.setMsg("request successful.");
        ResultBean<T> resultBean = new ResultBean<>();
        resultBean.setTotal(1);
        List<T> dataList = new ArrayList<>(1);
        dataList.add(data);
        resultBean.setData(dataList);
        result.setResult(resultBean);
        return result;
    }

    public static <T> Result<T> success(List<T> dataList){
        Result<T> result = new Result<>();
        result.setCode(0);
        result.setMsg("request successful.");
        ResultBean<T> resultBean = new ResultBean<>();
        resultBean.setTotal(dataList.size());
        resultBean.setData(dataList);
        result.setResult(resultBean);
        return result;
    }
    /**
     * 异常返回
     *
     * @param demoException
     * @param <T>
     * @return
     */
    public static <T> Result<T> error(DemoException demoException) {
        Result<T> result = new Result<>();
        result.setCode(demoException.getErrorEnum().getErrorCode());
        result.setMsg(demoException.getErrorEnum().getErrorMessage());
        return result;
    }

    /**
     * 异常返回
     * @param errorEnum
     * @param <T>
     * @return
     */
    public static <T> Result<T> error(ErrorEnum errorEnum) {
        Result<T> result = new Result<>();
        result.setCode(errorEnum.getErrorCode());
        result.setMsg(errorEnum.getErrorMessage());
        return result;
    }
}
```

## 4. 实践

以 `BookController` 为例，进行正常返回对象，正常返回数组，异常返回。

```java
package com.example.demo.controller;

import com.example.demo.entity.Book;
import com.example.demo.entity.common.Result;
import com.example.demo.exception.DemoException;
import com.example.demo.service.IBookService;
import com.example.demo.utils.ResultUtils;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2018/12/10 23:07
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
@RestController
@RequestMapping(value = "/v1/book", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@Api("Book 接口")
public class BookController {

    @Autowired
    private IBookService bookService;

    @RequestMapping(value = "/book-list", method = RequestMethod.POST)
    @ApiOperation("返回测试的 book 列表")
    public Result<Book> bookList() {
        try {
            List<Book> books = bookService.bookList();
            return ResultUtils.success(books);
        } catch (DemoException demoException) {
            return ResultUtils.error(demoException);
        }

    }

    @RequestMapping(value = "one-book", method = RequestMethod.POST)
    public Result<Book> oneBook(@RequestBody Book book) {
        try {
            Book book1 = bookService.oneBook(book.getName(), book.getAuther(), book.getPublisher());
            return ResultUtils.success(book1);
        } catch (DemoException demoException) {
            return ResultUtils.error(demoException);
        }
    }

}
```

**对应`IService`文件：`IBookService.java`**

```java
package com.example.demo.service;

import com.example.demo.entity.Book;

import java.util.List;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 23:01
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
public interface IBookService {

    /**
     * 返回测试列表
     * @return
     */
    List<Book> bookList();

    /**
     * 构造一个书的对象
     * @param name
     * @param author
     * @param publisher
     * @return
     */
    Book oneBook(String name, String author, String publisher);

}

```

**Service实现类：`BookServiceImpl.java`**

```java
package com.example.demo.service.impl;

import com.example.demo.entity.Book;
import com.example.demo.exception.DemoException;
import com.example.demo.service.IBookService;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

import static com.example.demo.enums.ErrorEnum.BOOK_NAME_NULL_ERROR;

/**
 * @author youngbear
 * @email youngbear@aliyun.com
 * @date 2019-04-30 23:05
 * @blog https://blog.csdn.net/next_second
 * @github https://github.com/YoungBear
 * @description
 */
@Service
public class BookServiceImpl implements IBookService {
    @Override
    public List<Book> bookList() {
        List<Book> books = new ArrayList<>();
        Book b1 = new Book();
        b1.setName("数学之美");
        b1.setPublisher("人民邮电出版社");
        b1.setAuther("吴军");
        Book b2 = new Book();
        b2.setName("重构 改善既有代码的设计");
        b2.setPublisher("人民邮电出版社");
        b2.setAuther("Martin Fowler");
        Book b3 = new Book();
        b3.setName("机器学习实战");
        b3.setPublisher("人民邮电出版社");
        b3.setAuther("Peter Harrington");
        Book b4 = new Book();
        b4.setName("Effective Java中文版");
        b4.setPublisher("机械工业出版社");
        b4.setAuther("Joshua Bloch");
        books.add(b1);
        books.add(b2);
        books.add(b3);
        books.add(b4);
        return books;
    }

    @Override
    public Book oneBook(String name, String author, String publisher) {
        if (null == name) {
            throw new DemoException(BOOK_NAME_NULL_ERROR);
        }
        Book book = new Book();
        book.setName(name);
        book.setAuther(author);
        book.setPublisher(publisher);
        return book;
    }
}

```



### 4.1 获取列表

**请求：**

```shell
curl -X POST "http://localhost:8080/v1/book/book-list" -H  "accept: application/json;charset=UTF-8"
```

**对应返回值：**

```json
{
  "code": 0,
  "msg": "request successful.",
  "result": {
    "total": 4,
    "data": [
      {
        "name": "数学之美",
        "publisher": "人民邮电出版社",
        "author": "吴军"
      },
      {
        "name": "重构 改善既有代码的设计",
        "publisher": "人民邮电出版社",
        "author": "Martin Fowler"
      },
      {
        "name": "机器学习实战",
        "publisher": "人民邮电出版社",
        "author": "Peter Harrington"
      },
      {
        "name": "Effective Java中文版",
        "publisher": "机械工业出版社",
        "author": "Joshua Bloch"
      }
    ]
  }
}
```

### 4.2 生成单个对象

**请求：**

```shell
curl -X POST "http://localhost:8080/v1/book/one-book" -H  "accept: application/json;charset=UTF-8" -H  "Content-Type: application/json" -d "{  \"author\": \"毛泽东\",  \"name\": \"毛泽东选集\",  \"publisher\": \"人民出版社\"}"
```

**返回结果：**

```json
{
  "code": 0,
  "msg": "request successful.",
  "result": {
    "total": 1,
    "data": [
      {
        "name": "毛泽东选集",
        "publisher": "人民出版社",
        "author": "毛泽东"
      }
    ]
  }
}
```

### 4.3 返回异常信息：

**请求：**

```shell
curl -X POST "http://localhost:8080/v1/book/one-book" -H  "accept: application/json;charset=UTF-8" -H  "Content-Type: application/json" -d "{  \"author\": \"毛泽东\",  \"publisher\": \"人民出版社\"}"
```

**返回结果：**

```json
{
  "code": 10001,
  "msg": "book name is null.",
  "result": null
}
```

用postman请求也可以实现同样的效果。具体可参考源代码。



## [Demo GitHub地址](https://github.com/YoungBear/SpringBootDemo)



## SpringBoot 学习笔记

### [1. HelloWorld](./SpringBoot-1-HelloWorld.md)

### [2. logback 日志配置](./SpringBoot-2-logback.md)

### [3. 返回 Json 串](./SpringBoot-3-Json.md)

### [4. Tomcat 部署](./SpringBoot-4-Tomcat.md)

### [5. Swagger-ui](./SpringBoot-5-Swagger-ui.md)

### [6. 返回统一的Json格式](./SpringBoot-6-CommonJson.md)

### [7. 处理全局异常](./SpringBoot-7-GlobalExceptionHandler.md)

### [8. GsonUtils 工具类](./SpringBoot-8-GsonUtils.md)

### [9. 多环境支持](./SpringBoot-9-MultipyEnv.md)

### [10. 集成数据库](./SpringBoot-10-Database.md)

