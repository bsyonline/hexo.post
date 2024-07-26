---
title: spring-data-elasticsearch 示例说明
date: 2016-07-15 22:25:09
tags:
 - Spring
 - Elastic
category: 
 - Spring
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---
### POM依赖
```
<properties>
    <java-version>1.7</java-version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring.version>4.1.2.RELEASE</spring.version>
    <elasticsearch.version>1.4.4</elasticsearch.version>
    <spring-data-elasticsearch.version>1.3.4.RELEASE</spring-data-elasticsearch.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>${elasticsearch.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-elasticsearch</artifactId>
        <version>${spring-data-elasticsearch.version}</version>
    </dependency>
</dependencies>
```

### spring 配置文件

1. beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">
    <context:annotation-config />
    <!-- 自动扫描所有注解该路径 -->
    <!-- <context:component-scan base-package="com.sf.heros.mq.*" /> -->
    <context:property-placeholder location="classpath:/config.properties" />

    <import resource="elasticsearch.xml" />
</beans>
```

2. elasticsearch.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xsi:schemaLocation="http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

    <elasticsearch:transport-client id="client" cluster-name="daas" cluster-nodes="192.168.11.11:9300"/>

    <bean name="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"/>
    </bean>
    <bean name="entQueryService" class="com.rolex.se.sample.service.EntQueryService">
        <property name="repository" ref="entQueryRepository"/>
        <property name="elasticsearchTemplate" ref="elasticsearchTemplate"/>
    </bean>

    <elasticsearch:repositories base-package="com.rolex.se.sample.dao"/>
</beans>
```
### 类
1. 实体类
```java
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.*;

import java.util.*;

@Document(indexName = "pen",type = "pen" , shards = 1, replicas = 0, indexStoreType = "memory", refreshInterval = "-1")
public class Pen {

    @Id
    private String id;
    private String name;
    private Long price;
    @Field(type = FieldType.String, index = FieldIndex.not_analyzed)
	private Set<String> type = new HashSet<String>();

    public Set<String> getType() {
        return type;
    }

    public void setType(Set<String> type) {
        this.type = type;
    }

    public Pen(){}

    public Pen(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Long getPrice() {
        return price;
    }

    public void setPrice(Long price) {
        this.price = price;
    }

}
```

2. 查询类
```java
import org.springframework.data.elasticsearch.entities.Pen;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

public interface PenRepository extends ElasticsearchRepository<Pen,String> {
}
```
3. 测试类
```java
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.data.elasticsearch.entities.Pen;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import java.util.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/springContext-test.xml")
public class PenRepositoryTest {

    @Resource
    private PenRepository penRepository;

    @Before
    public void emptyData(){
        penRepository.deleteAll();
    }

    @Test
    public void shouldIndexSingleBookEntity(){

        Pen pen = new Pen();
        pen.setId("123455");
        pen.setName("LIMY");
        Set<String> types = new HashSet<String>();
        types.add("E_PRI_PERSON.NAME");
        types.add("E_INV_INVESTMENT.SUBCONAM");
        pen.setType(types);
        //Indexing using sampleArticleRepository
        penRepository.save(pen);
        //lets try to search same record in elasticsearch
        Pen pen2 = penRepository.findOne(pen.getId());

        BoolQueryBuilder query = QueryBuilders.boolQuery();
            query.must(QueryBuilders.queryString("E_PRI_PERSON.NAME").field("pen.type"))
            .must(QueryBuilders.queryString("E_INV_INVESTMENT.SUBCONAM").field("pen.type"));



        Iterable<Pen> res = penRepository.search(query);

        Iterator<Pen> it = res.iterator();

        while (it.hasNext()){
            System.out.println(it.next().getType());
        }

    }
}
```
