---
title: SpringMvc 实现文件上传
tags: study
---
## 前言

  
- 在使用springMVC进行系统实现时，springMVC默认的解析器里面是没有加入对文件上传的解析的，这可以方便我们实现自己的文件上传。
- 但如果你想使用springMVC对文件上传的解析器来处理文件上传的时候就需要在spring的applicationContext里面加上springMVC提供的MultipartResolver的声明。
- 这样之后，客户端每次进行请求的时候，springMVC都会检查request里面是否包含多媒体信息，如果包含了就会使用MultipartResolver进行解析, springMVC会使用一个支持文件处理的MultipartHttpServletRequest来包裹当前的HttpServletRequest，然后使用MultipartHttpServletRequest就可以对文件进行处理了。
- Spring已经为我们提供了一个MultipartResolver的实现，我们只需要拿来用就可以了，那就是*org.springframework.web.multipart.commons.CommsMultipartResolver*。
- 因为springMVC的MultipartResolver底层使用的是Commons-fileupload，所以还需要加入对Commons-fileupload.jar的支持。


## 导入依赖


```xml-dtd

<!-- 文件上传依赖 -->

<dependency>

    <groupId>commons-io</groupId>

    <artifactId>commons-io</artifactId>

    <version>2.11.0</version>

</dependency>

<dependency>

    <groupId>commons-fileupload</groupId>

    <artifactId>commons-fileupload</artifactId>

    <version>1.4</version>

</dependency>

```

## 添加类型支持

在springmvc.xml中配置类型支持
  
```xml

<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">

    <!--限制文件上传大小-->

    <property name="maxUploadSize" value="104856"/>

</bean>

```


## 后端代码

```java

@RestController

@RequestMapping("/fupload")

public class FileUploadController {

    @PostMapping("/")

    public String uoload(@RequestParam("name") String name,

                       @RequestParam("file")MultipartFile file) throws IOException {

        if(!file.isEmpty()){

            //指定存储位置存储

            File targetFile = new File("D:\\", file.getOriginalFilename());

            FileUtils.writeByteArrayToFile(targetFile, file.getBytes());

            return "ok";
        }

        return "false";

    }

}

```

## 前端
> 在上传时涉及到文件需要修改media type