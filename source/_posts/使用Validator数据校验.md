---
title: 使用Validator数据校验
date: 2022-07-06 10:41:31
tags:
- validator
categorys:
- validator
---
# 一 传统的数据校验方法
在传统的数据校验时,我们通常在controller中接收到前端传来的参数进行校验
## 1.doamin
```java
package com.njpi.xyh.domain;

import lombok.Data;

import java.time.LocalDateTime;

/**
 * @author: xyh
 * @create: 2022/7/6 11:16
 */
@Data
public class Blog {
    private Integer id ;
    private Integer userId;
    private String title;
    private String context;
    private LocalDateTime createTime;
}
```
## 2.controller
```java
package com.njpi.xyh.controller;

import com.google.gson.Gson;
import com.njpi.xyh.domain.Blog;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;

/**
 * @author: xyh
 * @create: 2022/7/6 11:18
 */
@RestController
@RequestMapping("blog")
public class BlogController {

    /**
     * 添加一个博客
     *
     * @param blog
     * @return
     */
    @PostMapping
    public String add(@RequestBody Blog blog) {
        /**
         * 1. id  --- > 必须为空
         * 2. userId --- > 不能为空
         * 3. title ----> 不能为空,且长度>6 && < 20
         * 4. context ----> 可以为null
         * 5. createTime ---> 必须自己生成
         */

        if (blog.getId() != null) {
            return "blog id 不能存在";
        }

        if (blog.getUserId() == null) {
            return "blog userId 不能为空";
        }

        if (!StringUtils.hasText(blog.getTitle())) {
            return "blog title 不能为空";
        }

        if(blog.getTitle().trim().length()> 20 || blog.getTitle().trim().length()< 6){
            return "blog title 长度不符合";
        }

        // 设置当前时间
        blog.setCreateTime(LocalDateTime.now());


        Gson gson = new Gson();
        String res = gson.toJson(blog);
        return res;

    }

}

```

## 3.测试
```Java
POST http://localhost:8080/blog
Content-Type: application/json

{
  "id": "123456"
}
```

## 4.结果

<img src="https://hexo-blog-repo.oss-cn-hangzhou.aliyuncs.com/blog-pic-repo/202207061137783.png" alt="image-20220706113715704" style="zoom:150%;" />

## 5.总结

每一次不管时新增还是修改都要进行一大推的数据校验的工作,比较费事和费力,这和我们的业务逻辑没有太大的关系



# 二 使用Validator进行校验

## 1.导入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
```

## 2.基本使用

### 2.1 修改bean

```java
package com.njpi.xyh.domain;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import javax.validation.constraints.*;
import java.time.LocalDateTime;

/**
 * @author: xyh
 * @create: 2022/7/6 11:16
 */
@Data
public class Blog {
    @Null
    private Integer id ;
    @NotNull
    private Integer userId;
    @NotNull
    @Max(value = 20)
    @Min(value = 6)
    private String title;
    private String context;
    @Null // 日期必须为空 我们要自己生成
    private LocalDateTime createTime;
}

```

### 2.2 修改controller

```java
package com.njpi.xyh.controller;

import com.google.gson.Gson;
import com.njpi.xyh.domain.Blog;
import org.springframework.util.StringUtils;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.time.LocalDateTime;

/**
 * @author: xyh
 * @create: 2022/7/6 11:18
 */
@RestController
@RequestMapping("blog")
public class BlogController {

    /**
     * 添加一个博客
     *
     * @param blog
     * @return
     */
    @PostMapping
    public String add(@Validated @RequestBody Blog blog) {
        /**
         * 1. id  --- > 必须为空
         * 2. userId --- > 不能为空
         * 3. title ----> 不能为空,且长度>6 && < 20
         * 4. context ----> 可以为null
         * 5. createTime ---> 必须自己生成
         */

        // 设置当前时间
        blog.setCreateTime(LocalDateTime.now());


        Gson gson = new Gson();
        String res = gson.toJson(blog);
        return res;

    }

}

```



