title: Springboot 与 AOP 实现
author: Answer
tags:
  - Springboot
  - AOP
categories:
  - Spring学习
date: 2019-08-10 17:39:00
toc: true
---
# 介绍

Springboot 项目 AOP 实现，记录方法的执行过程

# pom 依赖

druid、JdbcTemplate、aop

```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
      <version>2.1.1.RELEASE</version>
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
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.8</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
```

# 代码实现

- Mysql 脚本
```
CREATE TABLE SYS_LOG (
   ID INT(20) NOT NULL COMMENT '用户名' PRIMARY KEY AUTO_INCREMENT ,
   USERNAME VARCHAR(50) NULL COMMENT '用户名',
   OPERATION VARCHAR(50) NULL COMMENT '用户操作',
   TIME INT(11) NULL COMMENT '响应时间',
   METHOD VARCHAR(1024) NULL COMMENT '请求方法',
   PARAMS VARCHAR(1024) NULL COMMENT '请求参数',
   IP VARCHAR(64) NULL COMMENT 'IP地址',
   CREATE_TIME DATE NULL COMMENT '创建时间'
);

```
- 实体类
```
import java.io.Serializable;
import java.util.Date;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Setter
@Getter
@ToString
public class SysLog implements Serializable {

    private Integer id;
    private String username;
    private String operation;
    private Integer time;
    private String method;
    private String params;
    private String ip;
    private Date createTime;

}
```

- 定义一个方法级别的 @Log 注解，需要监控的方法加上该注解就行

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}

```

- 定义 dao 方法，保存日志到数据库

```
public interface SysLogDao {
    void saveSysLog(SysLog sysLog);
}
```
```
@Component
public class SysLogDaoImpl implements SysLogDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void saveSysLog(SysLog sysLog) {
        StringBuffer sql = new StringBuffer("insert into sys_log ");
        sql.append("(username,operation,time,method,params,ip,create_time) ");
        sql.append("values(:username,:operation,:time,:method,");
        sql.append(":params,:ip,:createTime)");

        NamedParameterJdbcTemplate npjt = new NamedParameterJdbcTemplate(this.jdbcTemplate.getDataSource());
        npjt.update(sql.toString(), new BeanPropertySqlParameterSource(sysLog));
    }
}
```

- 切面切点定义
```
@Aspect
@Component
public class LogAspect {

    @Autowired
    private SysLogDao sysLogDao;

    @Pointcut("@annotation(com.example.demo.annotion.Log)")
    public void pointcut(){}

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint){
        Object result = null;
        long beginTime = System.currentTimeMillis();
        try {
            // 执行方法
            result = proceedingJoinPoint.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        // 执行时长(毫秒)
        long time = System.currentTimeMillis() - beginTime;
        // 保存日志
        saveLog(proceedingJoinPoint, time);
        return result;
    }

    private void saveLog(ProceedingJoinPoint joinPoint, long time) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        SysLog sysLog = new SysLog();
        Log logAnnotation = method.getAnnotation(Log.class);
        if (logAnnotation != null) {
            // 注解上的描述
            sysLog.setOperation(logAnnotation.value());
        }
        // 请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        sysLog.setMethod(className + "." + methodName + "()");
        // 请求的方法参数值
        Object[] args = joinPoint.getArgs();
        // 请求的方法参数名称
        LocalVariableTableParameterNameDiscoverer u = new LocalVariableTableParameterNameDiscoverer();
        String[] paramNames = u.getParameterNames(method);
        if (args != null && paramNames != null) {
            String params = "";
            for (int i = 0; i < args.length; i++) {
                params += "  " + paramNames[i] + ": " + args[i];
            }
            sysLog.setParams(params);
        }
        // 获取request
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        // 设置IP地址
        sysLog.setIp(IPUtils.getIpAddr(request));
        // 模拟一个用户名
        sysLog.setUsername("mrbird");
        sysLog.setTime((int) time);
        sysLog.setCreateTime(new Date());
        // 保存系统日志
        sysLogDao.saveSysLog(sysLog);
    }
}


public class IPUtils {

    /**
     * 获取IP地址
     *
     * 使用Nginx等反向代理软件， 则不能通过request.getRemoteAddr()获取IP地址 如果使用了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP地址，X-Forwarded-For中第一个非unknown的有效IP字符串，则为真实IP地址
     */
    public static String getIpAddr(HttpServletRequest request) {

        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return "0:0:0:0:0:0:0:1".equals(ip) ? "127.0.0.1" : ip;
    }

}
```

- Controller 方法
```
@RestController
public class TestController {

    @Log("执行方法一")
    @GetMapping("/one")
    public void methodOne(String name) { }

    @Log("执行方法二")
    @GetMapping("/two")
    public void methodTwo() throws InterruptedException {
        Thread.sleep(2000);
    }

    @Log("执行方法三")
    @GetMapping("/three")
    public void methodThree(String name, String age) { }

}
```


# 测试

```
http://localhost:8080/web/one?name=KangKang

http://localhost:8080/web/two

http://localhost:8080/web/three?name=Mike&age=25

> select * from sys_log order by id;
ID	USERNAME	OPERATION	TIME	METHOD	PARAMS	IP	CREATE_TIME
1	mrbird	执行方法一	3	com.example.demo.web.TestController.methodOne()	  name: KangKang	127.0.0.1	2019-08-10
2	mrbird	执行方法二	2002	com.example.demo.web.TestController.methodTwo()		127.0.0.1	2019-08-10
3	mrbird	执行方法三	0	com.example.demo.web.TestController.methodThree()	  name: Mike  age: 25	127.0.0.1	2019-08-10
4	mrbird	执行方法三	0	com.example.demo.web.TestController.methodThree()	  name: Mike  age: 25	127.0.0.1	2019-08-10
5	mrbird	执行方法二	2001	com.example.demo.web.TestController.methodTwo()		127.0.0.1	2019-08-10

```

# 总结
- 切面对项目响应时间的影响待测试，现在用的项目基本没有应用到切面，等有实际落地需求在改进学习下
- [源码](https://github.com/suiyia/JavaReposity/tree/master/springboot4all)