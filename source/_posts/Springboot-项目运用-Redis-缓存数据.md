title: Springboot 项目运用 Redis 缓存数据
author: Answer
tags:
  - Springboot
  - Redis
categories:
  - Spring学习
date: 2019-08-10 17:40:00
toc: true
---
# 介绍
Springboot 项目与 Redis 结合获取数据

# pom 依赖

主要是 mybatis、redis 的相关依赖

```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>2.1.5.RELEASE</version>
    </dependency>
    
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.8</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.16</version>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.10</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.2</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
      <version>2.1.3.RELEASE</version>
    </dependency>
```

# 配置文件
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

  redis:
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: localhost
    # Redis服务器连接端口
    port: 6379

logging:
  level:
    com.example.demo.dao: debug
```

# Redis 配置类
```
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    // 自定义缓存key生成策略
    @Override
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, java.lang.reflect.Method method, Object... params) {
                StringBuffer sb = new StringBuffer();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    // 缓存管理器
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheManager cacheManager = RedisCacheManager.create(connectionFactory);
        return cacheManager;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        setSerializer(template);// 设置序列化工具
        template.afterPropertiesSet();
        return template;
    }

    private void setSerializer(StringRedisTemplate template) {
        @SuppressWarnings({ "rawtypes", "unchecked" })
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
    }
}
```


# dao 层以及实现
使用 MyBatis 注解方式实现数据访问
```
@Mapper
public interface StudentManager {

    @Update("update student set sname=#{sname},ssex=#{ssex} where sno=#{sno}")
    int update(Student student);

    @Delete("delete from student where sno=#{sno}")
    void deleteStudentBySno(String sno);

    @Select("select * from student where sno=#{sno}")
    @Results(id = "student", value = { @Result(property = "sno", column = "sno", javaType = String.class),
        @Result(property = "sname", column = "sname", javaType = String.class),
        @Result(property = "ssex", column = "ssex", javaType = String.class) })
    Student queryStudentBySno(String sno);
}
```

# service 层及实现
queryStudentBySno 方法使用了注解 @Cacheable(key="#p0")，即将 id 作为 redis 中的 key 值
- @CacheConfig(cacheNames = "student") 一个类中可能会有多个缓存操作，而这些缓存操作可能是重复的
- @Cacheable 是用来声明方法是可缓存的
- @CachePut 主要用于数据新增和修改操作上
- @CacheEvict 通常用在删除方法上，用来从缓存中移除相应数据
```
@CacheConfig(cacheNames = "student")
public interface StudentService {

    @CachePut(key = "#p0.sno")
    Student update(Student student);

    @CacheEvict(key = "#p0", allEntries = true)
    void deleteStudentBySno(String sno);

    @Cacheable(key = "#p0")
    Student queryStudentBySno(String sno);

}

@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentManager studentManager;

    @Override
    public Student update(Student student) {
        this.studentManager.update(student);
        return this.studentManager.queryStudentBySno(student.getSno());
    }

    @Override
    public void deleteStudentBySno(String sno) {
        this.studentManager.deleteStudentBySno(sno);
    }

    @Override
    public Student queryStudentBySno(String sno) {
        return this.studentManager.queryStudentBySno(sno);
    }
}

```

# Test 用例
第一次从数据库中读取，第二次直接从缓存读取

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

    @Autowired
    private StudentService studentService;

    @Test
    public void contextLoads() {
        System.out.println(studentService.queryStudentBySno("001").toString());
        System.out.println(studentService.queryStudentBySno("001").toString());
    }

}


2019-08-10 17:27:32.739 DEBUG 7140 --- [           main] c.e.d.d.S.queryStudentBySno              : ==>  Preparing: select * from student where sno=? 
2019-08-10 17:27:33.053 DEBUG 7140 --- [           main] c.e.d.d.S.queryStudentBySno              : ==> Parameters: 001(String)
2019-08-10 17:27:33.108 DEBUG 7140 --- [           main] c.e.d.d.S.queryStudentBySno              : <==      Total: 1
Student(sno=001, sname=KangKang, ssex=M )
Student(sno=001, sname=KangKang, ssex=M )
```

# 总结
