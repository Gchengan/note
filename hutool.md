# hutool工具

## 导入依赖

```xml
<!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.11</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>5.2.3</version>
        </dependency>
```

## 操作excel

```java
public class ExcelController {

    /*
    *导出数据
    */

    @PostMapping("/export")
    public R<String> getExportData(@RequestBody List<Employee> employee){
     ExcelWriter writer= ExcelUtil.getWriter("e:/employee.xlsx");
     writer.addHeaderAlias("name", "姓名");
     writer.addHeaderAlias("sex", "性别");
     writer.addHeaderAlias("age", "年龄");
     writer.addHeaderAlias("phone", "手机号");
     writer.addHeaderAlias("username", "账号");
     writer.addHeaderAlias("email", "邮箱");
     writer.addHeaderAlias("address", "地址");
// 默认的，未添加alias的属性也会写出，如果想只写出加了别名的字段，可以调用此方法排除之
     writer.setOnlyAlias(true);
// 一次性写出内容，使用默认样式
     writer.write(employee,true);
// 关闭writer，释放内存
     writer.close();
     return R.send("导出成功");
    }

}
```


