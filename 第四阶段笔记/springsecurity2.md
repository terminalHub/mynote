# Security的授权_辉哥

## 1. 授权相关概念

### 1.1. 什么是授权

```properties
授权： 表示什么样的用户能访问什么样的资源，就是访问资源时 判断是否有这个权限。

用户授权： 则是根据系统提前设置好规则，给用户分配可以访问的权限，用户根据自己的权限，去执行响应的操作，当		   用户操作没有权限的资源时 告知用户 没有权限
```

### 1.2. 授权解决方案

```properties
1： 基于过滤器的权限管理（FilterSecurityInterceptor） 

2： 基于方法拦截器（aop）的权限管理(MethodSecurityInterceptor)
```

#### 1.2.1. 基于过滤器

```properties
这种方式主要是用于拦截http请求，根据http请求路径进行权限校验 

1： 当用户请求过来之后 经过FilterSecurityInterceptor，在这个过滤器中判断当前用户发送的这个请求路径到底有没有权限，如果有权限则代码继续向下走，执行controller 然后返回数据 返回数据 此时再走这个过滤器，在这个后置处理中 做一些收尾工作，
```

![](images/QQ截图20210714134557.png)

#### 1.2.2. 基于方法拦截器

```properties
基于方法拦截器MethodSecurityInterceptor 表示请求请求到某一个方法时 使用AOP机制，在指定方法之前判断
是否有这个权限，如果有的话 再去执行对应的方法，执行完方法之后，然后在AOP后置处理器中做收尾工作 
```

![](images/QQ截图20210714134945.png)

### 1.3. 授权相关类说明

#### 1.3.1. AccessDecisionManager

```properties
这个类表示决策器 用于决定这个用户到底有没有访问这个资源的权限  ，这个类中同时管理多个投票器，这些投屏器
是真正判断是否有权限的，各个投票器投票 是有这个类发起的  ，然后各个投票器针对这次访问进行投票，由于投票器有多个，那么投票的结果可能不一致，比如：有的投票器投票的结果是有权限，有的投票器投票的结果是没权限，那么最终结果到底听说的？ 此时就需要这个决策器 

实现类： 
	1：AffirmativeBased （默认值） 表示一票通过  只要有一个投票器认为有权限 那就是有权限 
	2：ConsensusBased： 表示少数服从多数，如果票数一致 并且必须的有一个同意的票 此时就按照同意
					  此时还受allowIfEqualGrantedDeniedDecisions变量的影响
					  
    3： UnanimousBased：表示一票否决制 只要有一个不同意的票 则就表示没有权限 
	
```

#### 1.3.2. AccessDecisionVoter

```properties
这个接口 表示真正投票的基类  这个类中维护了3个变量当做投票的结果  
```

![](images/QQ截图20210714141213.png)

**实现类**

![](images/QQ截图20210714142233.png)

#### 1.3.3. ConfigAttribute

```properties
这个接口字面意思就是配置属性，但是实际上时保存对应请求的相关权限 把这个权限表达式字符串封装到了这个对象身上 
```

![](images/QQ截图20210714211212.png)

#### 1.3.4. SecurityMetadataSource

```properties
整个类封装了访问资源需要的权限信息，里面就是存的集合集合中保存的就是ConfigAttribute,当访问资源时，用户所拥有的权限 和 访问这个资源所需要的权限做对比  用户的权限在Authentication对象中  而对比的权限就在这个对象中  ，可以认为是权限数据源 
```

![](images/QQ截图20210714212119.png)

#### 1.3.5. AfterInvocationManager

```properties
这个类表示权限认证通过之后 目标方法执行之后，后续处理的事情 
```

![](images/QQ截图20210714212636.png)

#### 1.3.6. AfterInvocationProvider

```properties
这个类表示 真正做后续处理的事情的提供者  有点类似 认证提供者 和投票器一样 
```

#### 1.3.7. AccessDeniedHandler 

```properties
这个表示授权失败 或者说访问某个资源时 权限不够的处理  
```

![](images/QQ截图20210715232159.png)

#### 1.3.8. securityExpressionOperations 

```properties
这个接口和这个接口的实现类定义了一些权限表达式  
```

| 权限表达式           | 作用                                                    |
| -------------------- | ------------------------------------------------------- |
| hasAuthority         | 是否有这个权限                                          |
| hasAnyAuthority      | 是否有多个权限中的其中一个                              |
| hasRole              | 是否有这个角色                                          |
| hasAnyRole           | 是否有多个角色中的一个                                  |
| permitAll            | 允许所有的请求访问                                      |
| denyAll              | 拒绝所有的请求访问                                      |
| isAnonymous          | 当前用户是否是匿名用户                                  |
| isAuthenticated      | 当前用户是否已经认证                                    |
| isRememberMe         | 当前用户是否使用rememberMe自动认证的用户                |
| isFullyAuthenticated | 当前用户既不是匿名用户 又不是通过Remember自动登录的用户 |
| hasPermission        | 当前用户是否有指定的权限                                |
| hasIpAddress         | 当前用户指定的ip是否为指定的ip                          |

## 2. 基于过滤器（URL）的权限拦截

### 2.1. 环境搭建

#### 2.1.1. 新建项目

![](images/QQ截图20210714222155.png)

#### 2.1.2. 编写主配置类

![](images/QQ截图20210714222405.png)

#### 2.1.3. 编写controller

![](images/QQ截图20210714222444.png)

#### 2.1.4. security的配置类 

![](images/QQ截图20210715215340.png)

#### 2.1.5. 配置文件

![](images/QQ截图20210715215412.png)

#### 2.1.6. 测试 

```properties
有权限将会正常访问 
没有权限将会返回状态码403
```

### 2.2. 原理分析

#### 2.2.1. 权限问题说明

```properties
在案例中 我们通过了角色进行了权限管理，也通过权限进行的权限管理，最终的权限都封装到Authentication对象身上   这个对象中的 getAuthorities（）方法 返回用户所有的权限，那么 返回的权限到底是角色 还是权限？ 其实不管是角色还是权限底层都会当成权限处理 不管设置什么 最后调用的方法是同一个 Authentication中的权限信息来源于用户 而用户则是UserDetails  在User中 不管设置角色 还是设置权限 都是会封装到UserDetails对象的权限中 
只是角色多了一个ROLE_的前缀 权限没有前缀而已，所以在编码层面 角色和权限最终是一个东西 
```

![](images/QQ截图20210703150209.png)

#### 2.2.2. 项目流程分析

* **编写配置文件**

  ![](images/QQ截图20210715220903.png)

* **返回表达式**

  ![](images/QQ截图20210715221014.png)

* **参数是表达式字符串**

  ![](images/QQ截图20210715221128.png)

* **封装表达式字符串**

  ![](images/QQ截图20210715221508.png)

* **保存请求和这个请求的权限**

  ![](images/QQ截图20210715221814.png)

* **父类AbstractInterceptUrlConfigurer config方法中**

  ![](images/QQ截图20210715222534.png)

* **子类重写**

  ![](images/QQ截图20210715223153.png)

* **把请求和请求权限和SPEL处理器封装到对象中**

  ![](images/QQ截图20210715224522.png)

* **添加过滤器**

  ![](images/QQ截图20210715224641.png)

* **登录完成之后用户访问**

  ![](images/QQ截图20210715224944.png)

* **拦截器FilterSecurityInterceptor拦截请求**

  ![](images/QQ截图20210715225138.png)

* **beforeInvocation方法**

  ![](images/QQ截图20210715225400.png)

  ![](images/QQ截图20210715225501.png)

  ![](images/QQ截图20210715225759.png)

* ***

  **投票失败 抛出异常  ExceptionTranslationFilter 处理异常**

  ![](images/QQ截图20210715231516.png)

* **AccessDeniedHandlerImpl**

  ![](images/QQ截图20210715231746.png)

#### 2.2.3. 流程总结

```properties
1： 项目启动时 把我们配置的请求路径和这个路径所需要的权限经过一些列的转换 最终把这个对应关系存储到了SecurityMetadataSource数据源身上，并且把这个数据源传递给了最后一个过滤器FilterSecurityInterceptor

2： 登录访问时 把登录的信息存到了Authentication 对象中 然后请求被FilterSecurityInterceptor拦截 

3： 拿到用户认证信息和FilterSecurityInterceptor中的SecurityMetadataSource存储的这个资源所需要的权限
	交给决策器 决策器调用投票器进行投票  默认一票通过值 如果投票成功 则有权限 继续访问controller  

4： 如果投票失败（没有这个权限） 则被ExceptionTranslationFilter拦截处理异常，

5： 调用AccessDeniedHandlerImpl中的handler方法 要嘛转发到错误页面 要嘛返回403错误
```

### 2.3. 项目改造

![](images/QQ截图20210715233907.png)

### 2.4. 自定义表达式

```properties
如果自带的表达式不能满足需求 我们可以自定义 比如自带的这些表达式只能对某一个URL或者某一些URL进行权限拦截  
如果在某些场景下 我们需要根据不同的参数进行权限管理 比如当你传递huige的时候 有权限 传的不是huige就没有权限 此时我们需要自定义权限表达式 
```

#### 2.4.1. 定义controller

![](images/QQ截图20210715235425.png)

#### 2.4.2. 自定义表达式

![](images/QQ截图20210715235455.png)

#### 2.4.3. 配置信息

![](images/QQ截图20210715235535.png)

### 2.5. 自定义错误响应

```properties
默认情况下 返回错误403 但是可以自定义
```

#### 2.5.1. 指定错误页面

![](images/QQ截图20210716000025.png)

#### 2.5.2. 返回json方式一

* **定义controller**

  ![](images/QQ截图20210716000340.png)

* **配置**

  ![](images/QQ截图20210716000405.png)

#### 2.5.3. 返回json方式二 

* **自定义handler**

  ![](images/QQ截图20210716000758.png)

* **配置**

  ![](images/QQ截图20210716000830.png)


### 2.6. 数据库用户

```properties
上面的案例中，我们的用户来源于内存 并且角色也配置死了 实际开发中 我们的用户 权限应该来自于数据库 
```

#### 2.6.1. 建库建表 

```mysql
## 新建表
CREATE TABLE `admin`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `admin_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `admin_password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
#############################################################
CREATE TABLE `role`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `role_desc` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

##################################################################
CREATE TABLE `perm`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `perm_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `perm_url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;


#######################
CREATE TABLE `admin_role`  (
  `admin_id` bigint(20) NOT NULL,
  `role_id` bigint(20) NOT NULL,
  PRIMARY KEY (`admin_id`, `role_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;


CREATE TABLE `role_perm`  (
  `role_id` bigint(20) NOT NULL,
  `perm_id` bigint(20) NOT NULL,
  PRIMARY KEY (`role_id`, `perm_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

```

#### 2.6.2. 插入数据

```mysql
INSERT INTO `admin` VALUES (1, 'huige1', '123456');
INSERT INTO `admin` VALUES (2, 'huige2', '123456');
INSERT INTO `admin` VALUES (3, 'huige3', '123456');
INSERT INTO `admin` VALUES (4, 'huige4', '123456');


INSERT INTO `role` VALUES (1, 'ROLE_demo1', 'ROLE_demo1');
INSERT INTO `role` VALUES (2, 'ROLE_demo2', 'ROLE_demo2');
INSERT INTO `role` VALUES (3, 'ROLE_demo3', 'ROLE_demo3');
INSERT INTO `role` VALUES (4, 'ROLE_demo4', 'ROLE_demo4');


INSERT INTO `perm` VALUES (1, 'demo1', '/demo1');
INSERT INTO `perm` VALUES (2, 'demo2', '/demo2');
INSERT INTO `perm` VALUES (3, 'demo3', '/demo3');
INSERT INTO `perm` VALUES (4, 'demo4', '/demo4');

INSERT INTO `admin_role` VALUES (1, 1);
INSERT INTO `admin_role` VALUES (2, 2);
INSERT INTO `admin_role` VALUES (3, 3);
INSERT INTO `admin_role` VALUES (4, 4);

INSERT INTO `role_perm` VALUES (1, 1);
INSERT INTO `role_perm` VALUES (2, 2);
INSERT INTO `role_perm` VALUES (3, 3);
INSERT INTO `role_perm` VALUES (4, 4);
```

#### 2.6.3. 新建工程

![](images/QQ截图20210716173456.png)

#### 2.6.4. 添加依赖

![](images/QQ截图20210716173456.png)

#### 2.6.5. 主配置类

![](images/QQ截图20210716173609.png)

#### 2.6.6. 配置文件

![](images/QQ截图20210716173636.png)

#### 2.6.7. 实体类

![](images/QQ截图20210716173748.png)

![](images/QQ截图20210716173800.png)

![](images/QQ截图20210716173814.png)

#### 2.6.7. 定义Mapper 

![](images/QQ截图20210716173843.png)

![](images/QQ截图20210716173903.png)

#### 2.6.8. 定义adminService

![](images/QQ截图20210716173948.png)

#### 2.6.9. 定义controller

![](images/QQ截图20210716174059.png)

#### 2.6.10. 定义metaSource

![](images/QQ截图20210716174155.png)

#### 2.6.11. 配置密码加密

![](images/QQ截图20210716174251.png)



#### 2.6.12. 实现方式思想

```properties
动态权限的实现方式有很多种  默认的情况下 我们的认证信息中的权限是ROLE_admin，但是SecurityMetaSource默认使用的是ExpressionBasedFilterInvocationSecurityMetadataSource  里面存储的是 hasRole('admin')表达式  比较的时候 默认调用 WebExpressionVoter投票器中的方法 在这个方法中使用ExpressionUtils 表达式工具类进行比较的  而现在我们认证信息中存储的是ROLE_demo1  和  SecurityMetaSource中存储的内容是一值的 所以默认的情况下 WebExpressionVoter比较方式是不适合的   


解决思想：
	1： 自定义投票器(代码太多)
	2： 直接在决策器中判断，直接不让走投票器  
	3： 直接不使用默认 直接彻底换掉所有的 （依赖RoleVoter）

```

#### 2.6.13. 方式一（全部换掉） 

```properties
默认情况下 使用的是ExpressionUrlAuthorizationConfigurer   在这个configurer中配置过滤器和投票器 
我们可以直接从根源换掉 把这个配置换掉 换成它的另外一个兄弟 UrlAuthorizationConfigurer
```

![](images/QQ截图20210716175721.png)

#### 2.6.14. 方式二（自定义决策器）

* **自定义决策器**

  ![](images/QQ截图20210716182501.png)

* **配置**

  ![](images/QQ截图20210716182539.png)

```properties
这种写法的好处 角色可以不加任何前缀 向写啥写啥
```

## 3. 基于AOP方法拦截 

### 3.1. 注解相关

```properties
SpringBoot使用AOP进行方法级别拦截主要依赖注解 SpringSecurity提供提供了大量的注解来完成方法级别权限拦截 
 
 @PerFilter： 表示请求目标方法之前对参数进行过滤 
 @PerAuthorize ： 表示请求目标方法之前进行权限校验 支持权限表达式 
 @PostFilter： 表示目标方法执行之后 对返回值过滤
 @PostAuthorize： 表示目标方法执行之后 对权限校验 主要用于ACL权限模型（了解）
 
 -----------------------以上四个需要使用perPostEnabled开启-------------------------
 
 
 @Secured： 表示执行某个方法需要拥有什么样的角色 不支持权限表达式 
 
 ---------------------------@Secured 需要使用securedEnabled开启---------------------
 
 @DenyAll： 表示拒绝所有访问 
 @PermitAll 表示允许所有访问  
 
 -----------------------以上2个注解需要使用jsr250Enabled开启-------------------------
```

### 3.2. 基本使用

#### 3.2.1. 新建项目

![](images/QQ截图20210716195127.png)

#### 3.2.2. 添加主配置类

![](images/QQ截图20210716195152.png)

#### 3.2.3. 编写配置文件

![](images/QQ截图20210716195212.png)

#### 3.2.4. 编写controller

![](images/QQ截图20210716195243.png)

#### 3.2.5. security的配置文件

![](images/QQ截图20210716195319.png)

### 3.3. 源码分析

#### 3.3.1.  EnableGlobalMethodSecurity

![](images/QQ截图20210717121237.png)

#### 3.3.2.  GlobalMethodSecuritySelector

![](images/QQ截图20210717121416.png)

#### 3.3.3. GlobalMethodSecurityConfiguration

![](images/QQ截图20210717121924.png)

![](images/QQ截图20210717122005.png)

![](images/QQ截图20210717122114.png)

![](images/QQ截图20210717122244.png)

#### 3.3.4. Jsr250MetadataSourceConfiguration

![](images/QQ截图20210717122436.png)

#### 3.3.5. MethodSecurityMetadataSourceAdvisorRegistrar 

![](images/QQ截图20210717122658.png)

#### 3.3.6. MethodSecurityMetadataSourceAdvisor 

![](images/QQ截图20210717123620.png)

#### 3.3.7. 流程描述

```properties
1： 使用@EnableGlobalMethodSecurity(prePostEnabled = true， securedEnabled = true,jsr250Enabled 	= true) 开始方法校验  在配置类中导入了GlobalMethodSecuritySelector 向容器中批量添加Bean

2： 在GlobalMethodSecuritySelector类中 根据EnableGlobalMethodSecurity注解不同的配置向容器中添加组件
	
	
3： 同时设置设置过滤器 设置决策器 设置投票器 设置权限数据源  

4： 浏览器发送请求 被过滤器拦截  被决策器决策  决策器调用投票器 投票器获取数据源中的权限 和认证信息中的	权限对比
```

### 3.4. 数据库用户（权限是角色） 

#### 3.4.1.  建库建表

```properties
还用上个数据库
```

#### 3.4.2. 新建工程

![](images/QQ截图20210717134954.png)

#### 2.4.3. 添加依赖

![](images/QQ截图20210717135047.png)

#### 2.4.4. 主配置类

![](images/QQ截图20210717135111.png)

#### 2.4.5. 配置文件

![](images/QQ截图20210717135132.png)

#### 2.4.6. 实体类

```properties
和上个实体类一致 
```

#### 2.4.7. 定义Mapper 

![](images/QQ截图20210717135231.png)

#### 2.4.8. 定义service

![](images/QQ截图20210717135300.png)

#### 2.4.9. security的配置

![](images/QQ截图20210717135339.png)

#### 2.4.9. 问题说明

```properties
这种写法是可以的 但是好像角色和权限表没有什么用处了  只是用户和角色 和 方法中的指定的字符串进行匹配而已
```

### 3.5. 数据库用户（权限是标识）

#### 3.5.1. 数据库添加列(标识)

![](images/QQ截图20210717135632.png)

#### 3.5.2. 修改controller

![](images/QQ截图20210717135801.png)

#### 3.5.3. 修改Admin

![](images/QQ截图20210717135937.png)

#### 3.5.4. 修改Perm类

![](images/QQ截图20210717140014.png)

#### 3.5.5. 修改Mapper

![](images/QQ截图20210717141008.png)

#### 3.5.6. 修改service 

![](images/QQ截图20210717141630.png)

### 3.6. 数据库用户（自定义表达式）

#### 3.6.1. 自定义表达式

![](images/QQ截图20210717142253.png)

#### 3.6.2. 使用表达式

![](images/QQ截图20210717142331.png)