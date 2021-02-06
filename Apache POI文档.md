# **Apache POI文档**

## 1. XWPF——操作word

### 1.1 XWPFDocument

**文档对象**

```java
InputStream is = new FileInputStream("E:\\项目\\03-智能文档推送\\测试文档\\测试word内容解析的文档.docx");
XWPFDocument document = new XWPFDocument(is);
```

### 1.2 XWPFParagraph

**段落对象**

```java
XWPFDocument document = new XWPFDocument(is);
List<XWPFParagraph> list=document.getParagraphs();
```

### 1.3 XWPFTable

**表格对象**

```java
XWPFDocument document = new XWPFDocument(is);
List<XWPFTable> list=document.getTables();
```

### 1.4 XWPFSDT

