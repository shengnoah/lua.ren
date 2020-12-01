---
layout: post
title: Java并发：分布式应用限流 Redis + Lua 实践 
tags: [lua文章]
categories: [topic]
---
任何限流都不是漫无目的的，也不是一个开关就可以解决的问题，常用的限流算法有：令牌桶，漏桶。在之前的文章中，也讲到过，但是那是基于单机场景来写。

之前文章：[接口限流算法：漏桶算法&令牌桶算法](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452964796&idx=1&sn=281842ac2a970d67b3946c85196b6cf4&chksm=88ede8d4bf9a61c2b1a5a19759098f88e56d6d0958667af90cc99fec863bb47e4cf4d26366ea&token=633981981&lang=zh_CN#rd)

然而再牛逼的机器，再优化的设计，对于特殊场景我们也是要特殊处理的。就拿秒杀来说，可能会有百万级别的用户进行抢购，而商品数量远远小于用户数量。如果这些请求都进入队列或者查询缓存，对于最终结果没有任何意义，徒增后台华丽的数据。对此，为了减少资源浪费，减轻后端压力，我们还需要对秒杀进行限流，只需保障部分用户服务正常即可。

就秒杀接口来说，当访问频率或者并发请求超过其承受范围的时候，这时候我们就要考虑限流来保证接口的可用性，以防止非预期的请求对系统压力过大而引起的系统瘫痪。通常的策略就是拒绝多余的访问，或者让多余的访问排队等待服务。

**分布式限流**

单机限流，可以用到 `AtomicInteger`、`RateLimiter`、`Semaphore` 这些。但是在分布式中，就不能使用了。常用分布式限流用
`Nginx`
限流，但是它属于网关层面，不能解决所有问题，例如内部服务，短信接口，你无法保证消费方是否会做好限流控制，所以自己在应用层实现限流还是很有必要的。

本文不涉及 `Nginx + Lua`，简单介绍 `Redis +
Lua`分布式限流的实现。如果是需要在接入层限流的话，应该直接采用nginx自带的连接数限流模块和请求限流模块。

## Redis + Lua 限流示例

本次项目使用`SpringBoot 2.0.4`，使用到 `Redis` 集群，`Lua` 限流脚本

## 引入依赖

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    

|

    
    
    <dependencies>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-data-redis</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-aop</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.commons</groupId>  
            <artifactId>commons-lang3</artifactId>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
        </dependency>  
    </dependencies>  
      
  
---|---  
  
## Redis 配置

 **application.properties**

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    

|

    
    
    spring.application.name=spring-boot-limit  
      
    # Redis数据库索引  
    spring.redis.database=0  
    # Redis服务器地址  
    spring.redis.host=10.4.89.161  
    # Redis服务器连接端口  
    spring.redis.port=6379  
    # Redis服务器连接密码（默认为空）  
    spring.redis.password=  
    # 连接池最大连接数（使用负值表示没有限制）  
    spring.redis.jedis.pool.max-active=8  
    # 连接池最大阻塞等待时间（使用负值表示没有限制）  
    spring.redis.jedis.pool.max-wait=-1  
    # 连接池中的最大空闲连接  
    spring.redis.jedis.pool.max-idle=8  
    # 连接池中的最小空闲连接  
    spring.redis.jedis.pool.min-idle=0  
    # 连接超时时间（毫秒）  
    spring.redis.timeout=10000  
      
  
---|---  
  
## Lua 脚本

参考： [聊聊高并发系统之限流特技](http://jinnianshilongnian.iteye.com/blog/2305117)  
<http://jinnianshilongnian.iteye.com/blog/2305117>

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    

|

    
    
    local key = "rate.limit:" .. KEYS[1] --限流KEY  
    local limit = tonumber(ARGV[1])        --限流大小  
    local current = tonumber(redis.call('get', key) or "0")  
    if current + 1 > limit then --如果超出限流大小  
      return 0  
    else  --请求数+1，并设置2秒过期  
      redis.call("INCRBY", key,"1")  
       redis.call("expire", key,"2")  
       return current + 1  
    end  
      
  
---|---  
  
1、我们通过KEYS[1] 获取传入的key参数  
2、通过ARGV[1]获取传入的limit参数  
3、redis.call方法，从缓存中get和key相关的值，如果为nil那么就返回0  
4、接着判断缓存中记录的数值是否会大于限制大小，如果超出表示该被限流，返回0  
5、如果未超过，那么该key的缓存值+1，并设置过期时间为1秒钟以后，并返回缓存值+1

## 限流注解

注解的目的，是在需要限流的方法上使用

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    

|

    
    
    package com.souyunku.example.annotation;  
    /**  
     * 描述: 限流注解  
     *  
     * @author yanpenglei  
     * @create 2018-08-16 15:24  
     **/  
    @Target({ElementType.TYPE, ElementType.METHOD})  
    @Retention(RetentionPolicy.RUNTIME)  
    public @interface RateLimit {  
      
        /**  
         * 限流唯一标示  
         *  
         * @return  
         */  
        String key() default "";  
      
        /**  
         * 限流时间  
         *  
         * @return  
         */  
        int time();  
      
        /**  
         * 限流次数  
         *  
         * @return  
         */  
        int count();  
    }  
      
  
---|---  
  
## 公共配置

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    

|

    
    
    package com.souyunku.example.config;  
      
    @Component  
    public class Commons {  
      
        /**  
         * 读取限流脚本  
         *  
         * @return  
         */  
        @Bean  
        public DefaultRedisScript<Number> redisluaScript() {  
            DefaultRedisScript<Number> redisScript = new DefaultRedisScript<>();  
            redisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("rateLimit.lua")));  
            redisScript.setResultType(Number.class);  
            return redisScript;  
        }  
      
        /**  
         * RedisTemplate  
         *  
         * @return  
         */  
        @Bean  
        public RedisTemplate<String, Serializable> limitRedisTemplate(LettuceConnectionFactory redisConnectionFactory) {  
            RedisTemplate<String, Serializable> template = new RedisTemplate<String, Serializable>();  
            template.setKeySerializer(new StringRedisSerializer());  
            template.setValueSerializer(new GenericJackson2JsonRedisSerializer());  
            template.setConnectionFactory(redisConnectionFactory);  
            return template;  
        }  
      
    }  
      
  
---|---  
  
## 拦截器

通过拦截器 拦截`@RateLimit`注解的方法，使用`Redsi execute` 方法执行我们的限流脚本，判断是否超过限流次数

以下下是核心代码

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49  
    50  
    51  
    52  
    53  
    54  
    55  
    56  
    57  
    58  
    59  
    60  
    61  
    62  
    63  
    64  
    65  
    66  
    67  
    68  
    69  
    70  
    71  
    72  
    73  
    74  
    75  
    76  
    77  
    78  
    79  
    80  
    

|

    
    
    package com.souyunku.example.config;  
      
    /**  
     * 描述:拦截器  
     *  
     * @author yanpenglei  
     * @create 2018-08-16 15:33  
     **/  
    @Aspect  
    @Configuration  
    public class LimitAspect {  
      
        private static final Logger logger = LoggerFactory.getLogger(LimitAspect.class);  
      
        @Autowired  
        private RedisTemplate<String, Serializable> limitRedisTemplate;  
      
        @Autowired  
        private DefaultRedisScript<Number> redisluaScript;  
      
        @Around("execution(* com.souyunku.example.controller ..*(..) )")  
        public Object interceptor(ProceedingJoinPoint joinPoint) throws Throwable {  
      
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
            Method method = signature.getMethod();  
            Class<?> targetClass = method.getDeclaringClass();  
            RateLimit rateLimit = method.getAnnotation(RateLimit.class);  
      
            if (rateLimit != null) {  
                HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();  
                String ipAddress = getIpAddr(request);  
      
                StringBuffer stringBuffer = new StringBuffer();  
                stringBuffer.append(ipAddress).append("-")  
                        .append(targetClass.getName()).append("- ")  
                        .append(method.getName()).append("-")  
                        .append(rateLimit.key());  
      
                List<String> keys = Collections.singletonList(stringBuffer.toString());  
      
                Number number = limitRedisTemplate.execute(redisluaScript, keys, rateLimit.count(), rateLimit.time());  
      
                if (number != null && number.intValue() != 0 && number.intValue() <= rateLimit.count()) {  
                    logger.info("限流时间段内访问第：{} 次", number.toString());  
                    return joinPoint.proceed();  
                }  
      
            } else {  
                return joinPoint.proceed();  
            }  
      
            throw new RuntimeException("已经到设置限流次数");  
        }  
      
        public static String getIpAddr(HttpServletRequest request) {  
            String ipAddress = null;  
            try {  
                ipAddress = request.getHeader("x-forwarded-for");  
                if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {  
                    ipAddress = request.getHeader("Proxy-Client-IP");  
                }  
                if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {  
                    ipAddress = request.getHeader("WL-Proxy-Client-IP");  
                }  
                if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {  
                    ipAddress = request.getRemoteAddr();  
                }  
                // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割  
                if (ipAddress != null && ipAddress.length() > 15) { // "***.***.***.***".length()  
                    // = 15  
                    if (ipAddress.indexOf(",") > 0) {  
                        ipAddress = ipAddress.substring(0, ipAddress.indexOf(","));  
                    }  
                }  
            } catch (Exception e) {  
                ipAddress = "";  
            }  
            return ipAddress;  
        }  
    }  
      
  
---|---  
  
## 控制层

添加 `@RateLimit()` 注解，会在 Redsi 中生成 10 秒中，可以访问5次 的key

`RedisAtomicLong` 是为测试例子例，记录累计访问次数，跟限流没有关系。

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    

|

    
    
    package com.souyunku.example.controller;  
      
    /**  
     * 描述: 测试页  
     *  
     * @author yanpenglei  
     * @create 2018-08-16 15:42  
     **/  
    @RestController  
    public class LimiterController {  
      
        @Autowired  
        private RedisTemplate redisTemplate;  
      
        // 10 秒中，可以访问10次  
        <