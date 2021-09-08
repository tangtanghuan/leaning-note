# Maven

## Maven打包跳过测试

### 方法一：修改pom.xml文件

```
<project>  
  <build>  
    <plugins>  
      <plugin>  
        <groupId>org.apache.maven.plugins</groupId>  
        <artifactId>maven-surefire-plugin</artifactId>  
        <version>2.18.1</version>  
        <configuration>  
          <skipTests>true</skipTests>  
        </configuration>  
      </plugin>  
    </plugins>  
  </build>  
</project> 
```

### 方法二：在Terminal执行命令

```
mvn install -DskipTests
```



### 方法三：在Terminal执行命令

```
mvn install -Dmaven.test.skip=true
```



### 方法四：Spring boot项目使用

spring-boot-maven-plugin插件已经集成了maven-surefire-plugin插件，会自动运行 junit test，只需要在pom.xml里增加如下配置：

```
<properties>
 	<!-- 跳过测试 -->
    <skipTests>true</skipTests>
</properties>
```

