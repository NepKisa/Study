# 1、Gitee下载代码生成器

人人开源代码生成器：https://gitee.com/renrenio/renren-generator

![image-20230307115231868](C:\Users\hakuou\Documents\program\Git\images\image-20230307115231868.png)

# 2、解压后将工程导入为maven项目

![image-20230307131548802](../../../images/image-20230307131548802.png)

![image-20230307131942552](../../../images/image-20230307131942552.png)

# 3、修改配置

## *application.yml*

![image-20230307131840819](../../../images/image-20230307131840819.png)

## *generator.properties*

![image-20230307132945872](../../../images/image-20230307132945872.png)

## *修改生成模板*

注释掉此注解，权限管理的

![image-20230307163711418](../../../images/image-20230307163711418.png)

![image-20230307162203516](../../../images/image-20230307162203516.png)

# 4、准备数据库

## *数据库需先建表*

![image-20230307135046191](../../../images/image-20230307135046191.png)

# 5、启动服务，生成代码

![image-20230307135138326](../../../images/image-20230307135138326.png)

![image-20230307135239323](../../../images/image-20230307135239323.png)

## *浏览器访问：http://localhost:9999/*

![image-20230307135344667](../../../images/image-20230307135344667.png)

### *选择需要生成代码的表*

![image-20230307140229553](../../../images/image-20230307140229553.png)

## **解压后将生成的代码复制到工程下**

![image-20230307140316138](../../../images/image-20230307140316138.png)

# 6、==Pom.xml==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.nepkisa</groupId>
    <artifactId>test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>test</name>
    <description>test</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.3.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

