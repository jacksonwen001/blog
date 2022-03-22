---
title: JPA使用记录-配置
date: 2022-03-22 17:45:17
tags: springboot
---
# 概述
记录如何使用 JPA

# build.gradle 配置 
```groovy
plugins {
	id 'org.springframework.boot' version '2.6.4'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'mysql:mysql-connector-java'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```
# 项目配置
首先将项目配置为多个不同环境

1. `application.yaml` 配置
```yaml
spring:
  profiles:
    active: dev
```

2. `application-dev.yaml` 配置
```yaml
server:
  port: 8099
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/atp?useUnicode=true&characterEncoding=utf-8
    username: root
    password: qwe123
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      minimum-idle: 5 # 最小空闲时间
      idle-timeout: 600000 # 空闲连接存活最大时间，默认是 60 00 00 ，也就是 10 分钟
      maximum-pool-size: 10 #连接池最大连接数 默认是 10
      auto-commit: true # 自动提交
      max-lifetime: 1800000 # 连接池连接的最大生命周期
      connection-timeout: 30000 # 连接查实的时间 默认 30 秒
  jpa:
    hibernate:
      ddl-auto: update
      connection:
        provider_class: com.zaxxer.hikari.hibernate.HikariConnectionProvider
    show-sql: true
    database: mysql
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
```
# 代码
新建一个实体类， 这个类会自动创建表
```java
import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.UpdateTimestamp;

import javax.persistence.*;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;
import java.time.LocalDateTime;

/**
 * CREATE TABLE IF NOT EXISTS `t_project`(
 *     `project_id` bigint auto_increment primary key comment '主键',
 *     `project_name` varchar(50) not null comment '项目名称',
 *     `project_owner` varchar(50) not null comment '项目owner',
 *     `project_desc` varchar(250) comment '项目描述',
 *     `created_time` datetime not null comment '创建时间',
 *     `updated_time` datetime not null comment '更新时间'
 * )ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
 */

@Table(name = "t_project")
@Getter
@Setter
@Entity
public class Project {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    @Column(name = "project_id",  length = 32)
    private String projectId;

    @Column(name = "project_name", columnDefinition = "VARCHAR(50) NOT NULL COMMENT '项目名称'")
    @NotBlank(message = "project name can't be null! ")
    @NotEmpty(message = "project name can't be null! ")
    public String projectName;

    @Column(name = "project_owner", columnDefinition = "varchar(50) not null comment '项目owner'")
    @NotBlank(message = "project owner can't be null")
    @NotEmpty(message = "project owner can't be null! ")
    public String projectOwner;

    @Column(name = "project_desc", columnDefinition = "varchar(250) comment '项目描述'")
    public String projectDesc;

    /**
    * 自动添加创建时间
    */
    @Column(name = "created_time")
    @CreationTimestamp
    public LocalDateTime createdAt;

    /**
    * 自动添加更新时间
    */
    @Column(name = "updated_time")
    @UpdateTimestamp
    public LocalDateTime updatedAt;
}
```

然后创建一个 `repository` 类来执行 `sql` 操作
```java
import com.chancetop.atp.entites.Project;
import org.springframework.data.repository.CrudRepository;

public interface ProjectRepository extends CrudRepository<Project, String>{
}
```

最后我们写一个测试类， 来测试下是否正常
```java
@SpringBootTest
public class ProjectTest {
    private ProjectRepository projectRepository;

    @Autowired
    public ProjectTest(ProjectRepository projectRepository){
        this.projectRepository = projectRepository;
    }

    @Test
    public void testSave(){
        Project project = new Project();
        project.setProjectName("test-project");
        project.setProjectOwner("jackson");
        project.setProjectDesc("this is test project");
        projectRepository.save(project);
    }
}
```