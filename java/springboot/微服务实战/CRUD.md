# CRUD

## 环境

| software   | version |
| ---------- | ------- |
| java       | 8       |
| maven      | 3.8.5   |
| springboot | 2.7.9   |

## 1、新建工程

## 2、引入依赖

```xml
        <!--Mybatis-Plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.3.1</version>
        </dependency>
        <!--druid数据库连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.17</version>
        </dependency>
        <!--MySQL驱动-->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
```

## 3、配置文件

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

## 4、数据库建表

```sql
create table student(
    id bigint(20) primary key ,
    name varchar(50)  ,
    address varchar(50)
)
```

## 5、Mapper

```java
@Mapper
public interface StudentMapper extends BaseMapper<Student> {

}
```

## 6、Service

```java
public interface StudentService extends IService<Student> {

}

================================================
实现类
@Service
public class StudentServiceImpl extends ServiceImpl<StudentMapper, Student> implements StudentService {

}
```

## 7、Controller

```java
@RestController
@RequestMapping("/student")
public class StudentController {

    @Autowired
    StudentService studentService;

    @RequestMapping("/list")
    public List list() {
        return studentService.list();
    }

    @RequestMapping("/add")
    public String add(Student student) {
        studentService.save(student);
        System.out.println(student);
        return "success";
    }

    @RequestMapping("/update")
    public String update(Student student) {
        studentService.updateById(student);
        return "success";
    }

    @RequestMapping("/delete")
    public String delete(Long id) {
        studentService.removeById(id);
        return "success";
    }
}
```

## 8、entity

```java
@ToString
@Data
public class Student {
    private Long id;
    private String name;
    private String address;
}
```

## 9、分页插件

```java
@Configuration
@MapperScan("com.neptune.springboot02webadmin.mapper")
public class MybatisPlusConfig {

    /**
     * MybatisPlusInterceptor
     *
     * @return
     */
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join

        //这是分页拦截器
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setOverflow(true);
        paginationInnerInterceptor.setMaxLimit(500L);
        mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);

        return mybatisPlusInterceptor;

    }
}
```

