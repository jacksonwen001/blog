---
title: JPA使用记录-开发流程
date: 2022-03-23 10:33:27
tags: springboot
---
# 概述
使用流程： 
1. 执行 `SQL` 脚本
2. 根据 `SQL` 脚本编写 `Entity`
3. 执行测试

# 编写 SQL 脚本
假设我们要创建一个表， 最好的方式是不要通过代码直接创建表，首先不安全，其次容易把数据都删了， 最安全的操作还是数据库操作一次， 代码修改一次。 
创建一个表:
```sql
CREATE TABLE IF NOT EXISTS `t_user` (
    `user_id`              varchar(32) COLLATE utf8mb4_general_ci NOT NULL COMMENT 'User ID',
    `user_name`            varchar(64) NOT NULL COMMENT 'User name',
    `email`                varchar(64) NOT NULL COMMENT 'E-Mail address',
    `password`             varchar(256) COLLATE utf8mb4_bin DEFAULT NULL COMMENT 'password',
    `status`               INT DEFAULT 0 COMMENT 'User status: 0-inactive; 1-active',
    `created_time` datetime DEFAULT NULL COMMENT 'created time',
    `updated_time` datetime DEFAULT NULL COMMENT 'updated time',
    PRIMARY KEY (`user_id`),
    UNIQUE KEY `t_user_unique_key` (`user_name`, `email`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE utf8mb4_general_ci;
```

# 编写相应的 Entity
```java
public class UserEntity {
    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    @Column(name = "user_id", length = 32, nullable = false)
    private String id;

    @Column(name = "user_name", columnDefinition = "VARCHAR(64) NOT NULL COMMENT 'user name'")
    private String username;

    @Column(name = "email", columnDefinition = "VARCHAR(64) NOT NULL COMMENT 'email'")
    private String email;

    @Column(name = "password", columnDefinition = "VARCHAR(256) COLLATE utf8mb4_bin DEFAULT NULL comment 'password'")
    private String password;

    @Column(name = "status", columnDefinition = "INT NOT NULL DEFAULT 0 COMMENT '0:inactive 1:active'")
    private Integer status;

    @Column(name = "created_time")
    @CreationTimestamp
    private LocalDateTime createdAt;

    @Column(name = "updated_time")
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

# 测试
```java
@SpringBootTest
public class UserEntityTest {
    UserRepository userRepository;
    @Autowired
    public UserEntityTest(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    @Test
    public void test(){
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername("Jackson");
        userEntity.setEmail("Jackson@email.com");
        userEntity.setStatus(0);
        userEntity.setPassword("password");
        userRepository.save(userEntity);
    }
}
```
# 总结
麻烦了一点，但是对于数据还是需要慎重一些
