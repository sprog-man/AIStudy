# AI学习交流平台 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 开发一个AI学习交流平台，用户可以发帖分享AI知识，设置价格，其他用户使用学习币购买，包含学习活跃排行榜系统。

**Architecture:** 前后端分离架构，后端使用Spring Boot提供RESTful API，前端使用Vue3+Element Plus构建单页应用，使用MySQL作为数据库，Redis用于缓存，JWT实现认证，支持读写分离。

**Tech Stack:** 
- Backend: Spring Boot 3.5.x, Spring Security, MyBatis Plus, JWT, MySQL 8.0, Redis
- Frontend: Vue 3, Vue Router, Pinia, Element Plus, Axios
- Cache: Redis (用于缓存预热)
- Database: MySQL 8.0 (主从读写分离)
- Reverse Proxy: Nginx (D:\nginx-1.30.0)
- Third-party: GitHub, WeChat OAuth integration

---

### Task 1: 创建项目基础结构

**Files:**
- Create: `backend/pom.xml`
- Create: `backend/src/main/java/com/aistudy/AIStudyApplication.java`
- Create: `backend/src/main/resources/application.yml`
- Create: `frontend/package.json`
- Create: `frontend/src/App.vue`
- Create: `frontend/src/main.js`

- [ ] **Step 1: 创建后端Maven项目结构**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.5</version>
        <relativePath/>
    </parent>

    <groupId>com.aistudy</groupId>
    <artifactId>backend</artifactId>
    <version>1.0.0</version>
    <name>AIStudy Backend</name>
    <description>AI学习交流平台后端服务</description>

    <properties>
        <java.version>17</java.version>
        <mybatis-plus.version>3.5.7</mybatis-plus.version>
        <jwt.version>0.12.6</jwt.version>
        <knife4j.version>4.5.0</knife4j.version>
        <mysql.version>8.0.33</mysql.version>
        <redis.version>3.3.1</redis.version>
        <spring-boot.version>3.5.5</spring-boot.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- Spring Boot Data Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Spring Boot Cache -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>

        <!-- MyBatis Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.4.1</version>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jwt.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- Knife4j API文档 -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
            <version>${knife4j.version}</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

- [ ] **Step 2: 创建后端主启动类**

```java
package com.aistudy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AIStudyApplication {
    public static void main(String[] args) {
        SpringApplication.run(AIStudyApplication.class, args);
    }
}
```

- [ ] **Step 3: 创建后端配置文件**

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/aistudy?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    host: localhost
    port: 6379
    database: 0
  cache:
    type: redis
    redis:
      time-to-live: 600000
      cache-null-values: false

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0

jwt:
  secret: aistudy-jwt-secret-key-2026
  expire: 86400000  # 24小时

file:
  upload-path: D:/my-ai-project/AIStudy/uploads/
```

- [ ] **Step 4: 创建前端package.json**

```json
{
  "name": "aistudy-frontend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.3.4",
    "vue-router": "^4.2.4",
    "pinia": "^2.1.6",
    "element-plus": "^2.3.9",
    "axios": "^1.4.0",
    "@element-plus/icons-vue": "^2.1.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.2.3",
    "vite": "^4.4.5"
  }
}
```

- [ ] **Step 5: 创建前端主入口文件**

```javascript
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
import router from './router'
import App from './App.vue'

const app = createApp(App)

// 注册所有图标
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}

app.use(ElementPlus)
app.use(router)
app.mount('#app')
```

- [ ] **Step 6: 创建根组件**

```vue
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script setup>
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB',
    'Microsoft YaHei', Arial, sans-serif;
  background-color: #f5f5f5;
}

#app {
  height: 100vh;
}
</style>
```

- [ ] **Step 7: 验证项目结构**

创建目录结构：
```
AIStudy/
├── backend/
│   ├── pom.xml
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/
│           │       └── aistudy/
│           │           └── AIStudyApplication.java
│           └── resources/
│               └── application.yml
└── frontend/
    ├── package.json
    └── src/
        ├── main.js
        └── App.vue
```

运行 `cd backend && mvn clean install` 和 `cd frontend && npm install` 验证项目能够正常构建。

- [ ] **Step 8: Commit**

```bash
git add .
git commit -m "feat: 创建项目基础架构"
```

### Task 2: 数据库设计和实体类

**Files:**
- Create: `backend/src/main/resources/schema.sql`
- Create: `backend/src/main/java/com/aistudy/entity/User.java`
- Create: `backend/src/main/java/com/aistudy/entity/Post.java`
- Create: `backend/src/main/java/com/aistudy/entity/Comment.java`
- Create: `backend/src/main/java/com/aistudy/entity/CoinTransaction.java`
- Create: `backend/src/main/java/com/aistudy/entity/DailySignIn.java`

- [ ] **Step 1: 创建数据库初始化脚本**

```sql
CREATE DATABASE IF NOT EXISTS aistudy DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE aistudy;

-- 用户表
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(100) NOT NULL COMMENT '密码',
  `nickname` varchar(50) NOT NULL COMMENT '昵称',
  `avatar` varchar(255) DEFAULT NULL COMMENT '头像URL',
  `coins` int(11) DEFAULT 0 COMMENT '学习币数量',
  `points` int(11) DEFAULT 0 COMMENT '积分',
  `level` int(11) DEFAULT 1 COMMENT '等级',
  `sign_in_count` int(11) DEFAULT 0 COMMENT '连续签到天数',
  `last_sign_in_date` date DEFAULT NULL COMMENT '最后签到日期',
  `role` varchar(20) DEFAULT 'USER' COMMENT '角色',
  `status` tinyint(1) DEFAULT 1 COMMENT '状态：0-禁用，1-正常',
  `deleted` tinyint(1) DEFAULT 0 COMMENT '删除标记',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- 帖子表
CREATE TABLE `post` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '帖子ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `title` varchar(100) NOT NULL COMMENT '标题',
  `content` text NOT NULL COMMENT '内容',
  `price` int(11) DEFAULT 0 COMMENT '价格（学习币）',
  `view_count` int(11) DEFAULT 0 COMMENT '浏览次数',
  `like_count` int(11) DEFAULT 0 COMMENT '点赞数',
  `comment_count` int(11) DEFAULT 0 COMMENT '评论数',
  `is_pinned` tinyint(1) DEFAULT 0 COMMENT '是否置顶',
  `status` tinyint(1) DEFAULT 1 COMMENT '状态：0-草稿，1-发布，2-删除',
  `deleted` tinyint(1) DEFAULT 0 COMMENT '删除标记',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='帖子表';

-- 评论表
CREATE TABLE `comment` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '评论ID',
  `post_id` bigint(20) NOT NULL COMMENT '帖子ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `content` varchar(500) NOT NULL COMMENT '评论内容',
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父评论ID',
  `like_count` int(11) DEFAULT 0 COMMENT '点赞数',
  `status` tinyint(1) DEFAULT 1 COMMENT '状态：0-删除，1-正常',
  `deleted` tinyint(1) DEFAULT 0 COMMENT '删除标记',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_post_id` (`post_id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_parent_id` (`parent_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论表';

-- 学习币交易记录表
CREATE TABLE `coin_transaction` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '交易ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `type` varchar(20) NOT NULL COMMENT '类型：SIGN_IN-签到, COMMENT-评论, VIEW-浏览, PURCHASE-购买, SELL-出售',
  `amount` int(11) NOT NULL COMMENT '金额（正数为收入，负数为支出）',
  `balance` int(11) NOT NULL COMMENT '余额',
  `description` varchar(100) DEFAULT NULL COMMENT '描述',
  `related_id` bigint(20) DEFAULT NULL COMMENT '相关ID（帖子ID、评论ID等）',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_type` (`type`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='学习币交易记录表';

-- 每日签到记录表
CREATE TABLE `daily_sign_in` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '签到记录ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `sign_in_date` date NOT NULL COMMENT '签到日期',
  `coins_earned` int(11) NOT NULL COMMENT '获得学习币',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_date` (`user_id`, `sign_in_date`),
  KEY `idx_sign_in_date` (`sign_in_date`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='每日签到记录表';
```

- [ ] **Step 2: 创建用户实体类**

```java
package com.aistudy.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDateTime;

@Data
@EqualsAndHashCode(callSuper = false)
@TableName("user")
public class User {
    
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    
    @TableField("username")
    private String username;
    
    @TableField("password")
    private String password;
    
    @TableField("nickname")
    private String nickname;
    
    @TableField("avatar")
    private String avatar;
    
    @TableField("coins")
    private Integer coins;
    
    @TableField("points")
    private Integer points;
    
    @TableField("level")
    private Integer level;
    
    @TableField("sign_in_count")
    private Integer signInCount;
    
    @TableField("last_sign_in_date")
    private String lastSignInDate;
    
    @TableField("role")
    private String role;
    
    @TableField("status")
    private Integer status;
    
    @TableLogic
    @TableField("deleted")
    private Integer deleted;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

- [ ] **Step 3: 创建帖子实体类**

```java
package com.aistudy.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDateTime;

@Data
@EqualsAndHashCode(callSuper = false)
@TableName("post")
public class Post {
    
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    
    @TableField("user_id")
    private Long userId;
    
    @TableField("title")
    private String title;
    
    @TableField("content")
    private String content;
    
    @TableField("price")
    private Integer price;
    
    @TableField("view_count")
    private Integer viewCount;
    
    @TableField("like_count")
    private Integer likeCount;
    
    @TableField("comment_count")
    private Integer commentCount;
    
    @TableField("is_pinned")
    private Integer isPinned;
    
    @TableField("status")
    private Integer status;
    
    @TableLogic
    @TableField("deleted")
    private Integer deleted;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

- [ ] **Step 4: 创建评论实体类**

```java
package com.aistudy.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDateTime;

@Data
@EqualsAndHashCode(callSuper = false)
@TableName("comment")
public class Comment {
    
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    
    @TableField("post_id")
    private Long postId;
    
    @TableField("user_id")
    private Long userId;
    
    @TableField("content")
    private String content;
    
    @TableField("parent_id")
    private Long parentId;
    
    @TableField("like_count")
    private Integer likeCount;
    
    @TableField("status")
    private Integer status;
    
    @TableLogic
    @TableField("deleted")
    private Integer deleted;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

- [ ] **Step 5: 创建学习币交易记录实体类**

```java
package com.aistudy.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDateTime;

@Data
@EqualsAndHashCode(callSuper = false)
@TableName("coin_transaction")
public class CoinTransaction {
    
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    
    @TableField("user_id")
    private Long userId;
    
    @TableField("type")
    private String type;
    
    @TableField("amount")
    private Integer amount;
    
    @TableField("balance")
    private Integer balance;
    
    @TableField("description")
    private String description;
    
    @TableField("related_id")
    private Long relatedId;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
}
```

- [ ] **Step 6: 创建每日签到记录实体类**

```java
package com.aistudy.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDateTime;

@Data
@EqualsAndHashCode(callSuper = false)
@TableName("daily_sign_in")
public class DailySignIn {
    
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    
    @TableField("user_id")
    private Long userId;
    
    @TableField("sign_in_date")
    private String signInDate;
    
    @TableField("coins_earned")
    private Integer coinsEarned;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
}
```

- [ ] **Step 7: Commit**

```bash
git add .
git commit -m "feat: 添加数据库设计和实体类"
```

### Task 3: 配置和数据访问层

**Files:**
- Create: `backend/src/main/java/com/aistudy/config/JacksonConfig.java`
- Create: `backend/src/main/java/com/aistudy/config/RedisConfig.java`
- Create: `backend/src/main/java/com/aistudy/config/SwaggerConfig.java`
- Create: `backend/src/main/java/com/aistudy/config/MyBatisPlusConfig.java`
- Create: `backend/src/main/java/com/aistudy/mapper/UserMapper.java`
- Create: `backend/src/main/java/com/aistudy/mapper/PostMapper.java`
- Create: `backend/src/main/java/com/aistudy/mapper/CommentMapper.java`
- Create: `backend/src/main/java/com/aistudy/mapper/CoinTransactionMapper.java`
- Create: `backend/src/main/java/com/aistudy/mapper/DailySignInMapper.java`

- [ ] **Step 1: 创建通用响应类**

```java
package com.aistudy.common;

import lombok.Data;

@Data
public class R<T> {
    
    private Integer code;
    private String message;
    private T data;
    
    public R() {}
    
    public R(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
    
    public R(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }
    
    public static <T> R<T> success() {
        return new R<>(200, "success");
    }
    
    public static <T> R<T> success(T data) {
        return new R<>(200, "success", data);
    }
    
    public static <T> R<T> error(String message) {
        return new R<>(500, message);
    }
    
    public static <T> R<T> error(Integer code, String message) {
        return new R<>(code, message);
    }
}
```

- [ ] **Step 2: 创建Jackson配置**

```java
package com.aistudy.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Configuration
public class JacksonConfig {
    
    @Bean
    public ObjectMapper objectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        simpleModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        
        objectMapper.registerModule(simpleModule);
        return objectMapper;
    }
}
```

- [ ] **Step 3: 创建Redis配置**

```java
package com.aistudy.config;

import org.springframework.cache.CacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        
        template.afterPropertiesSet();
        return template;
    }
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();
    }
}
```

- [ ] **Step 4: 创建Swagger配置**

```java
package com.aistudy.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("AI学习交流平台API")
                        .version("1.0.0")
                        .description("AI学习交流平台后端API文档")
                        .contact(new Contact()
                                .name("AIStudy Team")
                                .email("support@aistudy.com")));
    }
}
```

- [ ] **Step 5: 创建MyBatis Plus配置**

```java
package com.aistudy.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyBatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

- [ ] **Step 6: 创建Mapper接口**

```java
package com.aistudy.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.aistudy.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

```java
package com.aistudy.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.aistudy.entity.Post;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface PostMapper extends BaseMapper<Post> {
}
```

```java
package com.aistudy.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.aistudy.entity.Comment;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface CommentMapper extends BaseMapper<Comment> {
}
```

```java
package com.aistudy.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.aistudy.entity.CoinTransaction;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface CoinTransactionMapper extends BaseMapper<CoinTransaction> {
}
```

```java
package com.aistudy.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.aistudy.entity.DailySignIn;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface DailySignInMapper extends BaseMapper<DailySignIn> {
}
```

- [ ] **Step 7: 创建填充配置**

```java
package com.aistudy.config;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
    
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

- [ ] **Step 8: Commit**

```bash
git add .
git commit -m "feat: 添加配置和数据访问层"
```

### Task 4: 用户认证和授权

**Files:**
- Create: `backend/src/main/java/com/aistudy/common/Constants.java`
- Create: `backend/src/main/java/com/aistudy/common/JwtUtil.java`
- Create: `backend/src/main/java/com/aistudy/config/WebSecurityConfig.java`
- Create: `backend/src/main/java/com/aistudy/service/UserService.java`
- Create: `backend/src/main/java/com/aistudy/service/impl/UserServiceImpl.java`
- Create: `backend/src/main/java/com/aistudy/controller/UserController.java`
- Create: `backend/src/main/java/com/aistudy/vo/LoginVO.java`
- Create: `backend/src/main/java/com/aistudy/vo/RegisterVO.java`

- [ ] **Step 1: 创建常量类**

```java
package com.aistudy.common;

public class Constants {
    
    public static final String JWT_HEADER = "Authorization";
    
    public static final String USER_ROLE = "USER";
    public static final String ADMIN_ROLE = "ADMIN";
    
    public static final int SIGN_IN_COINS = 5;  // 签到获得学习币
    public static final int COMMENT_COINS = 2; // 评论获得学习币
    public static final int VIEW_COINS = 1;    // 浏览获得学习币
    
    public static final String COIN_TYPE_SIGN_IN = "SIGN_IN";
    public static final String COIN_TYPE_COMMENT = "COMMENT";
    public static final String COIN_TYPE_VIEW = "VIEW";
    public static final String COIN_TYPE_PURCHASE = "PURCHASE";
    public static final String COIN_TYPE_SELL = "SELL";
}
```

- [ ] **Step 2: 创建JWT工具类**

```java
package com.aistudy.common;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expire}")
    private Long expire;
    
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }
    
    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }
    
    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }
    
    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, userDetails.getUsername());
    }
    
    private String doGenerateToken(Map<String, Object> claims, String subject) {
        final Date createdDate = new Date();
        final Date expirationDate = new Date(createdDate.getTime() + expire * 1000);
        
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(createdDate)
                .setExpiration(expirationDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

- [ ] **Step 3: 创建Spring Security配置**

```java
package com.aistudy.config;

import com.aistudy.common.JwtUtil;
import com.aistudy.security.JwtAuthenticationFilter;
import com.aistudy.security.JwtAuthenticationProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }
    
    @Bean
    public JwtAuthenticationProvider jwtAuthenticationProvider() {
        return new JwtAuthenticationProvider(jwtUtil, userDetailsService);
    }
    
    @Bean
    public JwtAuthenticationFilter jwtAuthenticationFilter() {
        return new JwtAuthenticationFilter(jwtUtil, userDetailsService);
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api-docs/**", "/swagger-ui/**", "/swagger-ui.html").permitAll()
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:5173", "http://localhost:3000"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("Content-Type", "Authorization"));
        configuration.setExposedHeaders(Arrays.asList("Authorization"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

- [ ] **Step 4: 创建用户详情服务**

```java
package com.aistudy.security;

import com.aistudy.entity.User;
import com.aistudy.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userMapper.selectOne(
            new com.baomidou.mybatisplus.core.conditions.query.QueryWrapper<User>()
                .eq("username", username)
                .eq("status", 1)
        );
        
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        
        GrantedAuthority authority = new SimpleGrantedAuthority("ROLE_" + user.getRole());
        
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            Collections.singletonList(authority)
        );
    }
}
```

- [ ] **Step 5: 创建JWT认证过滤器**

```java
package com.aistudy.security;

import com.aistudy.common.JwtUtil;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(JwtUtil jwtUtil, UserDetailsService userDetailsService) {
        this.jwtUtil = jwtUtil;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                  FilterChain filterChain) throws ServletException, IOException {
        final String requestTokenHeader = request.getHeader("Authorization");
        
        String username = null;
        String jwt = null;
        
        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwt = requestTokenHeader.substring(7);
            username = jwtUtil.getUsernameFromToken(jwt);
        }
        
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
            
            if (jwtUtil.validateToken(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

- [ ] **Step 6: 创建JWT认证提供者**

```java
package com.aistudy.security;

import com.aistudy.common.JwtUtil;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.stereotype.Component;

@Component
public class JwtAuthenticationProvider implements AuthenticationProvider {
    
    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationProvider(JwtUtil jwtUtil, UserDetailsService userDetailsService) {
        this.jwtUtil = jwtUtil;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String token = (String) authentication.getCredentials();
        String username = jwtUtil.getUsernameFromToken(token);
        
        if (username == null || !jwtUtil.validateToken(token, userDetailsService.loadUserByUsername(username))) {
            throw new RuntimeException("Invalid JWT token");
        }
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        authentication = new UsernamePasswordAuthenticationToken(
            userDetails, token, userDetails.getAuthorities());
        
        return authentication;
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return JwtAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

- [ ] **Step 7: 创建JWT认证令牌**

```java
package com.aistudy.security;

import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.core.GrantedAuthority;

import java.util.Collection;

public class JwtAuthenticationToken extends AbstractAuthenticationToken {
    
    private final String token;
    
    public JwtAuthenticationToken(String token) {
        super(null);
        this.token = token;
        this.setAuthenticated(false);
    }
    
    public JwtAuthenticationToken(String token, Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.token = token;
        this.setAuthenticated(true);
    }
    
    @Override
    public Object getCredentials() {
        return token;
    }
    
    @Override
    public Object getPrincipal() {
        return null;
    }
}
```

- [ ] **Step 8: 创建用户Service**

```java
package com.aistudy.service;

import com.aistudy.entity.User;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.IService;

public interface UserService extends IService<User> {
    
    User register(User user);
    
    User login(String username, String password);
    
    Page<User> getUserRanking(Page<User> page);
    
    boolean signIn(Long userId);
    
    boolean checkUsernameExist(String username);
    
    boolean checkNicknameExist(String nickname);
    
    User getUserById(Long userId);
}
```

- [ ] **Step 9: 创建用户Service实现**

```java
package com.aistudy.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.User;
import com.aistudy.mapper.UserMapper;
import com.aistudy.service.UserService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.aistudy.common.Constants;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

@Service
@Transactional
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public User register(User user) {
        if (checkUsernameExist(user.getUsername())) {
            throw new RuntimeException("用户名已存在");
        }
        if (checkNicknameExist(user.getNickname())) {
            throw new RuntimeException("昵称已存在");
        }
        
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        user.setCoins(0);
        user.setPoints(0);
        user.setLevel(1);
        user.setSignInCount(0);
        user.setStatus(1);
        user.setRole(Constants.USER_ROLE);
        
        save(user);
        return user;
    }
    
    @Override
    public User login(String username, String password) {
        User user = getOne(new LambdaQueryWrapper<User>()
            .eq(User::getUsername, username)
            .eq(User::getStatus, 1));
        
        if (user == null || !passwordEncoder.matches(password, user.getPassword())) {
            throw new RuntimeException("用户名或密码错误");
        }
        
        return user;
    }
    
    @Override
    public Page<User> getUserRanking(Page<User> page) {
        return page(page, new LambdaQueryWrapper<User>()
            .eq(User::getStatus, 1)
            .orderByDesc(User::getPoints)
            .orderByDesc(User::getSignInCount)
            .last("LIMIT 50"));
    }
    
    @Override
    public boolean signIn(Long userId) {
        User user = getById(userId);
        if (user == null) {
            throw new RuntimeException("用户不存在");
        }
        
        LocalDate today = LocalDate.now();
        String todayStr = today.format(DateTimeFormatter.ISO_DATE);
        String lastSignInDate = user.getLastSignInDate();
        
        // 检查今天是否已经签到
        if (todayStr.equals(lastSignInDate)) {
            throw new RuntimeException("今日已签到");
        }
        
        // 更新用户信息
        user.setCoins(user.getCoins() + Constants.SIGN_IN_COINS);
        user.setPoints(user.getPoints() + 10);
        user.setSignInCount(user.getSignInCount() + 1);
        user.setLastSignInDate(todayStr);
        
        // 检查连续签到
        if (lastSignInDate != null) {
            LocalDate lastDate = LocalDate.parse(lastSignInDate, DateTimeFormatter.ISO_DATE);
            if (today.minusDays(1).equals(lastDate)) {
                // 连续签到，额外奖励
                user.setCoins(user.getCoins() + 5);
                user.setPoints(user.getPoints() + 5);
            }
        }
        
        updateById(user);
        return true;
    }
    
    @Override
    public boolean checkUsernameExist(String username) {
        return count(new LambdaQueryWrapper<User>()
            .eq(User::getUsername, username)) > 0;
    }
    
    @Override
    public boolean checkNicknameExist(String nickname) {
        return count(new LambdaQueryWrapper<User>()
            .eq(User::getNickname, nickname)) > 0;
    }
    
    @Override
    public User getUserById(Long userId) {
        return getById(userId);
    }
}
```

- [ ] **Step 9: 创建用户VO**

```java
package com.aistudy.vo;

import lombok.Data;

@Data
public class LoginVO {
    private String username;
    private String password;
}
```

```java
package com.aistudy.vo;

import lombok.Data;

@Data
public class RegisterVO {
    private String username;
    private String password;
    private String nickname;
    private String confirmPassword;
}
```

- [ ] **Step 10: 创建用户Controller**

```java
package com.aistudy.controller;

import com.aistudy.common.R;
import com.aistudy.entity.User;
import com.aistudy.service.UserService;
import com.aistudy.vo.LoginVO;
import com.aistudy.vo.RegisterVO;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
@CrossOrigin(origins = "http://localhost:5173")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @PostMapping("/register")
    public R<String> register(@Valid @RequestBody RegisterVO registerVO) {
        if (!registerVO.getPassword().equals(registerVO.getConfirmPassword())) {
            return R.error("两次密码不一致");
        }
        
        User user = new User();
        user.setUsername(registerVO.getUsername());
        user.setPassword(registerVO.getPassword());
        user.setNickname(registerVO.getNickname());
        
        user = userService.register(user);
        return R.success("注册成功");
    }
    
    @PostMapping("/login")
    public R<User> login(@Valid @RequestBody LoginVO loginVO) {
        User user = userService.login(loginVO.getUsername(), loginVO.getPassword());
        return R.success(user);
    }
    
    @GetMapping("/ranking")
    public R<Page<User>> getRanking(@RequestParam(defaultValue = "1") int page,
                                  @RequestParam(defaultValue = "10") int size) {
        Page<User> pageParam = new Page<>(page, size);
        pageParam = userService.getUserRanking(pageParam);
        return R.success(pageParam);
    }
    
    @PostMapping("/sign-in")
    @PreAuthorize("hasRole('USER')")
    public R<String> signIn() {
        Long userId = getCurrentUserId();
        userService.signIn(userId);
        return R.success("签到成功");
    }
    
    private Long getCurrentUserId() {
        return 1L; // TODO: 从Security Context中获取当前用户ID
    }
}
```

- [ ] **Step 11: Commit**

```bash
git add .
git commit -m "feat: 实现用户认证和授权系统"
```

### Task 5: 帖子管理功能

**Files:**
- Create: `backend/src/main/java/com/aistudy/service/PostService.java`
- Create: `backend/src/main/java/com/aistudy/service/impl/PostServiceImpl.java`
- Create: `backend/src/main/java/com/aistudy/service/CommentService.java`
- Create: `backend/src/main/java/com/aistudy/service/impl/CommentServiceImpl.java`
- Create: `backend/src/main/java/com/aistudy/service/CoinService.java`
- Create: `backend/src/main/java/com/aistudy/service/impl/CoinServiceImpl.java`
- Create: `backend/src/main/java/com/aistudy/controller/PostController.java`
- Create: `backend/src/main/java/com/aistudy/controller/CommentController.java`
- Create: `backend/src/main/java/com/aistudy/vo/PostVO.java`
- Create: `backend/src/main/java/com/aistudy/vo/CommentVO.java`
- Create: `backend/src/main/java/com/aistudy/dto/PurchasePostDTO.java`

- [ ] **Step 1: 创建帖子Service**

```java
package com.aistudy.service;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Post;
import com.baomidou.mybatisplus.extension.service.IService;
import com.aistudy.vo.PostVO;

public interface PostService extends IService<Post> {
    
    Page<PostVO> getPostList(Page<Post> page, String keyword, Long userId);
    
    PostVO getPostDetail(Long postId, Long currentUserId);
    
    boolean createPost(Post post);
    
    boolean updatePost(Post post);
    
    boolean deletePost(Long postId, Long userId);
    
    boolean purchasePost(Long postId, Long userId);
    
    boolean likePost(Long postId, Long userId);
    
    boolean addViewCount(Long postId);
    
    Page<PostVO> getHotPosts(Page<Post> page);
    
    Page<PostVO> getPinnedPosts(Page<Post> page);
}
```

- [ ] **Step 2: 创建帖子Service实现**

```java
package com.aistudy.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Post;
import com.aistudy.entity.Comment;
import com.aistudy.entity.CoinTransaction;
import com.aistudy.mapper.PostMapper;
import com.aistudy.mapper.CommentMapper;
import com.aistudy.mapper.CoinTransactionMapper;
import com.aistudy.service.PostService;
import com.aistudy.service.CommentService;
import com.aistudy.service.CoinService;
import com.aistudy.vo.PostVO;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@Transactional
public class PostServiceImpl extends ServiceImpl<PostMapper, Post> implements PostService {
    
    @Autowired
    private CommentService commentService;
    
    @Autowired
    private CoinService coinService;
    
    @Autowired
    private CoinTransactionMapper coinTransactionMapper;
    
    @Override
    public Page<PostVO> getPostList(Page<Post> page, String keyword, Long userId) {
        LambdaQueryWrapper<Post> queryWrapper = new LambdaQueryWrapper<Post>()
            .eq(Post::getStatus, 1)
            .eq(Post::getDeleted, 0)
            .like(keyword != null, Post::getTitle, keyword)
            .like(keyword != null, Post::getContent, keyword)
            .orderByDesc(Post::getIsPinned)
            .orderByDesc(Post::getCreateTime);
        
        if (userId != null) {
            queryWrapper.eq(Post::getUserId, userId);
        }
        
        Page<Post> postPage = page(page, queryWrapper);
        
        Page<PostVO> result = new Page<>();
        BeanUtils.copyProperties(postPage, result);
        
        List<PostVO> postVOList = postPage.getRecords().stream().map(post -> {
            PostVO postVO = new PostVO();
            BeanUtils.copyProperties(post, postVO);
            // TODO: 设置作者信息
            return postVO;
        }).collect(Collectors.toList());
        
        result.setRecords(postVOList);
        return result;
    }
    
    @Override
    public PostVO getPostDetail(Long postId, Long currentUserId) {
        Post post = getById(postId);
        if (post == null || post.getStatus() != 1 || post.getDeleted() == 1) {
            throw new RuntimeException("帖子不存在");
        }
        
        PostVO postVO = new PostVO();
        BeanUtils.copyProperties(post, postVO);
        
        // 增加浏览次数
        addViewCount(postId);
        
        // 给当前用户增加浏览学习币
        if (currentUserId != null) {
            coinService.addCoins(currentUserId, Constants.VIEW_COINS, Constants.COIN_TYPE_VIEW, "浏览帖子获得", postId);
        }
        
        return postVO;
    }
    
    @Override
    public boolean createPost(Post post) {
        post.setViewCount(0);
        post.setLikeCount(0);
        post.setCommentCount(0);
        post.setStatus(1);
        post.setDeleted(0);
        save(post);
        return true;
    }
    
    @Override
    public boolean updatePost(Post post) {
        Post existingPost = getById(post.getId());
        if (existingPost == null || existingPost.getUserId().equals(post.getUserId())) {
            throw new RuntimeException("无权限修改此帖子");
        }
        
        post.setUpdateTime(LocalDateTime.now());
        updateById(post);
        return true;
    }
    
    @Override
    public boolean deletePost(Long postId, Long userId) {
        Post post = getById(postId);
        if (post == null || !post.getUserId().equals(userId)) {
            throw new RuntimeException("无权限删除此帖子");
        }
        
        post.setDeleted(1);
        post.setStatus(0);
        updateById(post);
        return true;
    }
    
    @Override
    public boolean purchasePost(Long postId, Long userId) {
        Post post = getById(postId);
        if (post == null || post.getStatus() != 1 || post.getDeleted() == 1) {
            throw new RuntimeException("帖子不存在");
        }
        
        if (post.getUserId().equals(userId)) {
            throw new RuntimeException("不能购买自己的帖子");
        }
        
        // 扣除购买者的学习币
        coinService.deductCoins(userId, post.getPrice(), "购买帖子: " + post.getTitle());
        
        // 增加出售者的学习币
        coinService.addCoins(post.getUserId(), post.getPrice(), Constants.COIN_TYPE_SELL, "出售帖子获得", postId);
        
        // 记录交易
        CoinTransaction purchaseRecord = new CoinTransaction();
        purchaseRecord.setUserId(userId);
        purchaseRecord.setType(Constants.COIN_TYPE_PURCHASE);
        purchaseRecord.setAmount(-post.getPrice());
        purchaseRecord.setBalance(coinService.getUserBalance(userId));
        purchaseRecord.setDescription("购买帖子: " + post.getTitle());
        purchaseRecord.setRelatedId(postId);
        coinTransactionMapper.insert(purchaseRecord);
        
        CoinTransaction sellRecord = new CoinTransaction();
        sellRecord.setUserId(post.getUserId());
        sellRecord.setType(Constants.COIN_TYPE_SELL);
        sellRecord.setAmount(post.getPrice());
        sellRecord.setBalance(coinService.getUserBalance(post.getUserId()));
        sellRecord.setDescription("出售帖子获得");
        sellRecord.setRelatedId(postId);
        coinTransactionMapper.insert(sellRecord);
        
        return true;
    }
    
    @Override
    public boolean likePost(Long postId, Long userId) {
        // TODO: 实现点赞功能
        return true;
    }
    
    @Override
    public boolean addViewCount(Long postId) {
        Post post = getById(postId);
        if (post != null) {
            post.setViewCount(post.getViewCount() + 1);
            updateById(post);
        }
        return true;
    }
    
    @Override
    public Page<PostVO> getHotPosts(Page<Post> page) {
        LambdaQueryWrapper<Post> queryWrapper = new LambdaQueryWrapper<Post>()
            .eq(Post::getStatus, 1)
            .eq(Post::getDeleted, 0)
            .orderByDesc(Post::getViewCount)
            .orderByDesc(Post::getLikeCount)
            .orderByDesc(Post::getCommentCount)
            .orderByDesc(Post::getCreateTime);
        
        Page<Post> postPage = page(page, queryWrapper);
        
        Page<PostVO> result = new Page<>();
        BeanUtils.copyProperties(postPage, result);
        
        List<PostVO> postVOList = postPage.getRecords().stream().map(post -> {
            PostVO postVO = new PostVO();
            BeanUtils.copyProperties(post, postVO);
            return postVO;
        }).collect(Collectors.toList());
        
        result.setRecords(postVOList);
        return result;
    }
    
    @Override
    public Page<PostVO> getPinnedPosts(Page<Post> page) {
        LambdaQueryWrapper<Post> queryWrapper = new LambdaQueryWrapper<Post>()
            .eq(Post::getStatus, 1)
            .eq(Post::getDeleted, 0)
            .eq(Post::getIsPinned, 1)
            .orderByDesc(Post::getCreateTime);
        
        Page<Post> postPage = page(page, queryWrapper);
        
        Page<PostVO> result = new Page<>();
        BeanUtils.copyProperties(postPage, result);
        
        List<PostVO> postVOList = postPage.getRecords().stream().map(post -> {
            PostVO postVO = new PostVO();
            BeanUtils.copyProperties(post, postVO);
            return postVO;
        }).collect(Collectors.toList());
        
        result.setRecords(postVOList);
        return result;
    }
}
```

- [ ] **Step 3: 创建评论Service**

```java
package com.aistudy.service;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Comment;
import com.baomidou.mybatisplus.extension.service.IService;

public interface CommentService extends IService<Comment> {
    
    Page<Comment> getCommentList(Page<Comment> page, Long postId);
    
    boolean createComment(Comment comment);
    
    boolean deleteComment(Long commentId, Long userId);
    
    boolean likeComment(Long commentId, Long userId);
}
```

- [ ] **Step 4: 创建评论Service实现**

```java
package com.aistudy.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Comment;
import com.aistudy.entity.CoinTransaction;
import com.aistudy.mapper.CommentMapper;
import com.aistudy.mapper.CoinTransactionMapper;
import com.aistudy.service.CommentService;
import com.aistudy.service.CoinService;
import com.aistudy.common.Constants;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

@Service
@Transactional
public class CommentServiceImpl extends ServiceImpl<CommentMapper, Comment> implements CommentService {
    
    @Autowired
    private CoinService coinService;
    
    @Autowired
    private CoinTransactionMapper coinTransactionMapper;
    
    @Override
    public Page<Comment> getCommentList(Page<Comment> page, Long postId) {
        LambdaQueryWrapper<Comment> queryWrapper = new LambdaQueryWrapper<Comment>()
            .eq(Comment::getPostId, postId)
            .eq(Comment::getStatus, 1)
            .eq(Comment::getDeleted, 0)
            .isNull(Comment::getParentId)
            .orderByDesc(Comment::getCreateTime);
        
        return page(page, queryWrapper);
    }
    
    @Override
    public boolean createComment(Comment comment) {
        comment.setLikeCount(0);
        comment.setStatus(1);
        comment.setDeleted(0);
        save(comment);
        
        // 给评论者增加学习币
        coinService.addCoins(comment.getUserId(), Constants.COMMENT_COINS, Constants.COIN_TYPE_COMMENT, "评论获得", comment.getId());
        
        return true;
    }
    
    @Override
    public boolean deleteComment(Long commentId, Long userId) {
        Comment comment = getById(commentId);
        if (comment == null || !comment.getUserId().equals(userId)) {
            throw new RuntimeException("无权限删除此评论");
        }
        
        comment.setDeleted(1);
        comment.setStatus(0);
        updateById(comment);
        return true;
    }
    
    @Override
    public boolean likeComment(Long commentId, Long userId) {
        // TODO: 实现评论点赞功能
        return true;
    }
}
```

- [ ] **Step 5: 创建学习币Service**

```java
package com.aistudy.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.aistudy.entity.CoinTransaction;

public interface CoinService extends IService<CoinTransaction> {
    
    int getUserBalance(Long userId);
    
    boolean addCoins(Long userId, int amount, String type, String description, Long relatedId);
    
    boolean deductCoins(Long userId, int amount, String description);
    
    Page<CoinTransaction> getTransactionHistory(Page<CoinTransaction> page, Long userId);
}
```

- [ ] **Step 6: 创建学习币Service实现**

```java
package com.aistudy.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.CoinTransaction;
import com.aistudy.entity.User;
import com.aistudy.mapper.CoinTransactionMapper;
import com.aistudy.mapper.UserMapper;
import com.aistudy.service.CoinService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class CoinServiceImpl extends ServiceImpl<CoinTransactionMapper, CoinTransaction> implements CoinService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    public int getUserBalance(Long userId) {
        User user = userMapper.selectById(userId);
        return user != null ? user.getCoins() : 0;
    }
    
    @Override
    public boolean addCoins(Long userId, int amount, String type, String description, Long relatedId) {
        User user = userMapper.selectById(userId);
        if (user == null) {
            throw new RuntimeException("用户不存在");
        }
        
        user.setCoins(user.getCoins() + amount);
        userMapper.updateById(user);
        
        CoinTransaction transaction = new CoinTransaction();
        transaction.setUserId(userId);
        transaction.setType(type);
        transaction.setAmount(amount);
        transaction.setBalance(user.getCoins());
        transaction.setDescription(description);
        transaction.setRelatedId(relatedId);
        
        save(transaction);
        return true;
    }
    
    @Override
    public boolean deductCoins(Long userId, int amount, String description) {
        User user = userMapper.selectById(userId);
        if (user == null) {
            throw new RuntimeException("用户不存在");
        }
        
        if (user.getCoins() < amount) {
            throw new RuntimeException("学习币不足");
        }
        
        user.setCoins(user.getCoins() - amount);
        userMapper.updateById(user);
        
        CoinTransaction transaction = new CoinTransaction();
        transaction.setUserId(userId);
        transaction.setType(Constants.COIN_TYPE_PURCHASE);
        transaction.setAmount(-amount);
        transaction.setBalance(user.getCoins());
        transaction.setDescription(description);
        
        save(transaction);
        return true;
    }
    
    @Override
    public Page<CoinTransaction> getTransactionHistory(Page<CoinTransaction> page, Long userId) {
        LambdaQueryWrapper<CoinTransaction> queryWrapper = new LambdaQueryWrapper<CoinTransaction>()
            .eq(CoinTransaction::getUserId, userId)
            .orderByDesc(CoinTransaction::getCreateTime);
        
        return page(page, queryWrapper);
    }
}
```

- [ ] **Step 7: 创建帖子Controller**

```java
package com.aistudy.controller;

import com.aistudy.common.R;
import com.aistudy.entity.Post;
import com.aistudy.service.PostService;
import com.aistudy.vo.PostVO;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/posts")
@CrossOrigin(origins = "http://localhost:5173")
public class PostController {
    
    @Autowired
    private PostService postService;
    
    @GetMapping("/list")
    public R<Page<PostVO>> getPostList(@RequestParam(defaultValue = "1") int page,
                                     @RequestParam(defaultValue = "10") int size,
                                     @RequestParam(required = false) String keyword,
                                     @RequestParam(required = false) Long userId) {
        Page<Post> pageParam = new Page<>(page, size);
        Page<PostVO> result = postService.getPostList(pageParam, keyword, userId);
        return R.success(result);
    }
    
    @GetMapping("/{postId}")
    public R<PostVO> getPostDetail(@PathVariable Long postId,
                                 @RequestParam(required = false) Long currentUserId) {
        PostVO postVO = postService.getPostDetail(postId, currentUserId);
        return R.success(postVO);
    }
    
    @PostMapping("/create")
    @PreAuthorize("hasRole('USER')")
    public R<String> createPost(@Valid @RequestBody Post post) {
        // TODO: 从Security Context中获取当前用户ID
        post.setUserId(1L);
        postService.createPost(post);
        return R.success("发帖成功");
    }
    
    @PutMapping("/update")
    @PreAuthorize("hasRole('USER')")
    public R<String> updatePost(@Valid @RequestBody Post post) {
        // TODO: 从Security Context中获取当前用户ID
        post.setUserId(1L);
        postService.updatePost(post);
        return R.success("更新成功");
    }
    
    @DeleteMapping("/{postId}")
    @PreAuthorize("hasRole('USER')")
    public R<String> deletePost(@PathVariable Long postId) {
        // TODO: 从Security Context中获取当前用户ID
        Long userId = 1L;
        postService.deletePost(postId, userId);
        return R.success("删除成功");
    }
    
    @PostMapping("/{postId}/purchase")
    @PreAuthorize("hasRole('USER')")
    public R<String> purchasePost(@PathVariable Long postId) {
        // TODO: 从Security Context中获取当前用户ID
        Long userId = 1L;
        postService.purchasePost(postId, userId);
        return R.success("购买成功");
    }
    
    @PostMapping("/{postId}/like")
    @PreAuthorize("hasRole('USER')")
    public R<String> likePost(@PathVariable Long postId) {
        // TODO: 从Security Context中获取当前用户ID
        Long userId = 1L;
        postService.likePost(postId, userId);
        return R.success("点赞成功");
    }
    
    @GetMapping("/hot")
    public R<Page<PostVO>> getHotPosts(@RequestParam(defaultValue = "1") int page,
                                     @RequestParam(defaultValue = "10") int size) {
        Page<Post> pageParam = new Page<>(page, size);
        Page<PostVO> result = postService.getHotPosts(pageParam);
        return R.success(result);
    }
    
    @GetMapping("/pinned")
    public R<Page<PostVO>> getPinnedPosts(@RequestParam(defaultValue = "1") int page,
                                        @RequestParam(defaultValue = "10") int size) {
        Page<Post> pageParam = new Page<>(page, size);
        Page<PostVO> result = postService.getPinnedPosts(pageParam);
        return R.success(result);
    }
}
```

- [ ] **Step 8: 创建评论Controller**

```java
package com.aistudy.controller;

import com.aistudy.common.R;
import com.aistudy.entity.Comment;
import com.aistudy.service.CommentService;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/comments")
@CrossOrigin(origins = "http://localhost:5173")
public class CommentController {
    
    @Autowired
    private CommentService commentService;
    
    @GetMapping("/list/{postId}")
    public R<Page<Comment>> getCommentList(@PathVariable Long postId,
                                         @RequestParam(defaultValue = "1") int page,
                                         @RequestParam(defaultValue = "10") int size) {
        Page<Comment> pageParam = new Page<>(page, size);
        Page<Comment> result = commentService.getCommentList(pageParam, postId);
        return R.success(result);
    }
    
    @PostMapping("/create")
    @PreAuthorize("hasRole('USER')")
    public R<String> createComment(@Valid @RequestBody Comment comment) {
        // TODO: 从Security Context中获取当前用户ID
        comment.setUserId(1L);
        commentService.createComment(comment);
        return R.success("评论成功");
    }
    
    @DeleteMapping("/{commentId}")
    @PreAuthorize("hasRole('USER')")
    public R<String> deleteComment(@PathVariable Long commentId) {
        // TODO: 从Security Context中获取当前用户ID
        Long userId = 1L;
        commentService.deleteComment(commentId, userId);
        return R.success("删除成功");
    }
    
    @PostMapping("/{commentId}/like")
    @PreAuthorize("hasRole('USER')")
    public R<String> likeComment(@PathVariable Long commentId) {
        // TODO: 从Security Context中获取当前用户ID
        Long userId = 1L;
        commentService.likeComment(commentId, userId);
        return R.success("点赞成功");
    }
}
```

- [ ] **Step 9: 创建帖子VO**

```java
package com.aistudy.vo;

import lombok.Data;
import java.time.LocalDateTime;

@Data
public class PostVO {
    private Long id;
    private Long userId;
    private String title;
    private String content;
    private Integer price;
    private Integer viewCount;
    private Integer likeCount;
    private Integer commentCount;
    private Integer isPinned;
    private Integer status;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    
    // 用户信息
    private String authorNickname;
    private String authorAvatar;
    
    // 购买状态
    private Boolean isPurchased;
}
```

- [ ] **Step 10: 创建评论VO**

```java
package com.aistudy.vo;

import lombok.Data;
import java.time.LocalDateTime;

@Data
public class CommentVO {
    private Long id;
    private Long postId;
    private Long userId;
    private String content;
    private Long parentId;
    private Integer likeCount;
    private Integer status;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    
    // 用户信息
    private String authorNickname;
    private String authorAvatar;
    
    // 子评论
    private List<CommentVO> children;
}
```

- [ ] **Step 11: 创建购买DTO**

```java
package com.aistudy.dto;

import lombok.Data;
import jakarta.validation.constraints.NotNull;

@Data
public class PurchasePostDTO {
    @NotNull(message = "帖子ID不能为空")
    private Long postId;
}
```

- [ ] **Step 12: Commit**

```bash
git add .
git commit -m "feat: 添加帖子管理功能"
```

### Task 8: 安全增强模块

**Files:**
- Create: `backend/src/main/java/com/aistudy/security/RateLimitService.java`
- Create: `backend/src/main/java/com/aistudy/security/SensitiveWordFilter.java`
- Create: `backend/src/main/java/com/aistudy/security/IPBlockService.java`
- Create: `backend/src/main/java/com/aistudy/config/SecurityConfig.java`
- Create: `backend/src/main/java/com/aistudy/filter/SecurityFilter.java`
- Create: `backend/src/main/resources/sensitive-words.txt`

- [ ] **Step 1: 创建防刷服务**

```java
package com.aistudy.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class RateLimitService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 滑动窗口限流
     * @param key 限流键
     * @param limit 限制次数
     * @param windowSeconds 时间窗口（秒）
     * @return 是否允许访问
     */
    public boolean checkRateLimit(String key, int limit, int windowSeconds) {
        String redisKey = "rate_limit:" + key;
        
        // Redis原子操作增加计数
        Long count = redisTemplate.opsForValue().increment(redisKey);
        
        // 如果是第一次设置过期时间
        if (count != null && count == 1) {
            redisTemplate.expire(redisKey, windowSeconds, TimeUnit.SECONDS);
        }
        
        // 如果超过限制
        if (count > limit) {
            return false;
        }
        
        return true;
    }
    
    /**
     * 漏桶算法限流
     * @param key 限流键
     * @param capacity 桶容量
     * @param rate 流出速率（每秒）
     * @return 是否允许访问
     */
    public boolean leakyBucketRateLimit(String key, int capacity, double rate) {
        String redisKey = "leaky_bucket:" + key;
        String timestampKey = "leaky_bucket:timestamp:" + key;
        
        // 获取当前时间戳
        long now = System.currentTimeMillis();
        long lastTime = redisTemplate.opsForValue().get(timestampKey) == null ? 
            now : (long) redisTemplate.opsForValue().get(timestampKey);
        
        // 计算当前水量
        double water = Math.max(0, (double) redisTemplate.opsForValue().get(redisKey) - 
            (now - lastTime) * rate / 1000);
        
        // 尝试加水
        if (water < capacity) {
            water += 1;
            redisTemplate.opsForValue().set(redisKey, water);
            redisTemplate.opsForValue().set(timestampKey, now);
            return true;
        }
        
        return false;
    }
}
```

- [ ] **Step 2: 创建敏感词过滤器**

```java
package com.aistudy.security;

import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class SensitiveWordFilter {
    
    private static final String DEFAULT_REPLACEMENT = "***";
    private final Map<String, Object> sensitiveWordMap = new ConcurrentHashMap<>();
    private static final int MIN_MATCH_TYPE = 1;  // 最小匹配规则
    private static final int MAX_MATCH_TYPE = 2;  // 最大匹配规则
    
    @PostConstruct
    public void init() throws IOException {
        // 加载敏感词库
        InputStream is = new ClassPathResource("sensitive-words.txt").getInputStream();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
            String word;
            while ((word = reader.readLine()) != null) {
                if (!word.trim().isEmpty()) {
                    addWordToMap(word.trim());
                }
            }
        }
    }
    
    /**
     * 添加敏感词到内存
     */
    private void addWordToMap(String word) {
        Map<String, Object> currentMap = sensitiveWordMap;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            Map<String, Object> subMap = (Map<String, Object>) currentMap.get(String.valueOf(c));
            if (subMap == null) {
                subMap = new HashMap<>();
                currentMap.put(String.valueOf(c), subMap);
            }
            currentMap = subMap;
        }
        currentMap.put("isEnd", true);
    }
    
    /**
     * 检查文本是否包含敏感词
     */
    public boolean containsSensitiveWord(String text) {
        return checkSensitiveWord(text, MIN_MATCH_TYPE) != null;
    }
    
    /**
     * 获取第一个敏感词
     */
    public String getFirstSensitiveWord(String text) {
        return checkSensitiveWord(text, MIN_MATCH_TYPE);
    }
    
    /**
     * 过滤敏感词
     */
    public String filter(String text) {
        return replaceSensitiveWord(text, DEFAULT_REPLACEMENT);
    }
    
    /**
     * 替换敏感词
     */
    public String replaceSensitiveWord(String text, String replacement) {
        if (text == null || text.isEmpty()) {
            return text;
        }
        
        Set<String> sensitiveWords = checkSensitiveWord(text, MAX_MATCH_TYPE);
        if (sensitiveWords == null || sensitiveWords.isEmpty()) {
            return text;
        }
        
        // 替换敏感词
        for (String word : sensitiveWords) {
            text = text.replace(word, replacement);
        }
        
        return text;
    }
    
    /**
     * 检查敏感词
     */
    private Set<String> checkSensitiveWord(String text, int matchType) {
        Set<String> sensitiveWords = new HashSet<>();
        for (int i = 0; i < text.length(); i++) {
            int length = getSensitiveWordLength(text, i, matchType);
            if (length > 0) {
                String word = text.substring(i, i + length);
                sensitiveWords.add(word);
                if (matchType == MIN_MATCH_TYPE) {
                    return sensitiveWords;
                }
                i += length - 1;
            }
        }
        return sensitiveWords.isEmpty() ? null : sensitiveWords;
    }
    
    /**
     * 获取敏感词长度
     */
    private int getSensitiveWordLength(String text, int beginIndex, int matchType) {
        if (beginIndex < 0 || beginIndex >= text.length()) {
            return 0;
        }
        
        Map<String, Object> currentMap = sensitiveWordMap;
        int wordLength = 0;
        boolean flag = false;
        
        for (int i = beginIndex; i < text.length(); i++) {
            char c = text.charAt(i);
            Map<String, Object> subMap = (Map<String, Object>) currentMap.get(String.valueOf(c));
            
            if (subMap == null) {
                break;
            }
            
            wordLength++;
            if (subMap.get("isEnd") != null) {
                flag = true;
                if (matchType == MIN_MATCH_TYPE) {
                    break;
                }
            }
            currentMap = subMap;
        }
        
        return flag ? wordLength : 0;
    }
}
```

- [ ] **Step 3: 创建IP限制服务**

```java
package com.aistudy.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class IPBlockService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // IP黑名单缓存（小时）
    private static final long BLACKLIST_EXPIRE_HOURS = 24;
    
    // 短时间多次登录失败（分钟）
    private static final int FAILED_LOGIN_COUNT_THRESHOLD = 5;
    private static final int FAILED_LOGIN_TIME_WINDOW = 5;
    
    // 发帖频率限制（分钟）
    private static final int POST_COUNT_THRESHOLD = 10;
    private static final int POST_TIME_WINDOW = 1;
    
    // 评论频率限制（分钟）
    private static final int COMMENT_COUNT_THRESHOLD = 20;
    private static final int COMMENT_TIME_WINDOW = 1;
    
    /**
     * 检查IP是否在黑名单中
     */
    public boolean isIPBlocked(String ip) {
        String blackListKey = "ip:blacklist:" + ip;
        return redisTemplate.hasKey(blackListKey);
    }
    
    /**
     * 将IP加入黑名单
     */
    public void blockIP(String ip, String reason) {
        String blackListKey = "ip:blacklist:" + ip;
        redisTemplate.opsForValue().set(blackListKey, reason, BLACKLIST_EXPIRE_HOURS, TimeUnit.HOURS);
    }
    
    /**
     * 解除IP黑名单
     */
    public void unblockIP(String ip) {
        String blackListKey = "ip:blacklist:" + ip;
        redisTemplate.delete(blackListKey);
    }
    
    /**
     * 记录登录失败
     */
    public void recordLoginFailure(String ip) {
        String key = "ip:login_failed:" + ip;
        Long count = redisTemplate.opsForValue().increment(key);
        
        if (count == 1) {
            redisTemplate.expire(key, FAILED_LOGIN_TIME_WINDOW, TimeUnit.MINUTES);
        }
        
        // 如果达到阈值，加入黑名单
        if (count >= FAILED_LOGIN_COUNT_THRESHOLD) {
            blockIP(ip, "多次登录失败");
        }
    }
    
    /**
     * 清除登录失败记录
     */
    public void clearLoginFailures(String ip) {
        String key = "ip:login_failed:" + ip;
        redisTemplate.delete(key);
    }
    
    /**
     * 检查发帖频率
     */
    public boolean checkPostFrequency(String ip) {
        String key = "ip:post_count:" + ip;
        Long count = redisTemplate.opsForValue().increment(key);
        
        if (count == 1) {
            redisTemplate.expire(key, POST_TIME_WINDOW, TimeUnit.MINUTES);
        }
        
        return count <= POST_COUNT_THRESHOLD;
    }
    
    /**
     * 检查评论频率
     */
    public boolean checkCommentFrequency(String ip) {
        String key = "ip:comment_count:" + ip;
        Long count = redisTemplate.opsForValue().increment(key);
        
        if (count == 1) {
            redisTemplate.expire(key, COMMENT_TIME_WINDOW, TimeUnit.MINUTES);
        }
        
        return count <= COMMENT_COUNT_THRESHOLD;
    }
    
    /**
     * 获取IP状态
     */
    public IPStatus getIPStatus(String ip) {
        IPStatus status = new IPStatus();
        status.ip = ip;
        status.isBlocked = isIPBlocked(ip);
        
        if (!status.isBlocked) {
            status.loginFailures = (Long) redisTemplate.opsForValue().get("ip:login_failed:" + ip) || 0;
            status.postCount = (Long) redisTemplate.opsForValue().get("ip:post_count:" + ip) || 0;
            status.commentCount = (Long) redisTemplate.opsForValue().get("ip:comment_count:" + ip) || 0;
        }
        
        return status;
    }
    
    public static class IPStatus {
        public String ip;
        public boolean isBlocked;
        public long loginFailures;
        public long postCount;
        public long commentCount;
    }
}
```

- [ ] **Step 4: 创建敏感词库文件**

```
# sensitive-words.txt
# 敏感词列表，每行一个

暴力
色情
赌博
毒品
反动
恐怖
分裂
政治
色情
成人
性交易
卖淫
嫖娼
强奸
猥亵
变态
恋童
兽交
毒品
吸毒
贩毒
走私
诈骗
盗窃
抢劫
杀人
自杀
恐怖主义
极端主义
邪教
传销
传销
虚假
广告
垃圾
信息
谣言
谣言
污蔑
诽谤
侮辱
威胁
恐吓
勒索
敲诈
黑客
病毒
木马
钓鱼
诈骗
电话诈骗
网络诈骗
赌博网站
色情网站
成人内容
暴力内容
血腥
恐怖
恶心
令人不适
违法
违规
违反
犯罪
黄赌毒
反社会
自杀倾向
自残
厌世
仇恨
种族歧视
性别歧视
宗教歧视
地域歧视
年龄歧视
身体歧视
智商歧视
学历歧视
职业歧视
家庭歧视
经济歧视
政治观点
敏感话题
政治敏感
军事敏感
外交敏感
国家机密
商业机密
个人隐私
泄露
隐私侵犯
侵权
盗版
盗取
窃取
黑产
暗网
暗网市场
暗网交易
暗网网站
暗网论坛
暗网社区
暗网聊天室
暗网社交
暗网交友
暗网约炮
暗网找小姐
暗网找少爷
暗网找公关
暗网找模特
暗网找演员
暗网找歌手
暗网找舞者
暗网找DJ
暗网找MC
暗网找主播
暗网找网红
```

- [ ] **Step 5: 在Controller中使用安全功能**

```java
@RestController
@RequestMapping("/api/posts")
@CrossOrigin(origins = "http://localhost:5173")
public class PostController {
    
    @Autowired
    private PostService postService;
    
    @Autowired
    private SensitiveWordFilter sensitiveWordFilter;
    
    @PostMapping("/create")
    @PreAuthorize("hasRole('USER')")
    public R<String> createPost(@Valid @RequestBody Post post) {
        // 获取当前用户ID
        Long userId = getCurrentUserId();
        post.setUserId(userId);
        
        // 过滤敏感词
        if (sensitiveWordFilter.containsSensitiveWord(post.getTitle())) {
            return R.error("标题包含敏感词");
        }
        post.setTitle(sensitiveWordFilter.filter(post.getTitle()));
        
        if (sensitiveWordFilter.containsSensitiveWord(post.getContent())) {
            return R.error("内容包含敏感词");
        }
        post.setContent(sensitiveWordFilter.filter(post.getContent()));
        
        postService.createPost(post);
        return R.success("发帖成功");
    }
}
```

- [ ] **Step 6: Commit**

```bash
git add .
git commit -m "feat: 添加安全增强模块"
```

### Task 9: 异步和多线程优化

**Files:**
- Create: `backend/src/main/java/com/aistudy/config/AsyncConfig.java`
- Create: `backend/src/main/java/com/aistudy/service/async/AsyncNotificationService.java`
- Create: `backend/src/main/java/com/aistudy/service/async/AsyncAnalyticsService.java`
- Create: `backend/src/main/java/com/aistudy/service/async/AsyncRecommendationService.java`
- Create: `backend/src/main/java/com/aistudy/service/async/AsyncFileProcessor.java`
- Create: `backend/src/main/java/com/aistudy/async/PostMessage.java`
- Create: `backend/src/main/java/com/aistudy/aspect/AsyncMonitorAspect.java`
- Create: `backend/src/main/java/com/aistudy/exception/AsyncExceptionHandler.java`

- [ ] **Step 1: 创建异步配置**

```java
package com.aistudy.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.web.servlet.config.annotation.AsyncSupportConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.concurrent.Executor;

@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer, WebMvcConfigurer {
    
    @Bean
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("Async-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        executor.initialize();
        return executor;
    }
    
    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        configurer.setTaskExecutor(getAsyncExecutor());
        configurer.setDefaultTimeout(30000);  // 30秒超时
    }
}
```

- [ ] **Step 2: 创建异步通知服务**

```java
package com.aistudy.service.async;

import com.aistudy.entity.Post;
import com.aistudy.entity.User;
import com.aistudy.service.PostService;
import com.aistudy.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.CompletableFuture;

@Slf4j
@Service
public class AsyncNotificationService {
    
    @Autowired
    private PostService postService;
    
    @Autowired
    private UserService userService;
    
    @Async
    public CompletableFuture<Void> sendPostNotification(Long postId, String message) {
        try {
            // 获取帖子作者
            Post post = postService.getById(postId);
            if (post == null) {
                throw new RuntimeException("帖子不存在");
            }
            
            // 异步发送邮件通知（模拟）
            log.info("正在发送邮件通知给用户：{}, 内容：{}", post.getUserId(), message);
            
            // 异步发送站内信（模拟）
            log.info("正在发送站内信给用户：{}, 内容：{}", post.getUserId(), message);
            
            // 如果是置顶帖子，额外通知管理员
            if (post.getIsPinned() == 1) {
                log.info("发送管理员通知：置顶新帖 - {}", post.getTitle());
            }
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("发送通知失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async
    public CompletableFuture<Void> sendCommentNotification(Long postId, Long commentUserId, String message) {
        try {
            // 获取帖子作者
            Post post = postService.getById(postId);
            if (post == null) {
                throw new RuntimeException("帖子不存在");
            }
            
            // 如果评论者不是帖子作者，通知帖子作者
            if (!post.getUserId().equals(commentUserId)) {
                log.info("通知帖子作者：{} 收到新评论", post.getUserId());
            }
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("发送评论通知失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async
    public CompletableFuture<Void> sendPurchaseNotification(Long postId, Long buyerId) {
        try {
            // 获取帖子信息
            Post post = postService.getById(postId);
            if (post == null) {
                throw new RuntimeException("帖子不存在");
            }
            
            // 通知卖家
            log.info("通知卖家：{} 有人购买了你的帖子《{}》", post.getUserId(), post.getTitle());
            
            // 通知买家
            log.info("通知买家：{} 购买成功！开始学习吧", buyerId);
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("发送购买通知失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

- [ ] **Step 3: 创建异步统计分析服务**

```java
package com.aistudy.service.async;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Post;
import com.aistudy.entity.User;
import com.aistudy.mapper.PostMapper;
import com.aistudy.mapper.UserMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

@Slf4j
@Service
public class AsyncAnalyticsService {
    
    @Autowired
    private PostMapper postMapper;
    
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Async
    public CompletableFuture<Void> analyzePostTrend(Long postId) {
        try {
            log.info("开始分析帖子趋势：{}", postId);
            
            // 模拟耗时分析
            Thread.sleep(1000);
            
            // 获取帖子数据
            Post post = postMapper.selectById(postId);
            if (post == null) {
                throw new RuntimeException("帖子不存在");
            }
            
            // 计算热度指数
            double heatIndex = post.getViewCount() * 0.3 + 
                              post.getLikeCount() * 0.5 + 
                              post.getCommentCount() * 0.2;
            
            // 缓存热度指数
            redisTemplate.opsForValue().set("post:heat:" + postId, heatIndex, 24, TimeUnit.HOURS);
            
            // 更新热门帖子榜单
            updateHotPosts();
            
            log.info("帖子 {} 热度分析完成，指数：{}", postId, heatIndex);
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("帖子趋势分析失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async
    public CompletableFuture<Void> updateUserActivity(Long userId) {
        try {
            log.info("更新用户活跃度：{}", userId);
            
            // 获取用户数据
            User user = userMapper.selectById(userId);
            if (user == null) {
                throw new RuntimeException("用户不存在");
            }
            
            // 计算活跃度分数
            int activityScore = user.getPoints() + user.getSignInCount() * 5;
            
            // 缓存活跃度
            redisTemplate.opsForValue().set("user:activity:" + userId, activityScore, 12, TimeUnit.HOURS);
            
            // 更新排行榜
            updateUserRanking();
            
            log.info("用户 {} 活跃度更新完成，分数：{}", userId, activityScore);
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("用户活跃度更新失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    private void updateHotPosts() {
        // 更新热门帖子榜单逻辑
        log.info("更新热门帖子榜单");
    }
    
    private void updateUserRanking() {
        // 更新用户排行榜逻辑
        log.info("更新用户排行榜");
    }
}
```

- [ ] **Step 4: 创建异步推荐服务**

```java
package com.aistudy.service.async;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.aistudy.entity.Post;
import com.aistudy.mapper.PostMapper;
import com.aistudy.service.PostService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

@Slf4j
@Service
public class AsyncRecommendationService {
    
    @Autowired
    private PostService postService;
    
    @Autowired
    private PostMapper postMapper;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Async
    public CompletableFuture<List<Post>> generateRecommendationsAsync(Long userId) {
        try {
            log.info("为用户 {} 生成个性化推荐", userId);
            
            // 模拟推荐计算
            Thread.sleep(500);
            
            // 基于用户行为生成推荐（简化版）
            List<Post> recommendations = postMapper.selectList(null)
                .stream()
                .limit(10)
                .toList();
            
            // 缓存推荐结果
            redisTemplate.opsForValue().set(
                "user:recommend:" + userId, 
                recommendations, 
                1, 
                TimeUnit.HOURS
            );
            
            log.info("用户 {} 推荐生成完成，推荐数量：{}", userId, recommendations.size());
            
            return CompletableFuture.completedFuture(recommendations);
        } catch (Exception e) {
            log.error("生成推荐失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async
    public CompletableFuture<List<Post>> getHotPostsAsync(Long userId) {
        try {
            log.info("获取热门帖子推荐");
            
            Page<Post> hotPosts = postService.getHotPosts(new Page<>(1, 8));
            List<Post> posts = hotPosts.getRecords();
            
            // 缓存热门帖子
            redisTemplate.opsForValue().set(
                "posts:hot", 
                posts, 
                30, 
                TimeUnit.MINUTES
            );
            
            return CompletableFuture.completedFuture(posts);
        } catch (Exception e) {
            log.error("获取热门帖子失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

- [ ] **Step 5: 创建异步文件处理器**

```java
package com.aistudy.service.async;

import com.aistudy.service.FileUploadService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.util.concurrent.CompletableFuture;

@Slf4j
@Service
public class AsyncFileProcessor {
    
    @Autowired
    private FileUploadService fileUploadService;
    
    @Async
    public CompletableFuture<String> processImageAsync(MultipartFile file, String filename) {
        try {
            log.info("开始处理图片：{}", filename);
            
            // 异步生成不同尺寸的图片
            String originalPath = fileUploadService.saveAvatar(file, filename);
            
            log.info("图片处理完成：{}", filename);
            
            return CompletableFuture.completedFuture(originalPath);
        } catch (Exception e) {
            log.error("图片处理失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async
    public CompletableFuture<Void> cleanupTempFiles() {
        try {
            log.info("开始清理临时文件");
            
            // 模拟清理过程
            Thread.sleep(1000);
            
            log.info("临时文件清理完成");
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("清理临时文件失败", e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

- [ ] **Step 6: 创建异步监控切面**

```java
package com.aistudy.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import java.util.concurrent.CompletableFuture;

@Slf4j
@Aspect
@Component
public class AsyncMonitorAspect {
    
    @Around("@annotation(org.springframework.scheduling.annotation.Async)")
    public Object monitorAsyncMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().getName();
        
        log.info("异步方法 {} 开始执行", methodName);
        
        try {
            Object result = joinPoint.proceed();
            long duration = System.currentTimeMillis() - startTime;
            
            if (result instanceof CompletableFuture) {
                CompletableFuture<?> future = (CompletableFuture<?>) result;
                future.whenComplete((res, ex) -> {
                    if (ex != null) {
                        log.error("异步方法 {} 执行失败，耗时：{}ms", methodName, duration, ex);
                    } else {
                        log.info("异步方法 {} 执行完成，耗时：{}ms", methodName, duration);
                    }
                });
            } else {
                log.info("异步方法 {} 执行完成，耗时：{}ms", methodName, duration);
            }
            
            return result;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            log.error("异步方法 {} 执行异常，耗时：{}ms", methodName, duration, e);
            throw e;
        }
    }
}
```

- [ ] **Step 7: 创建异步异常处理器**

```java
package com.aistudy.exception;

import com.aistudy.common.R;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.util.concurrent.FailureCallback;

@Slf4j
@RestControllerAdvice
public class AsyncExceptionHandler {
    
    @ExceptionHandler(org.springframework.util.concurrent.FailureCallback.class)
    public R<String> handleAsyncFailure(FailureCallback callback) {
        log.error("异步任务执行失败", callback);
        return R.error("任务执行失败，请稍后重试");
    }
    
    @ExceptionHandler(java.util.concurrent.ExecutionException.class)
    public R<String> handleExecutionException(ExecutionException e) {
        log.error("异步任务执行异常", e.getCause());
        return R.error("任务执行失败：" + e.getMessage());
    }
    
    @ExceptionHandler(java.util.concurrent.TimeoutException.class)
    public R<String> handleTimeoutException(TimeoutException e) {
        log.error("异步任务超时", e);
        return R.error("任务执行超时，请稍后重试");
    }
    
    @ExceptionHandler(Exception.class)
    public R<String> handleAsyncException(Exception e) {
        log.error("异步任务未知异常", e);
        return R.error("系统错误，请稍后重试");
    }
}
```

- [ ] **Step 8: 在Controller中使用异步服务**

```java
@RestController
@RequestMapping("/api/posts")
@CrossOrigin(origins = "http://localhost:5173")
public class PostController {
    
    @Autowired
    private AsyncNotificationService notificationService;
    
    @Autowired
    private AsyncAnalyticsService analyticsService;
    
    @Autowired
    private AsyncRecommendationService recommendationService;
    
    @PostMapping("/create")
    public R<String> createPost(@Valid @RequestBody Post post) {
        // 同步保存帖子
        postService.createPost(post);
        
        // 异步执行通知
        notificationService.sendPostNotification(post.getId(), "新帖子已发布");
        
        // 异步分析趋势
        analyticsService.analyzePostTrend(post.getId());
        
        // 异步生成推荐
        recommendationService.getHotPostsAsync(getCurrentUserId());
        
        return R.success("发帖成功");
    }
    
    @GetMapping("/recommend")
    public R<List<Post>> getRecommendations(@RequestParam Long userId) {
        CompletableFuture<List<Post>> recommendations = 
            recommendationService.generateRecommendationsAsync(userId);
        
        // 等待结果（生产环境可以使用回调）
        List<Post> posts = recommendations.join();
        return R.success(posts);
    }
}
```

- [ ] **Step 9: Commit**

```bash
git add .
git commit -m "feat: 添加异步和多线程优化"
```

### Task 6: 前端页面开发

**Files:**
- Create: `frontend/src/router/index.js`
- Create: `frontend/src/store/user.js`
- Create: `frontend/src/store/post.js`
- Create: `frontend/src/views/Login.vue`
- Create: `frontend/src/Register.vue`
- Create: `frontend/src/views/Home.vue`
- Create: `frontend/src/views/PostList.vue`
- Create: `frontend/src/views/PostDetail.vue`
- Create: `frontend/src/views/CreatePost.vue`
- Create: `frontend/src/views/Ranking.vue`
- Create: `frontend/src/components/PostCard.vue`
- Create: `frontend/src/components/CommentItem.vue`

（这部分任务将在前端部分详细实现）

### Task 7: 部署配置

**Files:**
- Create: `docker-compose.yml`
- Create: `backend/Dockerfile`
- Create: `frontend/Dockerfile`
- Create: `nginx.conf`

（这部分任务将在部署部分详细实现）