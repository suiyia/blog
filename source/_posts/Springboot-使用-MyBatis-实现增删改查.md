title: Springboot 使用 MyBatis 实现增删改查
author: Answer
tags:
  - Springboot
  - MyBatis
categories:
  - Spring学习
date: 2019-08-10 17:39:00
toc: true
---
## 内容

Springboot 项目结合 Mybatis 实现 CRUD 增删改查

## 基本 Mysql 脚本
```
CREATE TABLE student (
	 sno VARCHAR(10) NOT NULL ,
    sname VARCHAR(10) NOT NULL ,
    ssex VARCHAR(2) NOT NULL 
);

INSERT INTO STUDENT VALUES ('001', 'KangKang', 'M ');
INSERT INTO STUDENT VALUES ('002', 'Mike', 'M ');
INSERT INTO STUDEN VALUES ('003', 'Jane', 'F ');

```

## pom 基本包依赖
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <version>2.1.5.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.1.0</version>
</dependency>

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid-spring-boot-starter</artifactId>
  <version>1.1.10</version>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.16</version>
</dependency>
```

## 配置文件 application.yml
```
server:
  servlet:
    context-path: /web
  port: 8080

spring:
  datasource:
    druid:
      # 数据库访问配置, 使用druid数据源
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/test?useunicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC
      username: root
      password: 123456
      # 连接池配置
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 连接等待超时时间
      max-wait: 30000
      # 配置检测可以关闭的空闲连接间隔时间
      time-between-eviction-runs-millis: 60000
      # 配置连接在池中的最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: select '1' from dual
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-open-prepared-statements: 20
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙
      filters: stat,wall
      # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
      aop-patterns: com.springboot.servie.*


      # WebStatFilter配置
      web-stat-filter:
        enabled: true
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤的格式
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      # StatViewServlet配置
      stat-view-servlet:
        enabled: true
        # 访问路径为/druid时，跳转到StatViewServlet
        url-pattern: /druid/*
        # 是否能够重置数据
        reset-enable: false
        # 需要账号密码才能访问控制台
        login-username: druid
        login-password: druid123
        # IP白名单
        # allow: 127.0.0.1
        #　IP黑名单（共同存在时，deny优先于allow）
        # deny: 192.168.1.218

      # 配置StatFilter
      filter:
        stat:
          log-slow-sql: true

mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration-properties: BEFORE

```

### 代码实现

- 接口+注解形式实现数据库读取写入
```
@Mapper
public interface StudentManagerWithInject {

    @Insert("insert into student(sno,sname,ssex)values(#{sno},#{name},#{ssex})")
    int add(Student student);

    @Update("update student set sname=#{name},ssex=#{sex} where sno=#{sno}")
    int update(Student student);

    @Delete("delete from student where sno=#{sno}")
    int deleteBySno(String sno);

    @Select("select * from student where sno=#{sno}")
    @Results(id = "student",value= {
        @Result(property = "sno", column = "sno", javaType = String.class),
        @Result(property = "name", column = "sname", javaType = String.class),
        @Result(property = "sex", column = "ssex", javaType = String.class)
    })
    Student queryStudentBySno(String sno);
}
```
- 自动装配实现业务逻辑
```
@RestController
public class TestController {


    @Autowired
    private StudentManagerWithInject studentManagerWithInject;

    @Autowired
    private StudentMapperWithXML studentMapperWithXML;

    @RequestMapping(value = "/querystudent",method = RequestMethod.GET)
    public Student queryStudentBySno(
        @RequestParam String sno){
        return studentManagerWithInject.queryStudentBySno(sno);
    }

    @RequestMapping(value = "/querystudentwithxml",method = RequestMethod.GET)
    public Student queryStudentWithXMLBySno(
        @RequestParam String sno){
        return studentMapperWithXML.queryStudentBySno(sno);
    }
}
```

- 调用接口，得到结果
```
http://localhost:8080/web/querystudent?sno=001

输出
{
  "sno": "001",
  "name": "KangKang",
  "sex": "M "
}
```


## 最后

- 一般项目会增加一个 service 服务层，一个服务可能对应多个数据库操作，这里节省时间就没有加，工作中还是根据项目规范来。

- [学习参考](https://github.com/wuyouzhuguli/SpringAll)

- [源码第一章节](https://github.com/suiyia/JavaReposity/tree/master/springboot4all)