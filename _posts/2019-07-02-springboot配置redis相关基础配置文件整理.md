---
layout:     post
title:      springboot配置redis相关基础配置文件整理
subtitle:   springboot相关配置----redis
date:       2019-07-02
author:     lwj108
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - java
    - code
    - redis
---
## 1.pom.xml引入对应数据文件
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
```

## 2.yml配置redis连接信息
```yml
redis:
    database: 0  
    port: 7000
    jedis:
      pool:
        max-idle: 20
        min-idle: 2
        max-active: 50
        max-wait: 3000
    host: 192.168.1.140
    timeout: 5000
```

## 3.RedisConfig配置
* RedisConfig
```java
import java.net.UnknownHostException;
import java.util.List;
import java.util.Map;

import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig extends CachingConfigurerSupport {
//    @Bean
//    public RedisTemplate<Object,SendParameter> empRedisTemplate(RedisConnectionFactory redisConnectionFactory)
//        throws UnknownHostException{
//        RedisTemplate<Object,SendParameter> template = new RedisTemplate<Object, SendParameter>();
//        template.setConnectionFactory(redisConnectionFactory);
//        Jackson2JsonRedisSerializer<SendParameter> ser = new Jackson2JsonRedisSerializer<SendParameter>(SendParameter.class);
//        template.setDefaultSerializer(ser);
//        return template;
//    }
    
    @Bean
    public RedisTemplate<String, List<Map<String, Object>>> redisTemplate(RedisConnectionFactory redisConnectionFactory) {

        RedisTemplate<String, List<Map<String, Object>>> template = new RedisTemplate<String, List<Map<String, Object>>>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new RedisObjectSerializer());
        return template;
    }
}
```
* RedisObjectSerializer
```java
import org.springframework.core.convert.converter.Converter;
import org.springframework.core.serializer.support.DeserializingConverter;
import org.springframework.core.serializer.support.SerializingConverter;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;

public class RedisObjectSerializer implements RedisSerializer<Object> {
    private Converter<Object, byte[]> serializer = new SerializingConverter();
    private Converter<byte[], Object> deserializer = new DeserializingConverter();
    static final byte[] EMPTY_ARRAY = new byte[0];

    @Override
    public Object deserialize(byte[] bytes) {
        if (isEmpty(bytes)) {
            return null;
        }
        try {
            return deserializer.convert(bytes);
        } catch (Exception ex) {
            throw new SerializationException("Cannot deserialize", ex);
        }
    }

    @Override
    public byte[] serialize(Object object) {
        if (object == null) {
            return EMPTY_ARRAY;
        }
        try {
            return serializer.convert(object);
        } catch (Exception ex) {
            return EMPTY_ARRAY;
        }
    }

    private boolean isEmpty(byte[] data) {
        return (data == null || data.length == 0);
    }
}

```