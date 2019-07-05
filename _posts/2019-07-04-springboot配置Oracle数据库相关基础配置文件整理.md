---
layout:     post
title:      springboot配置Oracle数据库相关基础配置文件整理
subtitle:   springboot相关配置----Oracle
date:       2019-07-04
author:     lwj108
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - java
    - code
    - oracle
---
## 1.pom.xml引入对应数据文件
```xml
            <dependency>
                <groupId>com.oracle</groupId>
                <artifactId>ojdbc14</artifactId>
                <version>11.2.0.1.0</version>
                <scope>runtime</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>1.1.0</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.starter}</version>
            </dependency>
```

## 2.yml配置数据库连接信息
```yml
main:
    allow-bean-definition-overriding: true
  mybatis:
    typeAliasesPackage: com.supwisdom.platform.portal.kafka.domain
    mapperLocations: classpath*:mapper/*.xml
    dialect: oracle
  exception:
    restErrorResolver:
      defaultExCode: 400
      defaultExMsg: 未知原因，访问服务器出错
      exceptionMappingDefinitions:
        com.supwisdom.platform.core.framework.exception.SignificantRestException: _exmsg
        com.supwisdom.platform.portal.api.exception.ValidateException: _exmsg
        com.supwisdom.platform.core.framework.exception.RestException: 400, common.unknownException
        com.supwisdom.platform.core.framework.exception.ManagerException: 400, common.unknownException
        Throwable: 400
    defaultMessageSource:
      basename: classpath:exception/messages
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: oracle.jdbc.OracleDriver
    url:  jdbc:oracle:thin:@${JDBC_URL:192.168.1.190:1521:dev}
    username: ${JDBC_USERNAME:platform_portal_service}
    password: ${JDBC_PASSWORD:kingstar}
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 6000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    filters: stat,wall,log4j
    logSlowSql: true
```
## 3.Configuration配置

* DruidConfig配置
```java
import java.sql.SQLException;

import javax.sql.DataSource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;

@Configuration
@ConfigurationProperties(prefix = "spring.datasource")
public class DruidConfig {

    private Logger logger = LoggerFactory.getLogger(DruidConfig.class);

    private String url;

    private String username;

    private String password;

    private String driverClassName;

    private int initialSize;

    private int minIdle;

    private int maxActive;

    private int maxWait;

    private String validationQuery;

    private boolean testWhileIdle;

    private String filters;

    private String logSlowSql;

    @Bean
    public ServletRegistrationBean druidServlet() {
        ServletRegistrationBean reg = new ServletRegistrationBean();
        reg.setServlet(new StatViewServlet());
        reg.addUrlMappings("/druid/*");
        reg.addInitParameter("loginUsername", username);
        reg.addInitParameter("loginPassword", password);
        reg.addInitParameter("logSlowSql", logSlowSql);
        return reg;
    }

    @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new WebStatFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        filterRegistrationBean.addInitParameter("profileEnable", "true");
        return filterRegistrationBean;
    }

    @Bean
    public DataSource DruidDataSource() {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(url);
        datasource.setUsername(username);
        datasource.setPassword(password);
        datasource.setDriverClassName(driverClassName);
        datasource.setInitialSize(initialSize);
        datasource.setMinIdle(minIdle);
        datasource.setMaxActive(maxActive);
        datasource.setMaxWait(maxWait);
        datasource.setValidationQuery(validationQuery);
        datasource.setTestWhileIdle(testWhileIdle);
        try {
            datasource.setFilters(filters);
        } catch (SQLException e) {
            logger.error("druid configuration initialization filter", e);
        }
        return datasource;
    }

    public Logger getLogger() {
        return logger;
    }

    public void setLogger(Logger logger) {
        this.logger = logger;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getDriverClassName() {
        return driverClassName;
    }

    public void setDriverClassName(String driverClassName) {
        this.driverClassName = driverClassName;
    }

    public int getInitialSize() {
        return initialSize;
    }

    public void setInitialSize(int initialSize) {
        this.initialSize = initialSize;
    }

    public int getMinIdle() {
        return minIdle;
    }

    public void setMinIdle(int minIdle) {
        this.minIdle = minIdle;
    }

    public int getMaxActive() {
        return maxActive;
    }

    public void setMaxActive(int maxActive) {
        this.maxActive = maxActive;
    }

    public int getMaxWait() {
        return maxWait;
    }

    public void setMaxWait(int maxWait) {
        this.maxWait = maxWait;
    }

    public String getValidationQuery() {
        return validationQuery;
    }

    public void setValidationQuery(String validationQuery) {
        this.validationQuery = validationQuery;
    }

    public boolean isTestWhileIdle() {
        return testWhileIdle;
    }

    public void setTestWhileIdle(boolean testWhileIdle) {
        this.testWhileIdle = testWhileIdle;
    }

    public String getFilters() {
        return filters;
    }

    public void setFilters(String filters) {
        this.filters = filters;
    }

    public String getLogSlowSql() {
        return logSlowSql;
    }

    public void setLogSlowSql(String logSlowSql) {
        this.logSlowSql = logSlowSql;
    }

}
```

* MyBatisConfig配置
```java
import java.util.Properties;

import javax.annotation.Resource;
import javax.sql.DataSource;

import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.TransactionManagementConfigurer;

import com.supwisdom.platform.portal.service.framework.interceptor.PageInterceptor;

@Configuration
public class MyBatisConfig implements TransactionManagementConfigurer {

    @Autowired
    private DataSource dataSource;

    @Value("${spring.mybatis.typeAliasesPackage}")
    private String typeAliasesPackage;
    
    @Value("${spring.mybatis.mapperLocations}")
    private String mapperLocations;
    
    @Value("${spring.mybatis.dialect}")
    private String dialect;
    
    
    @Bean(name = "sqlSessionFactory") // 3
    public SqlSessionFactory sqlSessionFactoryBean() {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        //设置datasource
        bean.setDataSource(dataSource);
        //设置typeAlias 包扫描路径   
        bean.setTypeAliasesPackage(typeAliasesPackage);

        Properties properties = new Properties();
        properties.setProperty("databaseType", dialect);
        PageInterceptor pageInterceptor = new PageInterceptor();
        pageInterceptor.setProperties(properties);

        Interceptor[] plugins = new Interceptor[] { pageInterceptor };
        bean.setPlugins(plugins);

        // 添加XML目录
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        try {
            //添加mapper 扫描路径 
            bean.setMapperLocations(resolver.getResources(mapperLocations));
            return bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Override
    @Bean
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        return new DataSourceTransactionManager(dataSource);
    }

}
```