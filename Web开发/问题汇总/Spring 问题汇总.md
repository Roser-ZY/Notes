[TOC]

## *MyBatis*

### Q：`Invalid bound statement (not found)`

此错误为找不到 `XXXMapper.xml` 中的和目标语句对应的内容，通常有几种解决方式：

* 查看 `XXXMapper.xml` 中的 `namepsace` 字段是否与目标 `XXXMapper` 路径匹配。
* `application.properties` 中 `mybatis.mapper-locations` 配置项的路径错误，如果在单元测试中路径正确但扔报错，则可尝试修改路径不使用 `*` 等通配符。

## *JUnit5*

使用 *JUnit5* 时，需要在 *Gradle* 的配置文件内加入

```
test {
    useJUnitPlatform()
}
```

### Q：*JUnit5* 如何使用 *Spring* 依赖注入机制？

只需要在测试方法前使用注解 `@SpringBootTest` 即可。
