## 基于 Vue+SpringBoot 的CRUD

### 截图

![image-20200229180140232](Vue+SpringBoot---CRUD.assets/image-20200229180140232.png)

### Vue

#### 环境搭建

```shell
# 安装 Vue 和 Vue-cli
npm i -g vue
npm i -g @vue/cli@3.7.0

# 创建项目
vue ui

# 安装依赖
npm i axios
npm i element-ui
```



### SpringBoot

#### 环境搭建

##### pom.xml

```xml
<dependencies>
  <!-- Web -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <!-- Mybatis -->
  <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
  </dependency>
  <!-- Mysql 驱动包 -->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <exclusions>
      <exclusion>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```

#### 整合 lombok 日志

- pom.xml

  ```xml
  <!-- lombok 日志 -->
  <dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  </dependency>
  ```

- logback.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>
      <!-- 尽量别用绝对路径，如果带参数不同容器路径解释可能不同,以下配置参数在pom.xml里 -->
      <property name="log.root.level" value="INFO"/> <!-- 日志级别 -->
      <property name="log.other.level" value="INFO"/> <!-- 其他日志级别 -->
      <property name="log.base"
                value="logs"/> <!-- 日志路径，这里是相对路径，web项目eclipse下会输出到eclipse的安装目录下，如果部署到linux上的tomcat下，会输出到tomcat/bin目录 下 -->
      <property name="log.moduleName" value="blog"/>  <!-- 模块名称， 影响日志配置名，日志文件名 -->
      <property name="log.max.size" value="20MB"/> <!-- 日志文件大小 -->
  
      <!--控制台输出 -->
      <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
          <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
              <pattern>%red(%d{yyyy-MM-dd HH:mm:ss}) %green([%thread]) %highlight(%-5level) %boldMagenta(%logger.%method:%L) - %cyan(%msg%n)</pattern>
          </encoder>
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>debug</level>
          </filter>
      </appender>
  
      <!-- info文件输出 -->
      <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <File>${log.base}/${log.moduleName}-info.log
          </File><!-- 设置日志不超过${log.max.size}时的保存路径，注意如果 是web项目会保存到Tomcat的bin目录 下 -->
          <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <FileNamePattern>${log.base}/archive/${log.moduleName}-info-%d{yyyy-MM-dd}.%i.log
              </FileNamePattern>
              <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>${log.max.size}</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
          </rollingPolicy>
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>INFO</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <!-- 日志输出的文件的格式  -->
          <layout class="ch.qos.logback.classic.PatternLayout">
              <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method:%L -%msg%n</pattern>
          </layout>
      </appender>
  
      <!-- debug文件输出 -->
      <appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <File>${log.base}/${log.moduleName}-debug.log
          </File><!-- 设置日志不超过${log.max.size}时的保存路径，注意如果 是web项目会保存到Tomcat的bin目录 下 -->
          <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <FileNamePattern>${log.base}/archive/${log.moduleName}-debug-%d{yyyy-MM-dd}.%i.log
              </FileNamePattern>
              <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>${log.max.size}</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
          </rollingPolicy>
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>DEBUG</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <!-- 日志输出的文件的格式  -->
          <layout class="ch.qos.logback.classic.PatternLayout">
              <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method:%L -%msg%n</pattern>
          </layout>
      </appender>
  
      <!-- warning文件输出 -->
      <appender name="warn" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <File>${log.base}/${log.moduleName}-warn.log
          </File><!-- 设置日志不超过${log.max.size}时的保存路径，注意如果 是web项目会保存到Tomcat的bin目录 下 -->
          <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <FileNamePattern>${log.base}/archive/${log.moduleName}-warn-%d{yyyy-MM-dd}.%i.log
              </FileNamePattern>
              <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>${log.max.size}</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
          </rollingPolicy>
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>WARN</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <!-- 日志输出的文件的格式  -->
          <layout class="ch.qos.logback.classic.PatternLayout">
              <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method:%L -%msg%n</pattern>
          </layout>
      </appender>
  
      <!-- error文件输出 -->
      <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <File>${log.base}/${log.moduleName}-error.log
          </File><!-- 设置日志不超过${log.max.size}时的保存路径，注意如果 是web项目会保存到Tomcat的bin目录 下 -->
          <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <FileNamePattern>${log.base}/archive/${log.moduleName}-error-%d{yyyy-MM-dd}.%i.log
              </FileNamePattern>
              <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>${log.max.size}</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
          </rollingPolicy>
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>ERROR</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <!-- 日志输出的文件的格式  -->
          <layout class="ch.qos.logback.classic.PatternLayout">
              <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method:%L -%msg%n</pattern>
          </layout>
      </appender>
  
      <!-- error文件输出 -->
      <appender name="all" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <File>${log.base}/${log.moduleName}-all.log
          </File><!-- 设置日志不超过${log.max.size}时的保存路径，注意如果 是web项目会保存到Tomcat的bin目录 下 -->
          <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <FileNamePattern>${log.base}/archive/${log.moduleName}-all-%d{yyyy-MM-dd}.%i.log
              </FileNamePattern>
              <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>${log.max.size}</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
          </rollingPolicy>
          <!-- 日志输出的文件的格式  -->
          <layout class="ch.qos.logback.classic.PatternLayout">
              <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method:%L -%msg%n</pattern>
          </layout>
      </appender>
  
      <!-- 为某个包下的所有类的指定Appender 这里也可以指定类名称例如：com.aa.bb.ClassName -->
      <logger name="com.cloudfly.blog" additivity="false">
          <level value="debug"/>
          <appender-ref ref="stdout"/>
          <appender-ref ref="info"/>
          <appender-ref ref="debug"/>
          <appender-ref ref="warn"/>
          <appender-ref ref="error"/>
          <appender-ref ref="all"/>
      </logger>
      <!-- root将级别为“DEBUG”及大于“DEBUG”的日志信息交给已经配置好的名为“Console”的appender处理，“Console”appender将信息打印到Console -->
      <!-- 设置日志级别 -->
      <root level="info">
          <appender-ref ref="stdout"/> <!-- 标识这个appender将会添加到这个logger -->
          <appender-ref ref="info"/>
          <appender-ref ref="debug"/>
          <appender-ref ref="warn"/>
          <appender-ref ref="error"/>
          <appender-ref ref="all"/>
      </root>
  </configuration>
  ```

#### 整合 PageHelper 分页

- pom.xml

  ```xml
  <!-- pagehelper分页插件 -->
  <dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.5</version>
  </dependency>
  ```

- UserController 类

  ```java
  @RequestMapping(value = "/page", method = RequestMethod.POST)
  public Result getByPage(@RequestBody UserVo userVo, @RequestParam("pageNum") Integer pageNum, @RequestParam("pageSize") Integer pageSize) {
    //PageHelper
    PageHelper.startPage(pageNum, pageSize);
    List<User> userList = userService.getAll(userVo);
    PageInfo<User> userPageInfo = new PageInfo<>(userList, 2);
    return new Result(userPageInfo);
  }
  ```