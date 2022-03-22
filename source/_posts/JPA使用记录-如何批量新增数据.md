---
title: JPA使用记录-如何批量新增数据
date: 2022-03-22 17:52:30
tags: springboot
---
# 概述
实现基本功能： 批量增加， 批量更新， insert 和 update 方法

# 代码
首先需要自己定义一个基础接口类
```java
// 告诉 spring 不要实例化这个对象
@NoRepositoryBean
public interface BaseJapRepository<T,ID> extends JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
    <S extends T> List<S> addAll(Iterable<S> entities);
    <S extends T> List<S> updateAll(Iterable<S> entities);
    <S extends T> S update(S entity);
    <S extends T> S insert(S entity);
}
```
然后自定义一个实现类
```java
package com.chancetop.atp.repositories;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.jpa.repository.support.JpaEntityInformation;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import java.util.ArrayList;
import java.util.List;

public class CustomerJpaRepositoryImpl<T, ID> extends SimpleJpaRepository<T, ID> implements BaseJapRepository<T, ID> {
    private static Logger logger = LoggerFactory.getLogger(CustomerJpaRepositoryImpl.class);

    private EntityManager entityManager;
    private JpaEntityInformation<T, ?> entityInformation;


    public CustomerJpaRepositoryImpl(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
        this.entityInformation = entityInformation;
    }

    public CustomerJpaRepositoryImpl(Class<T> domainClass, EntityManager em) {
        super(domainClass, em);
        this.entityManager = em;
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public <S extends T> List<S> addAll(Iterable<S> entities) {
        List<S> result = new ArrayList<>();
        entities.forEach(item -> {
            entityManager.persist(item);
            result.add(item);
        });
        entityManager.flush();
        entityManager.clear();
        return result;
    }

    @Transactional(rollbackFor = Exception.class)
    public <S extends T> List<S> updateAll(Iterable<S> entities) {
        List<S> result = new ArrayList<>();
        entities.forEach(item -> {
            this.entityManager.merge(item);
            result.add(item);
        });
        entityManager.flush();
        entityManager.clear();
        return result;
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public <S extends T> S update(S entity) {
        entityManager.merge(entity);  // merge 是 update
        return entity;
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public <S extends T> S insert(S entity) {
        entityManager.persist(entity); // persist 是插入
        return entity;
    }
}
```
我们要把这个实现类作为一个 `JPA` 的实现类, 只需要在启动类上面定义一下即可
```java 
@SpringBootApplication
// 定义 JPA 类
@EnableJpaRepositories(repositoryBaseClass= CustomerJpaRepositoryImpl.class)
public class AtpApplication {
	public static void main(String[] args) {
		SpringApplication.run(AtpApplication.class, args);
	}
}
```
# 测试
```java
package com.chancetop.atp.entites;

import com.chancetop.atp.repositories.ProjectRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.List;

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

    /**
     * 测试批量添加
     */
    @Test
    public void testAddAll(){
        List<Project> projects = new ArrayList<>();
        Project project = new Project();
        project.setProjectName("test-project");
        project.setProjectOwner("jackson");
        project.setProjectDesc("this is test project");

        Project project2 = new Project();
        project2.setProjectName("test-project-2");
        project2.setProjectOwner("jackson2");
        project2.setProjectDesc("this is test project2");
        projects.add(project);
        projects.add(project2);
        projectRepository.addAll(projects);

    }
    
    @Test
    public void testUpdateAll(){
        List<Project> projects = projectRepository.findAll();
        int i = 0;
        for (Project project : projects) {
            project.setProjectName("aaaaa" + i);
            i++;
        }
        projectRepository.updateAll(projects);
    }
}
```

# 总结
1. 首先还是要先定义数据库表, 然后再自己编写类， 应该设置 `ddl-auto: none`
2. 如果要修改数据库字段， 还是需要通过 `SQL` 修改比较安全
3. 最后就是复杂的语句， 需要通过 `@Query(<SQL>)` 来写。 

目前来看，比 `Mybatis` 更清晰。 待深入再了解了解。 

