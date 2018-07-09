# Upload

SpringBoot的上传文件，在Controller方法中设置以下参数就可完成文件上传

```java
@PostMapping("/upload") 
public String singleFileUpload(@RequestParam("file") MultipartFile file)
```

另外，上传文件有大小限制，在2.0以上版本中，可在yml中配置

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 1000M
      max-request-size: 1000M
```



