# Spring Boot项目Resource下文件的读取

## 1. 当项目打成jar包时的读取方法

当项目打成jar包并部署在服务器上时，由于所有文件都存在jar包中，而linux不允许直接读取未解压的文件，因此无法通过路径来读取resource下的文件；

### 方法1：通过ClassResourcePath类通过文件流将文件拷贝到磁盘中

例：

```java
private void copyFile() throws IOException {
    ClassPathResource cpr = new ClassPathResource(".\\model\\mymodel.bin");
    OutputStream outputStream = new FileOutputStream(".\\mymodel.bin");
    outputStream.write(IOUtils.toByteArray(cpr.getInputStream()));
    outputStream.close();
}
```

这样就可以直接读取".\\mymodel.bin"来访问文件

