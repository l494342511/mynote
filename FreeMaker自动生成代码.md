# FreeMaker自动生成代码

## 1. 编写代码模板

${paraName} 是占位符，用来填入需要的参数

```
package com.tenny.${interfaceName?lower_case}.entity;

import java.util.Date;

public class ${entityName} {

<#list params as param>
	// ${param.fieldNote}
    private ${param.fieldType} ${param.fieldName};

</#list>
<#list params as param>
	public void set${param.fieldName?cap_first}(${param.fieldType} ${param.fieldName}){
        this.${param.fieldName} = ${param.fieldName};
    }

    public ${param.fieldType} get${param.fieldName?cap_first}(){
        return this.${param.fieldName};
    }

</#list>
}
```

## 2. 填充数据

```java
Map<String, Object> beanMap = new HashMap<String, Object>();
beanMap.put("beanName", "User");// 实体类名
beanMap.put("interfaceName", "User");// 接口名
List<Map<String, String>> paramsList = new ArrayList<Map<String, String>>();
for (int i = 0; i < 4; i++) {
      Map<String, String> tmpParamMap = new HashMap<String, String>();
      tmpParamMap.put("fieldNote", "fieldNote" + i);
      tmpParamMap.put("fieldType", "String");
      tmpParamMap.put("fieldName", "fieldName" + i);
      paramsList.add(tmpParamMap);
}
beanMap.put("params", paramsList);
```

## 3. 生成代码

```java
Configuration config = new Configuration();
config.setObjectWrapper(new DefaultObjectWrapper());
Template template = config.getTemplate("src/main/resources/model/entity.ftl", "UTF-8");
Writer out = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("src/main/resources/class/User.java"), "UTF-8"));
template.process(rootMap, out);
out.flush();
out.close();
```

