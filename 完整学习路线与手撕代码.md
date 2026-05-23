# 📚完整学习路线与代码指南

---

## 目录

1. [求职意向与技术栈全景](#1-求职意向与技术栈全景)
2. [Java 核心基础（含代码）](#2-java-核心基础含代码)
3. [Spring Boot 自动配置手写](#3-spring-boot-自动配置手写)
4. [Spring Security + JWT 完整实现](#4-spring-security--jwt-完整实现)
5. [项目一：程序员担保交易平台 完整代码剖析](#5-项目一程序员担保交易平台-完整代码剖析)
6. [项目二：工程公司合同管理系统 关键代码](#6-项目二工程公司合同管理系统-关键代码)
7. [项目三：化工集团合同管理系统 关键代码](#7-项目三化工集团合同管理系统-关键代码)
8. [项目四：健康产品集团法律事务系统 关键代码](#8-项目四健康产品集团法律事务系统-关键代码)
9. [MySQL 与 Redis 必会代码](#9-mysql-与-redis-必会代码)
10. [微服务与分布式代码](#10-微服务与分布式代码)
11. [前端 Vue3 必会代码](#11-前端-vue3-必会代码)
12. [DevOps 与部署](#12-devops-与部署)
13. [算法与数据结构手写代码](#13-算法与数据结构手写代码)
14. [AI Agent 辅助开发实践](#14-ai-agent-辅助开发实践)
15. [系统设计练习题](#15-系统设计练习题)
16. [面试 Checklist](#16-面试-checklist)

---

## 1. 求职意向与技术栈全景

### 1.1 目标岗位职责拆解

```
Java后端开发 → 需要掌握：SSM/SpringBoot、MySQL调优、Redis、微服务
全栈开发    → 需要掌握：Vue3 + 后端全套 + 部署
AI Agent    → 需要掌握：Prompt工程、Cursor/Claude使用、AI API调用
```

### 1.2 技术栈掌握程度自检表

| 技术 | 简历声称 | 面试要求 | 代码要求 |
|------|---------|---------|---------|
| Java基础 | 熟悉 | 集合/多线程/JVM底层 | 手写HashMap、线程池、LRU |
| Spring Boot | 熟悉 | 自动配置原理、启动流程 | 手写Starter |
| Spring Cloud | 了解 | Nacos/Sentinel/Gateway | 配置+API调用 |
| MyBatis-Plus | 熟悉 | 多租户、分页、条件构造 | CRUD+拦截器 |
| MySQL | 熟悉 | 索引、锁、MVCC、SQL优化 | EXPLAIN+慢SQL优化 |
| Redis | 熟悉 | 数据结构、缓存问题、分布式锁 | Redisson+Lua脚本 |
| WebSocket | 项目使用 | 原理、集群方案 | 双通道实现 |
| RabbitMQ | 项目使用 | 消息可靠、幂等、顺序 | 生产者+消费者代码 |
| Vue3 | 熟悉 | 响应式原理、Composition API | 组件通信+Pinia |
| Nginx | 了解 | 反向代理、try_files | 配置改写 |

---

## 2. Java 核心基础（含代码）

### [掌握] 2.1 HashMap 手写核心流程

```java
// ========== 面试手写：HashMap put 流程 ==========

// 1. 计算 hash：key.hashCode() 高16位参与异或
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 2. put 流程（核心逻辑）
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // ① 数组为空 → 扩容（resize）
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // ② 桶位为空 → 直接插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // ③ key 已存在 → 覆盖
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // ④ 红黑树节点 → 树插入
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // ⑤ 链表遍历
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash); // 链表长度≥8 → 树化
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // ⑥ 值覆盖 + 返回旧值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // ⑦ 超过阈值 → 扩容（0.75 * capacity）
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

// 扩容：2倍扩容，元素重新分配（rehash）
// JDK8 优化：元素在新数组的位置 = 原位置 OR 原位置 + 旧容量
// 因为容量是2的幂，n-1的二进制最高位就是旧容量的那一位
if ((e.hash & oldCap) == 0) {
    // 原位置不变（lo链表）
} else {
    // 原位置 + oldCap（hi链表）
}
```

### [掌握] 2.2 ConcurrentHashMap 原理代码级理解

```java
// ========== JDK8 ConcurrentHashMap put 流程 ==========
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key 和 value 都不能为 null（区别于 HashMap）
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // ① 延迟初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // ② 桶位为空 → CAS 无锁插入
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break; // CAS 成功则退出
        }
        // ③ 检测到正在扩容 → 帮忙扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // ④ 桶位有元素 → synchronized 锁住桶首节点
        else {
            V oldVal = null;
            synchronized (f) { // 粒度：桶（链表/红黑树头节点）
                // 确认锁的对象没变
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) { // 链表
                        // 遍历链表插入/替换
                    } else if (f instanceof TreeBin) { // 红黑树
                        // 树插入
                    }
                }
            }
        }
    }
}
```

### [掌握] 2.3 线程池手写（含参数详解）

```java
// ========== 线程池创建（7大参数） ==========
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                  // corePoolSize：核心线程数（常驻）
    5,                  // maximumPoolSize：最大线程数
    60L,                // keepAliveTime：非核心线程空闲存活时间
    TimeUnit.SECONDS,   // 时间单位
    new LinkedBlockingQueue<>(100), // workQueue：任务队列
    Executors.defaultThreadFactory(), // threadFactory：线程工厂
    new ThreadPoolExecutor.AbortPolicy() // 拒绝策略
);

// ========== 线程池执行流程 ==========
// corePoolSize 未满 → 创建核心线程执行
// corePoolSize 已满 → 放入 workQueue 排队
// workQueue 已满 → 创建非核心线程执行
// 总线程数达到 maximumPoolSize + workQueue已满 → 执行拒绝策略

// ========== 手写简单线程池（面试） ==========
public class SimpleThreadPool {
    private final BlockingQueue<Runnable> workQueue;
    private final List<Worker> workers = new ArrayList<>();
    private volatile boolean running = true;
    
    public SimpleThreadPool(int poolSize, int queueSize) {
        workQueue = new LinkedBlockingQueue<>(queueSize);
        for (int i = 0; i < poolSize; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            worker.start();
        }
    }
    
    public void execute(Runnable task) {
        if (!running) throw new RejectedExecutionException();
        workQueue.offer(task); // 非阻塞入队
    }
    
    private class Worker extends Thread {
        @Override
        public void run() {
            while (running) {
                try {
                    Runnable task = workQueue.poll(1, TimeUnit.SECONDS);
                    if (task != null) task.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
    }
    
    public void shutdown() {
        running = false;
        for (Worker w : workers) w.interrupt();
    }
}
```

### [掌握] 2.4 ThreadLocal 内存泄漏与正确使用

```java
// ========== ThreadLocal 正确用法 ==========

// 定义（用 static 避免线程重复创建）
private static final ThreadLocal<UserContext> USER_HOLDER = new ThreadLocal<>();

// 使用（try-finally 保证 remove）
public void handleRequest(User user) {
    try {
        USER_HOLDER.set(new UserContext(user));
        // ... 业务逻辑中可以直接取出
        UserContext ctx = USER_HOLDER.get();
    } finally {
        USER_HOLDER.remove(); // 必须 remove！否则线程池复用线程导致内存泄漏
    }
}

// ========== ThreadLocal 原理 ==========
// 每个 Thread 维护一个 ThreadLocalMap
// ThreadLocalMap.Entry 的 key 是 WeakReference<ThreadLocal>
// 但 value 是强引用 → 如果线程存活且不 remove
// → key 被 GC 回收后 value 永远无法访问 → 内存泄漏
```

### [掌握] 2.5 AQS 原理与 ReentrantLock

```java
// ========== AQS 核心（AbstractQueuedSynchronizer）==========
// 三个核心要素：
// 1. volatile int state（同步状态）
// 2. CLH 双向队列（等待线程）
// 3. CAS 操作 state

// ========== ReentrantLock 使用 ==========
private final ReentrantLock lock = new ReentrantLock(true); // fair=true 公平锁

public void doSomething() {
    lock.lock();
    try {
        // 临界区代码
    } finally {
        lock.unlock();
    }
}

// ========== 非公平锁 vs 公平锁 ==========
// 非公平锁：新线程直接 CAS 抢锁（默认，性能高，可能饥饿）
// 公平锁：先检查队列中是否有等待线程，有则排队（避免饥饿，吞吐量低）

// ========== 可重入原理 ==========
// 同一线程多次 lock()：
// state 递增（每次 +1）
// unlock() 时 state 递减，减到 0 才真正释放锁
```

### [掌握] 2.6 JVM 内存模型与 GC

```java
// ========== JVM 参数配置 ==========
// -Xms2g -Xmx2g                         堆大小
// -Xmn512m                              新生代大小
// -XX:MetaspaceSize=256m                元空间
// -XX:+UseG1GC                          使用 G1 垃圾收集器
// -XX:MaxGCPauseMillis=200              最大停顿时间
// -Xlog:gc*:file=gc.log:time,pid        GC 日志

// ========== 对象创建流程 ==========
// 1. 类加载检查 → 2. 分配内存（指针碰撞/空闲列表）
// 3. 初始化零值 → 4. 设置对象头 → 5. 执行 <init>

// ========== 判断对象存活 ==========
// 1. 引用计数法（循环引用问题，主流JVM不用）
// 2. 可达性分析（GC Roots 向下搜索）
// GC Roots：栈帧本地变量、静态变量、JNI句柄、活跃线程、synchronized对象
```

---

## 3. Spring Boot 自动配置手写

### [掌握] 3.1 自定义 Starter 完整实现

```java
// ========== 项目结构 ==========
// my-starter/
// ├── my-starter-autoconfigure/
// │   ├── src/main/java/com/example/starter/
// │   │   ├── autoconfigure/
// │   │   │   └── RedisLockAutoConfiguration.java
// │   │   ├── properties/
// │   │   │   └── RedisLockProperties.java
// │   │   └── service/
// │   │       └── RedisLockService.java
// │   └── src/main/resources/META-INF/
// │       └── spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// └── my-starter/
//     └── pom.xml（只依赖 autoconfigure）

// ========== 1. 配置属性类 ==========
@ConfigurationProperties(prefix = "redis.lock")
public class RedisLockProperties {
    /** 默认锁超时时间（毫秒） */
    private long defaultTimeout = 30000;
    /** 是否启用 watchDog 自动续期 */
    private boolean watchDogEnabled = true;
    
    // getters & setters
}

// ========== 2. 自动配置类 ==========
@AutoConfiguration
@ConditionalOnClass(RedisTemplate.class) // 只有存在 RedisTemplate 才生效
@EnableConfigurationProperties(RedisLockProperties.class)
public class RedisLockAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public RedisLockService redisLockService(
            RedisTemplate<String, String> redisTemplate,
            RedisLockProperties properties) {
        return new RedisLockService(redisTemplate, properties);
    }
}

// ========== 3. SPI 注册文件 ==========
// META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// 内容：
// com.example.starter.autoconfigure.RedisLockAutoConfiguration

// ========== 4. 服务类 ==========
public class RedisLockService {
    private final RedisTemplate<String, String> redisTemplate;
    private final RedisLockProperties properties;
    
    // 加锁（Lua 脚本保证原子性）
    public boolean tryLock(String key, String value, long expireMs) {
        String lua = """
            if redis.call('SET', KEYS[1], ARGV[1], 'NX', 'PX', ARGV[2]) then
                return 1
            end
            return 0
            """;
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(lua, Long.class),
            List.of(key), value, String.valueOf(expireMs)
        );
        return Long.valueOf(1).equals(result);
    }
    
    // 解锁（Lua 脚本：比较 value 防止误删）
    public boolean unlock(String key, String value) {
        String lua = """
            if redis.call('GET', KEYS[1]) == ARGV[1] then
                return redis.call('DEL', KEYS[1])
            end
            return 0
            """;
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(lua, Long.class),
            List.of(key), value
        );
        return Long.valueOf(1).equals(result);
    }
}
```

### [掌握] 3.2 @SpringBootApplication 注解拆解

```java
// ========== @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan ==========

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration        // ★ 核心：开启自动配置
@ComponentScan(excludeFilters = { // ★ 组件扫描
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)
})
public @interface SpringBootApplication {}

// ========== @EnableAutoConfiguration 核心 ==========
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class) // ★ 关键：导入选择器
public @interface EnableAutoConfiguration {}

// AutoConfigurationImportSelector 会加载
// META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
// 文件中所有配置类，根据 @ConditionalOnXxx 条件生效
```

### [掌握] 3.3 Spring 启动流程

```java
// SpringApplication.run() 核心步骤：
// 1. 推断应用类型（Reactive/Servlet/None）
// 2. 从 spring.factories 加载所有 ApplicationContextInitializer
// 3. 加载所有 ApplicationListener
// 4. 推断主启动类
// 5. 创建 Environment，加载配置
// 6. 创建 ApplicationContext（AnnotationConfigServletWebServerApplicationContext）
// 7. prepareContext：设置 Environment、执行初始化器、注册 BeanDefinition
// 8. refreshContext：调用 AbstractApplicationContext.refresh()
//    ↓
//    refresh() 内部：
//    - prepareRefresh()          → 准备刷新
//    - obtainFreshBeanFactory()  → 获取BeanFactory
//    - prepareBeanFactory()      → 设置类加载器/表达式解析器
//    - postProcessBeanFactory()  → BeanFactory后置处理
//    - invokeBeanFactoryPostProcessors() → 执行 BeanFactoryPostProcessor
//    - registerBeanPostProcessors()     → 注册 BeanPostProcessor
//    - initMessageSource()       → 初始化消息源
//    - initApplicationEventMulticaster() → 初始化事件广播器
//    - onRefresh()               → 创建 WebServer（内嵌 Tomcat）
//    - registerListeners()       → 注册监听器
//    - finishBeanFactoryInitialization() → 实例化所有非懒加载单例 Bean
//    - finishRefresh()           → 发布 ContextRefreshedEvent，启动 WebServer
// 9. 调用 CommandLineRunner / ApplicationRunner
```

---

## 4. Spring Security + JWT 完整实现

### [掌握] 4.1 JWT 工具类

```java
// ========== JWT 工具类（使用 io.jsonwebtoken:jjwt）==========
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret:mySecretKeyForJWTTokenGeneration2024}")
    private String jwtSecret;
    
    @Value("${jwt.access-token-expiration:1800000}")   // 30分钟
    private long accessTokenExpiration;
    
    @Value("${jwt.refresh-token-expiration:604800000}") // 7天
    private long refreshTokenExpiration;
    
    // 生成 Access Token
    public String generateAccessToken(Long userId, String username, String role) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + accessTokenExpiration);
        
        return Jwts.builder()
                .setSubject(userId.toString())
                .claim("username", username)
                .claim("role", role)
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();
    }
    
    // 生成 Refresh Token（更长有效期）
    public String generateRefreshToken(Long userId) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + refreshTokenExpiration);
        
        return Jwts.builder()
                .setSubject(userId.toString())
                .claim("type", "refresh")
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();
    }
    
    // 从 Token 中提取用户 ID
    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
        return Long.parseLong(claims.getSubject());
    }
    
    // 从 Token 中提取角色
    public String getRoleFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
        return claims.get("role", String.class);
    }
    
    // 验证 Token（同时检查 Redis 黑名单）
    public boolean validateToken(String token, StringRedisTemplate redisTemplate) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            // 检查是否在黑名单中（已注销/已刷新）
            String blacklisted = redisTemplate.opsForValue()
                    .get("jwt:blacklist:" + token);
            return blacklisted == null;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

### [掌握] 4.2 JWT 无感刷新过滤器（滑动续期）

```java
// ========== JWT 认证过滤器 ==========
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    // 续期阈值：剩余有效期 < 30% 时续期
    private static final double RENEW_THRESHOLD = 0.3;
    // 强制刷新阈值：剩余有效期 < 10% 时强制刷新
    private static final double FORCE_RENEW_THRESHOLD = 0.1;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        
        String token = getTokenFromRequest(request);
        
        if (StringUtils.hasText(token)) {
            try {
                // 解析 token
                Claims claims = JwtUtils.parseToken(token);
                Long userId = Long.parseLong(claims.getSubject());
                
                // ★ 滑动续期逻辑
                Date expiration = claims.getExpiration();
                long now = System.currentTimeMillis();
                long totalLifetime = expiration.getTime() - claims.getIssuedAt().getTime();
                long remaining = expiration.getTime() - now;
                double ratio = (double) remaining / totalLifetime;
                
                if (ratio < FORCE_RENEW_THRESHOLD) {
                    // 剩余不足 10% → 强制刷新（返回 401，前端调 refresh 接口）
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    response.setHeader("X-Token-Expired", "true");
                    return;
                } else if (ratio < RENEW_THRESHOLD) {
                    // 剩余 10%-30% → 滑动续期（签发新 token，通过响应头返回）
                    String newToken = tokenProvider.generateAccessToken(
                        userId, 
                        claims.get("username", String.class),
                        claims.get("role", String.class)
                    );
                    response.setHeader("X-Refresh-Token", newToken);
                    
                    // 将旧 token 加入黑名单（防止重放）
                    redisTemplate.opsForValue().set(
                        "jwt:blacklist:" + token,
                        "true",
                        remaining,
                        TimeUnit.MILLISECONDS
                    );
                }
                
                // 设置 SecurityContext
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userId, null, 
                        List.of(new SimpleGrantedAuthority("ROLE_" + claims.get("role"))));
                SecurityContextHolder.getContext().setAuthentication(authentication);
                
            } catch (JwtException e) {
                // Token 无效
                SecurityContextHolder.clearContext();
            }
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

### [掌握] 4.3 前端 Axios 拦截器处理滑动续期

```javascript
// ========== axios 拦截器（前端核心代码）==========
import axios from 'axios'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const request = axios.create({
  baseURL: '/api',
  timeout: 15000
})

// 请求拦截器：自动带 token
request.interceptors.request.use(config => {
  const token = localStorage.getItem('accessToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 响应拦截器：处理 JWT 滑动续期
request.interceptors.response.use(
  response => {
    // ★ 检测响应头中是否有新 token → 滑动续期
    const newToken = response.headers['x-refresh-token']
    if (newToken) {
      localStorage.setItem('accessToken', newToken)
    }
    return response
  },
  async error => {
    const originalRequest = error.config
    
    // 401 → token 过期，尝试刷新
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true
      
      try {
        // 检查是否是因为需要强制刷新（X-Token-Expired 头）
        if (error.response.headers['x-token-expired'] === 'true') {
          // 调用 refresh 接口换新 token
          const refreshToken = localStorage.getItem('refreshToken')
          const res = await axios.post('/api/auth/refresh', {
            refreshToken
          })
          
          const newAccessToken = res.data.accessToken
          localStorage.setItem('accessToken', newAccessToken)
          
          // 重放原始请求
          originalRequest.headers.Authorization = `Bearer ${newAccessToken}`
          return request(originalRequest)
        }
      } catch (refreshError) {
        // 刷新也失败 → 跳转登录页
        localStorage.clear()
        const router = useRouter()
        router.push('/login')
        return Promise.reject(refreshError)
      }
    }
    
    return Promise.reject(error)
  }
)

export default request
```

### [掌握] 4.4 Spring Security 配置

```java
// ========== Spring Security 核心配置 ==========
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // 开启 @PreAuthorize
public class SecurityConfig {
    
    @Autowired
    private JwtAuthenticationFilter jwtFilter;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // REST API 不需要 CSRF
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // 无状态
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()    // 登录注册开放
                .requestMatchers("/api/public/**").permitAll()  // 公共接口开放
                .requestMatchers("/ws/**").permitAll()          // WebSocket 握手
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // 管理员
                .anyRequest().authenticated()                   // 其他都需要认证
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((request, response, authException) -> {
                    response.setContentType("application/json;charset=UTF-8");
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    response.getWriter().write("{\"code\":401,\"msg\":\"未登录或token过期\"}");
                })
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// ========== 方法级别权限控制 ==========
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @PreAuthorize("hasRole('DEVELOPER')")
    @PostMapping("/{orderId}/bid")
    public Result submitBid(@PathVariable Long orderId, @RequestBody BidDTO bid) {
        // 只有 DEVELOPER 角色能报价
        return orderService.submitBid(orderId, bid);
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    @PutMapping("/{orderId}/ban")
    public Result banUser(@PathVariable Long orderId) {
        // 只有 ADMIN 能封禁
        return orderService.banOrder(orderId);
    }
}
```

---

## 5. 项目一：程序员担保交易平台 完整代码剖析

> ★ **面试核心项目**，必须能讲清楚每一个技术点

### [掌握] 5.1 订单状态机

```java
// ========== 订单状态枚举 ==========
public enum OrderStatus {
    PENDING_QUOTE(0, "待报价"),
    PENDING_SELECT(1, "待选标"),
    PENDING_DEVELOP(2, "待开发"),
    PENDING_ACCEPT(3, "待验收"),
    COMPLETED(4, "已完成"),
    CLOSED(5, "已关闭"),
    DISPUTE(6, "纠纷中");
    
    private final int code;
    private final String desc;
    
    // getters
}

// ========== 状态机接口 ==========
public interface OrderState {
    /** 当前状态 */
    OrderStatus getStatus();
    /** 检查操作是否允许 */
    boolean canDo(OrderAction action);
    /** 执行操作 → 返回新状态 */
    OrderStatus doAction(OrderAction action, Order order);
}

// ========== 具体状态实现（以"待验收"状态为例）==========
@Component
public class PendingAcceptState implements OrderState {
    
    @Override
    public OrderStatus getStatus() {
        return OrderStatus.PENDING_ACCEPT;
    }
    
    @Override
    public boolean canDo(OrderAction action) {
        // 待验收状态下，只有买家可以执行的操作
        return action == OrderAction.ACCEPT    // 确认验收
            || action == OrderAction.REJECT    // 驳回
            || action == OrderAction.OPEN_DISPUTE; // 发起纠纷
    }
    
    @Override
    public OrderStatus doAction(OrderAction action, Order order) {
        return switch (action) {
            case ACCEPT ->        OrderStatus.COMPLETED;    // 验收通过 → 已完成
            case REJECT ->        OrderStatus.PENDING_DEVELOP; // 驳回 → 回到待开发
            case OPEN_DISPUTE ->  OrderStatus.DISPUTE;      // 纠纷
            default -> throw new IllegalStateException(
                "当前状态不允许操作: " + action);
        };
    }
}

// ========== 状态机管理器 ==========
@Component
public class OrderStateMachine {
    
    private final Map<OrderStatus, OrderState> stateMap;
    
    public OrderStateMachine(List<OrderState> states) {
        this.stateMap = states.stream()
            .collect(Collectors.toMap(OrderState::getStatus, Function.identity()));
    }
    
    @Transactional
    public void execute(Long orderId, OrderAction action) {
        Order order = orderMapper.selectById(orderId);
        OrderState currentState = stateMap.get(order.getStatus());
        
        // 1. 校验操作是否允许
        if (!currentState.canDo(action)) {
            throw new BusinessException("当前状态不允许此操作");
        }
        
        // 2. 执行状态转换
        OrderStatus newStatus = currentState.doAction(action, order);
        
        // 3. 持久化
        order.setStatus(newStatus);
        orderMapper.updateById(order);
        
        // 4. 记录状态变更日志
        orderLogService.record(orderId, order.getStatus(), newStatus, action);
    }
}
```

### [掌握] 5.2 竞价超卖问题解决方案（三层防护）

```java
// ========== 方案一：数据库唯一索引 ==========
// SQL:
// CREATE UNIQUE INDEX uk_order_developer ON bid (order_id, developer_id);

// ========== 方案二：Redisson 分布式锁 ==========
@Service
public class BidService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    @Autowired
    private BidMapper bidMapper;
    
    public Result submitBid(Long orderId, Long developerId, BigDecimal amount) {
        // 1. 获取分布式锁（key = "bid:lock:" + orderId）
        RLock lock = redissonClient.getLock("bid:lock:" + orderId);
        
        try {
            // 2. 尝试加锁（等待5秒，持有30秒，WatchDog自动续期）
            boolean locked = lock.tryLock(5, 30, TimeUnit.SECONDS);
            if (!locked) {
                return Result.error("系统繁忙，请稍后重试");
            }
            
            // 3. 检查当前报价数量
            int count = bidMapper.countByOrderId(orderId);
            if (count >= 3) {
                return Result.error("已达最大报价数量");
            }
            
            // 4. 插入报价
            Bid bid = new Bid();
            bid.setOrderId(orderId);
            bid.setDeveloperId(developerId);
            bid.setAmount(amount);
            bidMapper.insert(bid);
            
            return Result.success("报价成功");
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return Result.error("系统异常");
        } finally {
            // 5. 释放锁
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}

// ========== 方案三：SQL 条件更新（最后一道防线）==========
// 不依赖锁，数据库层面保证不超过限制
// 但需要配合业务判断返回值
@Update("UPDATE orders SET bid_count = bid_count + 1 " +
        "WHERE id = #{orderId} AND bid_count < #{maxCount}")
int incrementBidCount(@Param("orderId") Long orderId, @Param("maxCount") int maxCount);
// 返回值 > 0 表示更新成功（未超限），= 0 表示已满
```

### [掌握] 5.3 WebSocket 双通道实现

```java
// ========== WebSocket 配置 ==========
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 服务端推送前缀（客户端订阅）
        config.enableSimpleBroker("/topic", "/queue");
        // 客户端发送前缀
        config.setApplicationDestinationPrefixes("/app");
        // 点对点前缀
        config.setUserDestinationPrefix("/user");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 通道1：扫码登录（按临时 token 建立连接）
        registry.addEndpoint("/ws/login")
                .setAllowedOriginPatterns("*")
                .addInterceptors(new LoginHandshakeInterceptor());
        
        // 通道2：聊天/通知（按 JWT 鉴权）
        registry.addEndpoint("/ws/chat")
                .setAllowedOriginPatterns("*")
                .addInterceptors(new JwtHandshakeInterceptor());
    }
}

// ========== JWT 握手拦截器（通道2）==========
public class JwtHandshakeInterceptor implements HandshakeInterceptor {
    
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                   WebSocketHandler wsHandler, Map<String, Object> attributes) {
        String token = extractToken(request);
        if (token != null && jwtProvider.validateToken(token)) {
            Long userId = jwtProvider.getUserIdFromToken(token);
            attributes.put("userId", userId); // 后续从 session 中取
            return true;
        }
        return false; // 鉴权失败，拒绝连接
    }
}

// ========== 消息推送服务 ==========
@Service
public class NotificationService {
    
    @Autowired
    private SimpMessagingTemplate messagingTemplate;
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    // 向指定用户推送站内信
    public void sendNotification(Long userId, Notification notification) {
        // 1. 实时推送（WebSocket）
        messagingTemplate.convertAndSendToUser(
            userId.toString(), 
            "/queue/notification", 
            notification
        );
        
        // 2. 未读计数增加
        redisTemplate.opsForHash().increment(
            "unread:" + userId, 
            "notification", 
            1
        );
    }
    
    // 发送聊天消息
    public void sendChatMessage(Long receiverId, ChatMessage message) {
        messagingTemplate.convertAndSendToUser(
            receiverId.toString(),
            "/queue/chat",
            message
        );
    }
    
    // 扫码登录成功推送
    public void sendLoginSuccess(String tempToken, String jwtToken) {
        messagingTemplate.convertAndSend(
            "/topic/login/" + tempToken,
            new LoginSuccessMessage(jwtToken)
        );
        // 推送后连接会自动断开（客户端处理）
    }
}
```

### [掌握] 5.4 前端 WebSocket 双通道 + 指数退避重连

```javascript
// ========== 扫码登录通道 ==========
// utils/scanLogin.js
let stompClient = null

export function connectScanLogin(tempToken) {
  const socket = new SockJS('/ws/login')
  stompClient = Stomp.over(socket)
  
  // 指数退避重连参数
  let retryCount = 0
  const MAX_RETRY = 5
  const INITIAL_DELAY = 1000  // 1秒
  
  stompClient.connect({ tempToken }, frame => {
    retryCount = 0  // 连接成功，重置重试计数
    
    stompClient.subscribe(`/topic/login/${tempToken}`, message => {
      const data = JSON.parse(message.body)
      // 登录成功，保存 token
      localStorage.setItem('accessToken', data.accessToken)
      localStorage.setItem('refreshToken', data.refreshToken)
      // 主动断开此通道
      stompClient.disconnect()
      // 跳转到首页
      window.location.href = '/'
    })
  }, error => {
    // 连接失败 → 指数退避重连
    if (retryCount < MAX_RETRY) {
      const delay = Math.min(
        INITIAL_DELAY * Math.pow(2, retryCount),
        30000  // 最大 30 秒
      )
      retryCount++
      console.log(`WebSocket 将在 ${delay}ms 后重连 (第${retryCount}次)`)
      setTimeout(() => connectScanLogin(tempToken), delay)
    }
  })
}

// ========== 聊天/通知通道 ==========
// stores/websocket.js
import { defineStore } from 'pinia'
import { Client } from '@stomp/stompjs'

export const useWebSocketStore = defineStore('websocket', {
  state: () => ({
    client: null,
    connected: false,
    unreadCount: 0,
    messages: [],
    reconnectAttempts: 0
  }),
  
  actions: {
    connect() {
      const token = localStorage.getItem('accessToken')
      if (!token) return
      
      this.client = new Client({
        brokerURL: 'ws://localhost:8080/ws/chat',
        connectHeaders: { Authorization: `Bearer ${token}` },
        // 自动重连（指数退避）
        reconnectDelay: 5000,        // 基础延迟5秒
        maxReconnectDelay: 30000,    // 最大30秒
        heartbeatIncoming: 10000,    // 10秒心跳
        heartbeatOutgoing: 10000,
        
        onConnect: () => {
          this.connected = true
          this.reconnectAttempts = 0
          
          // 订阅个人通知
          this.client.subscribe('/user/queue/notification', msg => {
            const notification = JSON.parse(msg.body)
            this.handleNotification(notification)
          })
          
          // 订阅聊天消息
          this.client.subscribe('/user/queue/chat', msg => {
            const chatMsg = JSON.parse(msg.body)
            this.messages.push(chatMsg)
          })
        },
        
        onDisconnect: () => {
          this.connected = false
        },
        
        onStompError: (error) => {
          console.error('STOMP 错误:', error)
        }
      })
      
      this.client.activate()
      
      // ★ 30秒轮询保底（WebSocket 断开时兜底）
      this.startPolling()
    },
    
    startPolling() {
      // 每30秒轮询未读消息
      setInterval(async () => {
        try {
          const res = await axios.get('/api/notifications/unread')
          if (res.data.code === 200) {
            this.unreadCount = res.data.data.count
            // 如果有新消息但WebSocket没收到，补充拉取
            if (res.data.data.messages?.length) {
              this.messages.push(...res.data.data.messages)
            }
          }
        } catch (e) {
          // 静默失败（WebSocket 正常时轮询只是备胎）
        }
      }, 30000)
    },
    
    sendMessage(chatId, content) {
      if (this.client?.connected) {
        this.client.publish({
          destination: '/app/chat.send',
          body: JSON.stringify({ chatId, content })
        })
      }
    },
    
    disconnect() {
      this.client?.deactivate()
    }
  }
})
```

---

## 6. 项目二：工程公司合同管理系统 关键代码

### [掌握] 6.1 定时任务数据同步

```java
// ========== 定时同步任务 ==========
@Component
public class ContractSyncTask {
    
    @Autowired
    private ContractMapper contractMapper;
    
    @Autowired
    private RestTemplate restTemplate;
    
    // 上次同步时间（从数据库读取）
    private volatile LocalDateTime lastSyncTime;
    
    // 每日凌晨2点执行
    @Scheduled(cron = "0 0 2 * * ?")
    @Transactional
    public void syncContracts() {
        log.info("开始同步合同数据，上次同步时间：{}", lastSyncTime);
        
        // 1. 从人力系统拉取增量数据
        List<ContractDTO> contracts = fetchIncrementalContracts(lastSyncTime);
        
        // 2. 逐批处理（每批500条）
        List<Contract> batch = new ArrayList<>();
        for (ContractDTO dto : contracts) {
            batch.add(convertToEntity(dto));
            
            if (batch.size() >= 500) {
                // ★ 幂等插入：业务主键冲突时更新
                batchSaveOrUpdate(batch);
                batch.clear();
            }
        }
        
        // 3. 处理剩余批次
        if (!batch.isEmpty()) {
            batchSaveOrUpdate(batch);
        }
        
        // 4. 更新同步时间
        lastSyncTime = LocalDateTime.now();
        log.info("合同数据同步完成，共处理 {} 条", contracts.size());
    }
    
    private void batchSaveOrUpdate(List<Contract> contracts) {
        // ★ MyBatis-Plus 批量插入（配合 rewriteBatchedStatements=true）
        contractMapper.saveOrUpdateBatch(contracts);
    }
    
    private List<ContractDTO> fetchIncrementalContracts(LocalDateTime syncTime) {
        // 调用外部人力系统接口
        return restTemplate.postForObject(
            "http://hr-system/api/contracts/sync",
            new SyncRequest(syncTime),
            new ParameterizedTypeReference<List<ContractDTO>>() {}
        );
    }
}

// ========== 批量插入 SQL（XML） ==========
// <insert id="saveOrUpdateBatch">
//   INSERT INTO contract (contract_no, employee_name, department, ...)
//   VALUES
//   <foreach collection="list" item="item" separator=",">
//       (#{item.contractNo}, #{item.employeeName}, #{item.department}, ...)
//   </foreach>
//   ON DUPLICATE KEY UPDATE
//       employee_name = VALUES(employee_name),
//       department = VALUES(department),
//       update_time = NOW()
// </insert>

// ========== application.yml JDBC 配置 ==========
// spring:
//   datasource:
//     url: jdbc:mysql://localhost:3306/contract_db?rewriteBatchedStatements=true&useServerPrepStmts=false
```

### [掌握] 6.2 RBAC 权限控制

```java
// ========== Spring Security 方法级别授权 ==========
@RestController
@RequestMapping("/api/contracts")
public class ContractController {
    
    @PreAuthorize("hasPermission('contract', 'view')")
    @GetMapping("/list")
    public Result list() { ... }
    
    @PreAuthorize("hasPermission('contract', 'approve')")
    @PostMapping("/{id}/approve")
    public Result approve(@PathVariable Long id) { ... }
    
    @PreAuthorize("hasPermission('contract', 'edit')")
    @PutMapping("/{id}")
    public Result update(@PathVariable Long id, @RequestBody ContractDTO dto) { ... }
}

// ========== 自定义 PermissionEvaluator ==========
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    
    @Autowired
    private RolePermissionMapper rolePermissionMapper;
    
    @Override
    public boolean hasPermission(Authentication auth, Object targetDomainObject, Object permission) {
        // 从认证信息获取当前用户角色
        UserDetails user = (UserDetails) auth.getPrincipal();
        String role = user.getAuthorities().stream()
            .findFirst()
            .map(GrantedAuthority::getAuthority)
            .orElse("");
        
        // 查询角色是否有权限
        return rolePermissionMapper.checkPermission(
            role, 
            targetDomainObject.toString(), 
            permission.toString()
        ) > 0;
    }
    
    @Override
    public boolean hasPermission(Authentication auth, Serializable targetId, String targetType, Object permission) {
        return false;
    }
}
```

---

## 7. 项目三：化工集团合同管理系统 关键代码

### [掌握] 7.1 Nacos 配置热更新

```java
// ========== 配置类（@RefreshScope 实现热更新）==========
@Component
@RefreshScope  // ★ 关键：配置变更时重新创建 Bean
@ConfigurationProperties(prefix = "contract.system")
public class ContractSystemConfig {
    
    /** 审批超时时间（小时） */
    private int approveTimeoutHours = 48;
    
    /** 是否启用自动审批 */
    private boolean autoApproveEnabled = false;
    
    /** 合同最大附件大小（MB） */
    private int maxAttachmentSize = 20;
    
    // getters & setters
}

// ========== 使用配置的服务 ==========
@Service
public class ContractApproveService {
    
    @Autowired
    private ContractSystemConfig config;
    
    public void checkApproveTimeout(Long contractId) {
        // 配置变更后，这里获取的是最新的值（无需重启）
        int timeoutHours = config.getApproveTimeoutHours();
        // ... 业务逻辑
    }
}

// ========== Nacos 配置内容（Data ID: contract-system.yaml）==========
// contract:
//   system:
//     approve-timeout-hours: 48
//     auto-approve-enabled: true
//     max-attachment-size: 20
```

### [掌握] 7.2 MyBatis-Plus 多租户实现

```java
// ========== 多租户配置 ==========
@Configuration
public class MyBatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        
        // 添加租户插件
        interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(
            new TenantLineHandler() {
                
                @Override
                public Expression getTenantId() {
                    // 从 JWT 中获取当前用户的租户 ID
                    Long tenantId = TenantContext.getCurrentTenantId();
                    return new LongValue(tenantId);
                }
                
                @Override
                public String getTenantIdColumn() {
                    return "tenant_id"; // 表字段名
                }
                
                @Override
                public boolean ignoreTable(String tableName) {
                    // 全局共享表跳过租户过滤
                    return List.of(
                        "sys_dict", "sys_config", 
                        "sys_region", "sys_log"
                    ).contains(tableName);
                }
            }
        ));
        
        return interceptor;
    }
}

// ========== TenantContext（请求上下文）==========
public class TenantContext {
    private static final ThreadLocal<Long> TENANT_HOLDER = new ThreadLocal<>();
    
    public static void setCurrentTenantId(Long tenantId) {
        TENANT_HOLDER.set(tenantId);
    }
    
    public static Long getCurrentTenantId() {
        Long tenantId = TENANT_HOLDER.get();
        if (tenantId == null) {
            throw new BusinessException("未获取到租户信息");
        }
        return tenantId;
    }
    
    public static void clear() {
        TENANT_HOLDER.remove();
    }
}

// ========== 过滤器：请求时设置租户上下文 ==========
@Component
public class TenantFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        try {
            // 从 JWT 或请求头中提取租户 ID
            String tenantId = request.getHeader("X-Tenant-Id");
            if (StringUtils.hasText(tenantId)) {
                TenantContext.setCurrentTenantId(Long.parseLong(tenantId));
            }
            chain.doFilter(request, response);
        } finally {
            TenantContext.clear(); // 必须清理
        }
    }
}
```

### [掌握] 7.3 Object.freeze() 优化首屏性能

```javascript
// ========== Vue3 首屏性能优化（Object.freeze）==========
// stores/contract.js
import { defineStore } from 'pinia'

export const useContractStore = defineStore('contract', {
  state: () => ({
    // 只读展示的合同列表（已审批通过的，不需要响应式追踪）
    approvedContracts: [],
    
    // 需要编辑的合同（需要响应式）
    editableContract: null,
    
    loading: false
  }),
  
  actions: {
    async fetchApprovedContracts() {
      this.loading = true
      try {
        const res = await axios.get('/api/contracts/approved')
        if (res.data.code === 200) {
          // ★ 核心优化：Object.freeze 冻结数据
          // 告诉 Vue 不要递归代理这些对象
          // 减少 getter/setter 初始化开销
          this.approvedContracts = Object.freeze(
            res.data.data.map(item => Object.freeze(item))
          )
        }
      } finally {
        this.loading = false
      }
    }
  }
})

// ========== Object.freeze 原理 ==========
// 普通响应式数据：Vue 用 Proxy 代理每个属性
//    → 递归遍历所有层 → 递归开销大
//    → 每个属性都有 getter/setter → 内存占用大
//
// Object.freeze() 后的对象：
//    → 属性不可配置（writable: false, configurable: false）
//    → Vue 的响应式系统检测到冻结 → 跳过代理
//    → 节省大量 getter/setter 初始化时间
//
// 适用场景：纯展示、不需要修改的大数据集合
// 不适用场景：需要动态更新、双向绑定的数据

// ========== 对比测试 ==========
// 优化前：1000 条合同数据，每条 20 个字段
// Vue 递归创建 20000+ getter/setter → 耗时约 120ms
//
// 优化后：Object.freeze 冻结
// Vue 跳过代理 → 耗时约 5ms
// 首屏渲染性能提升约 35%（项目实测）
```

### [掌握] 7.4 Sentinel 规则持久化到 Nacos

```java
// ========== Sentinel 配置（规则持久化到 Nacos）==========
@Configuration
public class SentinelConfig {
    
    @PostConstruct
    public void initSentinelRules() {
        // 1. 从 Nacos 读取限流规则
        ReadableDataSource<String, List<FlowRule>> flowRuleDataSource =
            new NacosDataSource<>(
                "localhost:8848",           // Nacos 地址
                "DEFAULT_GROUP",            // 分组
                "sentinel-flow-rules",      // Data ID
                source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {})
            );
        
        // 2. 注册数据源（规则变更自动推送）
        FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
        
        // 3. 同样的方式处理熔断规则
        ReadableDataSource<String, List<DegradeRule>> degradeRuleDataSource =
            new NacosDataSource<>(
                "localhost:8848",
                "DEFAULT_GROUP",
                "sentinel-degrade-rules",
                source -> JSON.parseObject(source, new TypeReference<List<DegradeRule>>() {})
            );
        DegradeRuleManager.register2Property(degradeRuleDataSource.getProperty());
        
        log.info("Sentinel 规则已从 Nacos 加载");
    }
}

// ========== Nacos 中限流规则配置（sentinel-flow-rules）==========
// [
//     {
//         "resource": "contract:approve",
//         "limitApp": "default",
//         "grade": 1,
//         "count": 200,
//         "strategy": 0,
//         "controlBehavior": 0,
//         "clusterMode": false
//     },
//     {
//         "resource": "contract:list",
//         "limitApp": "default",
//         "grade": 1,
//         "count": 500,
//         "strategy": 0,
//         "controlBehavior": 0,
//         "clusterMode": false
//     }
// ]

// ========== 业务代码中使用 Sentinel ==========
@RestController
@RequestMapping("/api/contracts")
public class ContractController {
    
    @SentinelResource(
        value = "contract:approve",
        blockHandler = "approveBlockHandler",
        fallback = "approveFallback"
    )
    @PostMapping("/{id}/approve")
    public Result approve(@PathVariable Long id) {
        // 被限流后执行 blockHandler
        return contractService.approve(id);
    }
    
    // 限流处理（降级）
    public Result approveBlockHandler(Long id, BlockException e) {
        return Result.error("系统繁忙，请稍后重试");
    }
    
    // 异常处理（兜底）
    public Result approveFallback(Long id, Throwable e) {
        log.error("审批异常，contractId: {}", id, e);
        return Result.error("审批服务异常，已降级处理");
    }
}
```

### [掌握] 7.5 Nginx try_files 处理 SPA 404

```nginx
# ========== SPA 部署配置 ==========
server {
    listen       80;
    server_name  example.com;
    
    root   /usr/share/nginx/html;
    index  index.html;
    
    # ★ 核心：try_files 解决 History 模式刷新 404
    location / {
        try_files $uri $uri/ /index.html;
        # 解释：
        # 1. $uri      → 尝试访问确切的文件（如 /static/js/app.js）
        # 2. $uri/     → 尝试访问目录（失败则继续）
        # 3. /index.html → 兜底：所有未匹配路由都交给 index.html
        #                 前端 Vue Router 接管路由分发
    }
    
    # 静态资源缓存（JS/CSS/图片）
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
    
    # API 反向代理
    location /api/ {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # WebSocket 反向代理
    location /ws/ {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

---

## 8. 项目四：健康产品集团法律事务系统 关键代码

### [掌握] 8.1 Redisson 分布式锁 + WatchDog

```java
// ========== Redisson 分布式锁完整代码 ==========
@Service
public class ContractSubmitService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    @Autowired
    private ContractMapper contractMapper;
    
    /**
     * 提交合同（防止重复提交）
     */
    public Result submitContract(ContractSubmitDTO dto) {
        // 1. 获取分布式锁（key = "contract:submit:" + contractId）
        RLock lock = redissonClient.getLock("contract:submit:" + dto.getContractId());
        
        try {
            // 2. 尝试加锁（等待3秒）
            //    ★ WatchDog 自动续期：默认每10秒检查一次
            //       如果业务没执行完，自动续期30秒
            //       如果线程挂了，不会续期，锁自动过期
            boolean locked = lock.tryLock(3, 30, TimeUnit.SECONDS);
            
            if (!locked) {
                return Result.error("合同正在提交中，请勿重复操作");
            }
            
            // 3. 检查合同状态（幂等性检查）
            Contract contract = contractMapper.selectById(dto.getContractId());
            if (contract == null) {
                return Result.error("合同不存在");
            }
            if (contract.getStatus() != ContractStatus.DRAFT) {
                return Result.error("合同状态异常，不能重复提交");
            }
            
            // 4. 提交合同（可能长事务：OCR识别、文档比对等）
            contract.setStatus(ContractStatus.PENDING_APPROVE);
            contract.setSubmitTime(LocalDateTime.now());
            contractMapper.updateById(contract);
            
            // 5. 异步处理繁重任务（OCR识别等）
            rabbitTemplate.convertAndSend("contract.exchange", 
                "contract.submitted", 
                new ContractEvent(dto.getContractId()));
            
            return Result.success("合同提交成功，等待审批");
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return Result.error("系统异常");
        } finally {
            // 6. 释放锁（WatchDog 自动停止续期）
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}

// ========== Redisson 配置 ==========
@Configuration
public class RedissonConfig {
    
    @Value("${spring.redis.host:localhost}")
    private String redisHost;
    
    @Value("${spring.redis.port:6379}")
    private int redisPort;
    
    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        
        // 单节点模式
        config.useSingleServer()
              .setAddress("redis://" + redisHost + ":" + redisPort)
              .setConnectionPoolSize(10)
              .setConnectionMinimumIdleSize(5);
        
        // ★ WatchDog 配置
        // 默认：每 10 秒检查一次，续期 30 秒
        // 可以通过以下方式调整：
        // lockWatchdogTimeout = 30000（默认30秒）
        // 但不建议修改，Redisson 的默认值已经过大量验证
        
        return Redisson.create(config);
    }
}
```

### [掌握] 8.2 RabbitMQ 异步处理重型任务

```java
// ========== RabbitMQ 配置 ==========
@Configuration
public class RabbitMQConfig {
    
    // 合同交换机
    public static final String CONTRACT_EXCHANGE = "contract.exchange";
    // OCR 识别队列
    public static final String OCR_QUEUE = "contract.ocr.queue";
    // 合同比对队列
    public static final String COMPARE_QUEUE = "contract.compare.queue";
    // 通知队列
    public static final String NOTIFY_QUEUE = "contract.notify.queue";
    
    @Bean
    public TopicExchange contractExchange() {
        return new TopicExchange(CONTRACT_EXCHANGE);
    }
    
    @Bean
    public Queue ocrQueue() {
        return QueueBuilder.durable(OCR_QUEUE)
                .maxLength(10000)
                .build();
    }
    
    @Bean
    public Queue compareQueue() {
        return QueueBuilder.durable(COMPARE_QUEUE)
                .maxLength(10000)
                .build();
    }
    
    @Bean
    public Queue notifyQueue() {
        return QueueBuilder.durable(NOTIFY_QUEUE)
                .build();
    }
    
    @Bean
    public Binding ocrBinding() {
        return BindingBuilder.bind(ocrQueue())
                .to(contractExchange())
                .with("contract.ocr.*");
    }
    
    @Bean
    public Binding compareBinding() {
        return BindingBuilder.bind(compareQueue())
                .to(contractExchange())
                .with("contract.compare.*");
    }
    
    @Bean
    public Binding notifyBinding() {
        return BindingBuilder.bind(notifyQueue())
                .to(contractExchange())
                .with("contract.notify.*");
    }
}

// ========== 生产者：发送 OCR 任务 ==========
@Service
public class ContractEventPublisher {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    // 发送 OCR 识别任务
    public void sendOcrTask(Long contractId, String fileUrl) {
        OcrTask task = new OcrTask(contractId, fileUrl);
        
        // ★ 确认机制：确保消息到达交换机
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (!ack) {
                log.error("OCR 任务发送失败: {}, cause: {}", task, cause);
                // 可以重试或记录到数据库
            }
        });
        
        // 发送消息
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.CONTRACT_EXCHANGE,
            "contract.ocr.submitted",
            task,
            message -> {
                // 设置消息持久化
                message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                // 设置消息 ID（用于幂等）
                message.getMessageProperties().setMessageId(correlationData.getId());
                return message;
            },
            correlationData
        );
    }
}

// ========== 消费者：OCR 识别处理 ==========
@Component
@Slf4j
public class OcrConsumer {
    
    @Autowired
    private OcrService ocrService;
    
    @Autowired
    private WebSocketService webSocketService;
    
    // ★ 手动 ACK + 重试机制
    @RabbitListener(queues = RabbitMQConfig.OCR_QUEUE)
    public void handleOcrTask(OcrTask task, Channel channel, 
                              @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) {
        try {
            log.info("开始 OCR 识别，合同ID: {}, 文件: {}", task.getContractId(), task.getFileUrl());
            
            // 1. 幂等性检查（防止重复消费）
            String messageId = UUID.randomUUID().toString(); // 实际应从消息头获取
            Boolean processed = redisTemplate.opsForValue()
                .setIfAbsent("ocr:processed:" + task.getContractId(), "1", 1, TimeUnit.DAYS);
            if (Boolean.FALSE.equals(processed)) {
                log.info("OCR 任务已处理过，跳过: {}", task.getContractId());
                channel.basicAck(deliveryTag, false);
                return;
            }
            
            // 2. 执行 OCR 识别（耗时操作）
            OcrResult result = ocrService.recognize(task.getFileUrl());
            
            // 3. 处理结果写入数据库
            ocrService.saveResult(task.getContractId(), result);
            
            // 4. ★ WebSocket 通知前端 OCR 完成
            webSocketService.sendNotification(
                task.getContractId(),
                new Notification("OCR", "OCR识别完成")
            );
            
            // 5. 手动 ACK（确认消费成功）
            channel.basicAck(deliveryTag, false);
            
        } catch (Exception e) {
            log.error("OCR 处理失败: {}", task, e);
            
            try {
                // ★ 重试机制：requeue = false 不重新入队
                // 实际应记录到死信队列或数据库，人工介入
                channel.basicNack(deliveryTag, false, false);
            } catch (IOException ex) {
                log.error("ACK 失败", ex);
            }
        }
    }
}
```

### [掌握] 8.3 Spring AOP 统一操作日志

```java
// ========== 自定义操作日志注解 ==========
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OperationLog {
    /** 操作模块 */
    String module() default "";
    
    /** 操作类型（CREATE/UPDATE/DELETE/APPROVE/REJECT） */
    String type() default "";
    
    /** 操作描述（支持 SpEL） */
    String description() default "";
}

// ========== AOP 切面 ==========
@Aspect
@Component
@Slf4j
public class OperationLogAspect {
    
    @Autowired
    private OperationLogMapper logMapper;
    
    @Around("@annotation(operationLog)")
    public Object around(ProceedingJoinPoint joinPoint, OperationLog operationLog) throws Throwable {
        // 1. 记录操作前状态
        long startTime = System.currentTimeMillis();
        
        // 2. 获取方法参数（用于 SpEL 解析）
        Object[] args = joinPoint.getArgs();
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        
        // 3. 执行目标方法
        Object result = null;
        boolean success = true;
        String errorMsg = null;
        
        try {
            result = joinPoint.proceed();
            return result;
        } catch (Exception e) {
            success = false;
            errorMsg = e.getMessage();
            throw e;
        } finally {
            // 4. 异步记录日志
            long executionTime = System.currentTimeMillis() - startTime;
            LogRecord record = new LogRecord();
            record.setModule(operationLog.module());
            record.setType(operationLog.type());
            record.setDescription(parseExpression(operationLog.description(), joinPoint));
            record.setSuccess(success);
            record.setErrorMsg(errorMsg);
            record.setExecutionTime(executionTime);
            record.setOperator(SecurityContextHolder.getContext().getAuthentication().getName());
            record.setCreateTime(LocalDateTime.now());
            
            // 异步写入（不阻塞主流程）
            CompletableFuture.runAsync(() -> logMapper.insert(record));
        }
    }
    
    private String parseExpression(String expression, ProceedingJoinPoint joinPoint) {
        // 可以使用 Spring Expression Language (SpEL) 解析
        // 简化实现：直接返回
        return expression;
    }
}

// ========== 使用示例 ==========
@RestController
@RequestMapping("/api/contracts")
public class ContractController {
    
    @OperationLog(
        module = "合同管理",
        type = "APPROVE",
        description = "审批合同：#contractNo"
    )
    @PostMapping("/{id}/approve")
    public Result approve(@PathVariable Long id) {
        return contractService.approve(id);
    }
    
    @OperationLog(
        module = "合同管理",
        type = "UPDATE",
        description = "编辑合同"
    )
    @PutMapping("/{id}")
    public Result update(@PathVariable Long id, @RequestBody ContractDTO dto) {
        return contractService.update(id, dto);
    }
}
```

### [掌握] 8.4 Stream + Lambda 数据处理

```java
// ========== Stream API 处理审批记录 ==========
@Service
public class ApproveRecordService {
    
    @Autowired
    private ApproveRecordMapper recordMapper;
    
    /**
     * 获取某个合同的审批历程（按时间排序，过滤无效记录）
     */
    public List<ApproveRecordVO> getApproveHistory(Long contractId) {
        List<ApproveRecord> records = recordMapper.selectByContractId(contractId);
        
        return records.stream()
            // 1. 过滤：只保留有效和最终的记录
            .filter(r -> r.getStatus() != ApproveStatus.DELETED)
            // 2. 排序：按操作时间升序
            .sorted(Comparator.comparing(ApproveRecord::getOperateTime))
            // 3. 转换为 VO
            .map(this::toVO)
            // 4. 收集结果
            .collect(Collectors.toList());
    }
    
    /**
     * 批量统计合同审批效率
     */
    public Map<Long, Long> getApproveEfficiency(List<Long> contractIds) {
        List<ApproveRecord> records = recordMapper.selectByContractIds(contractIds);
        
        return records.stream()
            // 1. 按合同 ID 分组
            .collect(Collectors.groupingBy(
                ApproveRecord::getContractId,
                // 2. 计算每个合同的审批总耗时
                Collectors.summingLong(r -> 
                    Duration.between(r.getStartTime(), r.getEndTime()).toMinutes()
                )
            ));
    }
    
    // ========== 使用 parallelStream 处理大数据 ==========
    /**
     * 生成审批报表（大批量数据并行处理）
     */
    public ReportVO generateReport(LocalDateTime start, LocalDateTime end) {
        List<ApproveRecord> records = recordMapper.selectByTimeRange(start, end);
        
        // ★ 使用 parallelStream 并行处理
        Map<String, Long> stats = records.parallelStream()
            .collect(Collectors.groupingByConcurrent(
                r -> r.getApprover().getDepartment(),
                Collectors.counting()
            ));
        
        // ★ 聚合计算
        DoubleSummaryStatistics timeStats = records.parallelStream()
            .mapToLong(r -> Duration.between(r.getStartTime(), r.getEndTime()).toMinutes())
            .summaryStatistics();
        
        return ReportVO.builder()
                .totalCount(records.size())
                .avgTimeMinutes(timeStats.getAverage())
                .maxTimeMinutes(timeStats.getMax())
                .minTimeMinutes(timeStats.getMin())
                .departmentStats(stats)
                .build();
    }
}
```

---

## 9. MySQL 与 Redis 必会代码

### [掌握] 9.1 MySQL 索引优化

```sql
-- ========== 联合索引（最左前缀原则）==========
-- 创建联合索引
CREATE INDEX idx_status_type_time ON contract (status, type, create_time);

-- 走索引的查询：
SELECT * FROM contract WHERE status = 1;                          -- ✅ status
SELECT * FROM contract WHERE status = 1 AND type = 'A';          -- ✅ status + type
SELECT * FROM contract WHERE status = 1 AND create_time > '2024'; -- ✅ status + create_time
SELECT * FROM contract WHERE status = 1 AND type = 'A' AND create_time > '2024'; -- ✅ 全匹配

-- 不走索引的查询：
SELECT * FROM contract WHERE type = 'A';              -- ❌ 跳过了 status
SELECT * FROM contract WHERE create_time > '2024';   -- ❌ 跳过了 status
SELECT * FROM contract WHERE type = 'A' AND create_time > '2024'; -- ❌ 跳过了 status

-- ========== 分页优化（大偏移量）==========
-- 传统分页（越往后越慢）
SELECT * FROM contract ORDER BY id LIMIT 100000, 20; -- 前面 100000 条都扫描了

-- 优化：子查询方式
SELECT * FROM contract 
WHERE id > (SELECT id FROM contract ORDER BY id LIMIT 100000, 1) 
ORDER BY id LIMIT 20;

-- 优化：记录上次最后 ID
SELECT * FROM contract WHERE id > #{lastId} ORDER BY id LIMIT 20;

-- ========== EXPLAIN 解读 ==========
EXPLAIN SELECT * FROM contract WHERE status = 1 ORDER BY create_time DESC LIMIT 20;
-- type: ref（命中索引）
-- possible_keys: idx_status_type_time
-- key: idx_status_type_time
-- rows: 500（扫描行数）
-- Extra: Using where; Using index（覆盖索引，无需回表）

-- ========== Covering Index（覆盖索引，避免回表）==========
-- 如果查询只需要 select 中的字段 包含在索引中
-- Extra 显示 "Using index" → 无需回表，性能最好

-- ========== 大事务避免 ==========
-- ❌ 错误：事务中做耗时操作
@Transactional
public void processContract(Long id) {
    Contract c = contractMapper.selectById(id);          -- 1. 查询
    // 调用外部 OCR 接口（网络 I/O，可能几秒到几十秒）  -- 2. 耗时操作（不该在事务内）
    OcrResult result = ocrService.recognize(c.getFileUrl());
    c.setOcrResult(result);
    contractMapper.updateById(c);                         -- 3. 更新
}
-- → 事务中持有数据库连接几十秒 → 连接池耗尽 → 系统崩溃

-- ✅ 正确做法：事务只处理数据更新
public void processContract(Long id) {
    // 1. 先查
    Contract c = contractMapper.selectById(id);
    // 2. 调用外部服务（不在事务中）
    OcrResult result = ocrService.recognize(c.getFileUrl());
    // 3. 更新（开启事务）
    updateContractWithOcr(id, result);
}

@Transactional
public void updateContractWithOcr(Long id, OcrResult result) {
    Contract c = contractMapper.selectById(id);
    c.setOcrResult(result);
    contractMapper.updateById(c);
}
```

### [掌握] 9.2 MVCC 与隔离级别

```sql
-- ========== MVCC 原理 ==========
-- InnoDB 每一行记录隐含两个字段：
-- DB_TRX_ID：最近修改的事务 ID
-- DB_ROLL_PTR：回滚指针，指向 Undo Log 中的上一个版本

-- ReadView（读视图）决定可见性：
-- RR（可重复读）级别：事务开始时创建 ReadView，整个事务期间重用
-- RC（读已提交）级别：每次 SELECT 都创建新的 ReadView

-- ReadView 包含：
-- m_ids：当前活跃的事务 ID 列表
-- min_trx_id：最小活跃事务 ID
-- max_trx_id：下一个要分配的事务 ID
-- creator_trx_id：创建 ReadView 的事务 ID

-- 可见性判断：
-- 如果 DB_TRX_ID < min_trx_id → 已提交，可见
-- 如果 DB_TRX_ID >= max_trx_id → 未来事务，不可见
-- 如果 min_trx_id <= DB_TRX_ID < max_trx_id → 在活跃列表中？不在则可见

-- ========== 间隙锁（Gap Lock）==========
-- RR 级别下，范围查询会锁住间隙，防止幻读
-- 例：SELECT * FROM contract WHERE status = 1 FOR UPDATE;
-- InnoDB 不仅锁住 status=1 的行，还会锁住 status=1 之间的间隙
-- 其他事务无法插入 status=1 的新行
```

### [掌握] 9.3 Redis 缓存实战

```java
// ========== 缓存穿透解决方案 ==========
@Service
public class ContractCacheService {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    @Autowired
    private ContractMapper contractMapper;
    
    // 方案一：缓存空值（短TTL）
    public Contract getContractById(Long id) {
        // 1. 查缓存
        String json = redisTemplate.opsForValue().get("contract:" + id);
        if (json != null) {
            // ★ 判断是否是空值标记
            if ("NULL".equals(json)) {
                return null; // 缓存了空值，直接返回 null
            }
            return JSON.parseObject(json, Contract.class);
        }
        
        // 2. 查数据库
        Contract contract = contractMapper.selectById(id);
        
        // 3. ★ 缓存空值（防止缓存穿透）
        if (contract == null) {
            redisTemplate.opsForValue().set(
                "contract:" + id, 
                "NULL", 
                30, TimeUnit.SECONDS // 空值缓存时间短
            );
            return null;
        }
        
        // 4. 缓存结果
        redisTemplate.opsForValue().set(
            "contract:" + id, 
            JSON.toJSONString(contract), 
            1, TimeUnit.HOURS
        );
        
        return contract;
    }
}

// ========== 缓存击穿解决方案 ==========
@Service
public class HotContractService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    // 方案：互斥锁 + 缓存预热
    public Contract getHotContract(Long id) {
        // 1. 查缓存
        String json = redisTemplate.opsForValue().get("hot:contract:" + id);
        if (json != null) {
            return JSON.parseObject(json, Contract.class);
        }
        
        // 2. 缓存未命中 → 尝试加锁
        RLock lock = redissonClient.getLock("hot:contract:lock:" + id);
        
        try {
            // ★ 尝试加锁（等待 200ms）
            boolean locked = lock.tryLock(0, 5, TimeUnit.SECONDS);
            if (!locked) {
                // 没获取到锁 → 说明其他线程在重建缓存
                // 短暂休眠后重试
                Thread.sleep(100);
                return getHotContract(id); // 递归重试
            }
            
            // 3. ★ 双重检查（获取锁后再次检查缓存，防止重复加载）
            json = redisTemplate.opsForValue().get("hot:contract:" + id);
            if (json != null) {
                return JSON.parseObject(json, Contract.class);
            }
            
            // 4. 查数据库
            Contract contract = contractMapper.selectById(id);
            
            // 5. 重建缓存（带逻辑过期时间）
            redisTemplate.opsForValue().set(
                "hot:contract:" + id,
                JSON.toJSONString(contract),
                30, TimeUnit.MINUTES
            );
            
            return contract;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return contractMapper.selectById(id); // 降级：直接查库
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}

// ========== 布隆过滤器（防止缓存穿透的最优解）==========
@Configuration
public class BloomFilterConfig {
    
    @Bean
    public RedissonClient redissonClient() {
        return Redisson.create();
    }
    
    // 初始化布隆过滤器
    @PostConstruct
    public void initBloomFilter() {
        RBloomFilter<Long> bloomFilter = redissonClient()
            .getBloomFilter("contract:bloom");
        
        // 预计数据量 10000，误判率 1%
        bloomFilter.tryInit(10000L, 0.01);
        
        // 预热：将数据库中所有合同 ID 加入布隆过滤器
        List<Long> allIds = contractMapper.selectAllIds();
        allIds.forEach(bloomFilter::add);
    }
}

@Service
public class BloomFilterService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    public Contract getContractById(Long id) {
        RBloomFilter<Long> bloomFilter = redissonClient
            .getBloomFilter("contract:bloom");
        
        // ★ 布隆过滤器判断不存在 → 直接返回（避免查缓存/数据库）
        if (!bloomFilter.contains(id)) {
            return null;
        }
        
        // 可能存在（有误判率），继续查缓存 → 数据库
        // ... 正常查询逻辑
    }
}
```

### [掌握] 9.4 Redis 分布式锁 Lua 脚本

```lua
-- ========== 加锁 Lua 脚本 ==========
-- KEYS[1] = 锁的 key
-- ARGV[1] = 锁的 value（UUID:ThreadId，用于安全释放）
-- ARGV[2] = 过期时间（毫秒）

-- 1. 尝试设置 key（NX：不存在时才设置，PX：毫秒过期）
if redis.call('SET', KEYS[1], ARGV[1], 'NX', 'PX', ARGV[2]) then
    return 1  -- 加锁成功
end

-- 2. 可重入检查（同一线程再次加锁）
if redis.call('GET', KEYS[1]) == ARGV[1] then
    -- 重入次数 +1（需要额外存储重入计数）
    redis.call('HINCRBY', KEYS[1] .. ':count', ARGV[1], 1)
    -- 续期（刷新过期时间）
    redis.call('PEXPIRE', KEYS[1], ARGV[2])
    redis.call('PEXPIRE', KEYS[1] .. ':count', ARGV[2])
    return 1
end

return 0  -- 加锁失败

-- ========== 解锁 Lua 脚本 ==========
-- KEYS[1] = 锁的 key
-- ARGV[1] = 锁的 value（用于校验是否是自己的锁）

-- 1. 检查是否自己的锁
if redis.call('GET', KEYS[1]) ~= ARGV[1] then
    return 0  -- 不是自己的锁，不能释放
end

-- 2. 检查是否有重入计数
local count = redis.call('HGET', KEYS[1] .. ':count', ARGV[1])
if count and tonumber(count) > 1 then
    -- 还有重入次数，减一
    redis.call('HINCRBY', KEYS[1] .. ':count', ARGV[1], -1)
    -- 续期
    redis.call('PEXPIRE', KEYS[1], 30000)
    redis.call('PEXPIRE', KEYS[1] .. ':count', 30000)
    return 2  -- 重入减一
end

-- 3. 完全释放
redis.call('DEL', KEYS[1])
redis.call('DEL', KEYS[1] .. ':count')
return 1  -- 解锁成功
```

---

## 10. 微服务与分布式代码

### [掌握] 10.1 Nacos 服务注册与发现

```yaml
# ========== application.yml（Nacos 客户端配置）==========
spring:
  application:
    name: contract-service
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER:localhost:8848}
        namespace: ${NACOS_NAMESPACE:prod}
        group: DEFAULT_GROUP
      config:
        server-addr: ${NACOS_SERVER:localhost:8848}
        namespace: ${NACOS_NAMESPACE:prod}
        group: DEFAULT_GROUP
        file-extension: yaml
        # 共享配置（多个微服务共用）
        shared-configs:
          - data-id: common-database.yaml
            refresh: true
          - data-id: common-redis.yaml
            refresh: true
```

### [掌握] 10.2 OpenFeign 远程调用

```java
// ========== Feign 客户端 ==========
@FeignClient(
    name = "user-service",
    path = "/api/users",
    fallbackFactory = UserClientFallbackFactory.class
)
public interface UserClient {
    
    @GetMapping("/{id}")
    Result<UserDTO> getUserById(@PathVariable("id") Long id);
    
    @PostMapping("/batch")
    Result<List<UserDTO>> getUsersByIds(@RequestBody List<Long> ids);
    
    @GetMapping("/department/{deptId}")
    Result<List<UserDTO>> getUsersByDepartment(@PathVariable("deptId") Long deptId);
}

// ========== Feign 降级处理 ==========
@Component
public class UserClientFallbackFactory implements FallbackFactory<UserClient> {
    
    @Override
    public UserClient create(Throwable cause) {
        return new UserClient() {
            
            @Override
            public Result<UserDTO> getUserById(Long id) {
                log.error("获取用户信息失败，userId: {}", id, cause);
                return Result.error("用户服务暂时不可用");
            }
            
            @Override
            public Result<List<UserDTO>> getUsersByIds(List<Long> ids) {
                log.error("批量获取用户信息失败", cause);
                return Result.error("用户服务暂时不可用");
            }
            
            @Override
            public Result<List<UserDTO>> getUsersByDepartment(Long deptId) {
                log.error("获取部门用户失败", cause);
                return Result.error("用户服务暂时不可用");
            }
        };
    }
}

// ========== Feign 拦截器（传递 Token）==========
@Component
public class FeignTokenInterceptor implements RequestInterceptor {
    
    @Override
    public void apply(RequestTemplate template) {
        // 从请求上下文中获取 JWT Token
        ServletRequestAttributes attributes = (ServletRequestAttributes) 
            RequestContextHolder.getRequestAttributes();
        if (attributes != null) {
            String token = attributes.getRequest()
                .getHeader("Authorization");
            if (StringUtils.hasText(token)) {
                template.header("Authorization", token);
            }
            // 传递租户 ID
            String tenantId = attributes.getRequest()
                .getHeader("X-Tenant-Id");
            if (StringUtils.hasText(tenantId)) {
                template.header("X-Tenant-Id", tenantId);
            }
        }
    }
}
```

### [掌握] 10.3 Spring Cloud Gateway 网关

```yaml
# ========== application.yml（Gateway 配置）==========
spring:
  cloud:
    gateway:
      routes:
        # 用户服务
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@userKeyResolver}"
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
        
        # 合同服务
        - id: contract-service
          uri: lb://contract-service
          predicates:
            - Path=/api/contracts/**
          filters:
            - StripPrefix=1
        
        # 订单服务
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
      
      # 全局过滤器
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Origin
```

### [掌握] 10.4 RabbitMQ 消息可靠性

```java
// ========== 消息确认机制配置 ==========
@Configuration
public class RabbitMQConfirmConfig {
    
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        
        // ★ 确认模式：消息到达交换机后回调
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息已到达交换机: {}", correlationData.getId());
            } else {
                log.error("消息未到达交换机: {}, cause: {}", correlationData.getId(), cause);
                // 记录到数据库，定时任务重发
                messageRepository.markFailed(correlationData.getId());
            }
        });
        
        // ★ 回退模式：消息无法路由到队列时回调
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("消息无法路由到队列: exchange={}, routingKey={}, replyCode={}, replyText={}",
                returned.getExchange(), returned.getRoutingKey(),
                returned.getReplyCode(), returned.getReplyText());
            // 记录到数据库，人工介入
            messageRepository.markUnrouted(returned.getMessage().getMessageProperties().getMessageId());
        });
        
        return rabbitTemplate;
    }
}

// ========== 幂等消费者示例 ==========
@Component
public class IdempotentConsumer {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    @RabbitListener(queues = "contract.notify.queue")
    public void handleNotification(NotificationMessage message, Channel channel, 
                                    @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {
        
        String messageId = message.getMessageId();
        
        // ★ 幂等性检查：利用 Redis SETNX
        Boolean success = redisTemplate.opsForValue()
            .setIfAbsent("msg:processed:" + messageId, "1", 1, TimeUnit.HOURS);
        
        if (Boolean.FALSE.equals(success)) {
            log.info("消息已消费过，幂等跳过: {}", messageId);
            channel.basicAck(tag, false); // ACK 确认
            return;
        }
        
        try {
            // 处理消息
            processNotification(message);
            channel.basicAck(tag, false);
        } catch (Exception e) {
            log.error("消息处理失败: {}", messageId, e);
            // requeue=false 不重新入队（避免死循环）
            // 实际应发送到死信队列
            channel.basicNack(tag, false, false);
        }
    }
}
```

---

## 11. 前端 Vue3 必会代码

### [掌握] 11.1 Composition API（setup 语法）

```vue
<!-- ========== Vue3 <script setup> 基础 ========== -->
<script setup>
import { ref, reactive, computed, onMounted, watch } from 'vue'
import { useUserStore } from '@/stores/user'
import { ElMessage } from 'element-plus'

// 1. 基本响应式
const count = ref(0)           // 基本类型
const user = reactive({        // 对象类型
  name: '张三',
  age: 25
})

// 2. 计算属性
const doubleCount = computed(() => count.value * 2)
const isAdult = computed(() => user.age >= 18)

// 3. 监听器
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// 4. 生命周期
onMounted(() => {
  fetchData()
})

// 5. 方法
function increment() {
  count.value++
}

function updateUser() {
  user.name = '李四'
  user.age = 30
}

// 6. 使用 Pinia store
const userStore = useUserStore()
const userInfo = computed(() => userStore.userInfo)
</script>

<template>
  <div>
    <p>计数：{{ count }}（双倍：{{ doubleCount }}）</p>
    <p>用户：{{ user.name }}，{{ user.age }}岁</p>
    <p>{{ isAdult ? '成年人' : '未成年人' }}</p>
    <el-button @click="increment">+1</el-button>
    <el-button @click="updateUser">修改用户</el-button>
  </div>
</template>
```

### [掌握] 11.2 Pinia Store 完整示例

```javascript
// ========== stores/contract.js ==========
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import axios from '@/utils/request'

export const useContractStore = defineStore('contract', () => {
  // ===== State =====
  const contractList = ref([])
  const currentContract = ref(null)
  const loading = ref(false)
  const searchParams = ref({
    status: '',
    keyword: '',
    page: 1,
    pageSize: 20
  })
  const total = ref(0)
  
  // ===== Getters =====
  const pendingContracts = computed(() => 
    contractList.value.filter(c => c.status === 'PENDING_APPROVE')
  )
  
  const currentPageData = computed(() => ({
    list: contractList.value,
    total: total.value,
    page: searchParams.value.page,
    pageSize: searchParams.value.pageSize
  }))
  
  // ===== Actions =====
  async function fetchContractList() {
    loading.value = true
    try {
      const res = await axios.get('/api/contracts/list', {
        params: searchParams.value
      })
      if (res.data.code === 200) {
        contractList.value = Object.freeze(
          res.data.data.records.map(item => Object.freeze(item))
        )
        total.value = res.data.data.total
      }
    } finally {
      loading.value = false
    }
  }
  
  async function fetchContractDetail(id) {
    const res = await axios.get(`/api/contracts/${id}`)
    if (res.data.code === 200) {
      currentContract.value = res.data.data
    }
    return currentContract.value
  }
  
  function setSearchParams(params) {
    searchParams.value = { ...searchParams.value, ...params }
  }
  
  // ===== Return =====
  return {
    // state
    contractList, currentContract, loading, searchParams, total,
    // getters
    pendingContracts, currentPageData,
    // actions
    fetchContractList, fetchContractDetail, setSearchParams
  }
})
```

### [掌握] 11.3 组件通信

```vue
<!-- ========== 父组件 ========== -->
<script setup>
import { ref, provide } from 'vue'
import ChildComponent from './ChildComponent.vue'
import GrandchildComponent from './GrandchildComponent.vue'

const message = ref('来自父组件的消息')
const formData = ref({ name: '', age: '' })

// provide：跨级传递（父 → 孙）
provide('theme', 'dark')
provide('updateForm', (data) => {
  formData.value = { ...formData.value, ...data }
})

function handleChildEvent(data) {
  console.log('收到子组件事件：', data)
}
</script>

<template>
  <ChildComponent
    :message="message"
    :form-data="formData"
    @update="handleChildEvent"
    v-model:count="count"
  />
</template>

<!-- ========== 子组件 ========== -->
<script setup>
const props = defineProps({
  message: String,
  formData: Object
})

const emit = defineEmits(['update', 'update:count'])

// 暴露给父组件的方法
defineExpose({
  validate() { /* ... */ },
  reset() { /* ... */ }
})
</script>

<!-- ========== 孙组件（使用 inject 接收 provide）========== -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme')           // 接收父组件的 provide
const updateForm = inject('updateForm') // 调用父组件的更新方法
</script>
```

### [掌握] 11.4 Vue3 响应式原理

```javascript
// ========== Vue3 Proxy 响应式原理 ==========
// Vue2：Object.defineProperty → 只能拦截已有属性，无法检测新增/删除
// Vue3：Proxy → 可以拦截所有操作（get/set/has/deleteProperty 等）

// 简化版 reactive 实现
function reactive(target) {
  if (target === null || typeof target !== 'object') {
    return target
  }
  
  // 已经代理过的对象直接返回
  if (target.__isProxy) return target
  
  const handler = {
    get(target, key, receiver) {
      // 依赖收集（track）
      track(target, key)
      
      // 递归处理嵌套对象
      const result = Reflect.get(target, key, receiver)
      if (typeof result === 'object' && result !== null) {
        return reactive(result)
      }
      return result
    },
    
    set(target, key, value, receiver) {
      const oldValue = target[key]
      const result = Reflect.set(target, key, value, receiver)
      
      // 触发更新（trigger）
      if (oldValue !== value) {
        trigger(target, key)
      }
      return result
    },
    
    deleteProperty(target, key) {
      const hasKey = key in target
      const result = Reflect.deleteProperty(target, key)
      if (hasKey) {
        trigger(target, key) // ✅ 能检测到删除
      }
      return result
    }
  }
  
  const proxy = new Proxy(target, handler)
  proxy.__isProxy = true
  return proxy
}
```

---

## 12. DevOps 与部署

### [掌握] 12.1 Spring Boot 打包部署

```bash
# ========== 1. Maven 打包 ==========
mvn clean package -DskipTests -Pprod

# ========== 2. 部署脚本（deploy.sh）==========
#!/bin/bash
APP_NAME="contract-service"
JAR_NAME="${APP_NAME}.jar"
DEPLOY_DIR="/opt/app/${APP_NAME}"
BACKUP_DIR="${DEPLOY_DIR}/backup"
JAR_PATH="${DEPLOY_DIR}/${JAR_NAME}"

# 备份旧版本
if [ -f "${JAR_PATH}" ]; then
    timestamp=$(date +%Y%m%d_%H%M%S)
    cp ${JAR_PATH} ${BACKUP_DIR}/${JAR_NAME}.${timestamp}
    echo "已备份至 ${BACKUP_DIR}/${JAR_NAME}.${timestamp}"
fi

# 复制新版本
cp ${JAR_NAME} ${JAR_PATH}

# 停止旧服务
PID=$(ps -ef | grep ${JAR_NAME} | grep -v grep | awk '{print $2}')
if [ -n "$PID" ]; then
    echo "停止旧服务 PID: $PID"
    kill -15 $PID
    sleep 5
    # 强制停止
    if ps -p $PID > /dev/null; then
        kill -9 $PID
    fi
fi

# 启动新服务
nohup java -jar ${JAR_PATH} \
    --spring.profiles.active=prod \
    -Xms512m -Xmx1024m \
    -XX:+UseG1GC \
    -XX:MaxGCPauseMillis=200 \
    -Xlog:gc*:file=${DEPLOY_DIR}/logs/gc.log:time,pid \
    > ${DEPLOY_DIR}/logs/startup.log 2>&1 &

echo "服务已启动，日志路径：${DEPLOY_DIR}/logs/"
```

### [掌握] 12.2 Vue 打包部署

```bash
# ========== 1. 构建生产包 ==========
npm install
npm run build:prod
# 生成 dist/ 目录

# ========== 2. Nginx 配置 ==========
# 见 7.5 节完整配置

# ========== 3. 部署命令 ==========
scp -r dist/* root@server:/usr/share/nginx/html/
```

### [掌握] 12.3 Docker 部署

```dockerfile
# ========== Dockerfile（多阶段构建）==========
# 第一阶段：构建
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# 第二阶段：运行
FROM openjdk:17-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# 时区设置
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### [掌握] 12.4 Jenkins Pipeline

```groovy
// ========== Jenkinsfile ==========
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        PROJECT_NAME = 'contract-service'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_REGISTRY}/${PROJECT_NAME}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_REGISTRY}/${PROJECT_NAME}:${BUILD_NUMBER} ${DOCKER_REGISTRY}/${PROJECT_NAME}:latest
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    docker push ${DOCKER_REGISTRY}/${PROJECT_NAME}:${BUILD_NUMBER}
                    docker push ${DOCKER_REGISTRY}/${PROJECT_NAME}:latest
                    # 通知 K8s 滚动更新
                    kubectl set image deployment/${PROJECT_NAME} \
                        app=${DOCKER_REGISTRY}/${PROJECT_NAME}:${BUILD_NUMBER} \
                        --record
                '''
            }
        }
    }
    
    post {
        failure {
            emailext(
                subject: "构建失败: ${PROJECT_NAME} #${BUILD_NUMBER}",
                body: "项目 ${PROJECT_NAME} 构建失败，请检查。",
                to: 'dev@example.com'
            )
        }
    }
}
```

---

## 13. 算法与数据结构手写代码

### [掌握] 13.1 排序算法

```java
// ========== 快速排序（面试最常考）==========
public class QuickSort {
    
    public void quickSort(int[] arr, int left, int right) {
        if (left >= right) return;
        
        // 分区
        int pivotIndex = partition(arr, left, right);
        
        // 递归排序左右两部分
        quickSort(arr, left, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, right);
    }
    
    private int partition(int[] arr, int left, int right) {
        // 选择最右边的元素作为基准
        int pivot = arr[right];
        int i = left - 1; // i 指向小于 pivot 的最后一个元素
        
        for (int j = left; j < right; j++) {
            if (arr[j] < pivot) {
                i++;
                swap(arr, i, j);
            }
        }
        
        // 将 pivot 放到正确位置
        swap(arr, i + 1, right);
        return i + 1;
    }
    
    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

// ========== 归并排序 ==========
public class MergeSort {
    
    public void mergeSort(int[] arr, int left, int right) {
        if (left >= right) return;
        
        int mid = left + (right - left) / 2;
        
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
    
    private void merge(int[] arr, int left, int mid, int right) {
        int[] temp = new int[right - left + 1];
        int i = left, j = mid + 1, k = 0;
        
        // 合并两个有序数组
        while (i <= mid && j <= right) {
            if (arr[i] <= arr[j]) {
                temp[k++] = arr[i++];
            } else {
                temp[k++] = arr[j++];
            }
        }
        
        while (i <= mid) temp[k++] = arr[i++];
        while (j <= right) temp[k++] = arr[j++];
        
        // 复制回原数组
        System.arraycopy(temp, 0, arr, left, temp.length);
    }
}
```

### [掌握] 13.2 常见数据结构

```java
// ========== 反转链表 ==========
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    
    return prev;
}

// ========== 判断链表是否有环 ==========
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) return false;
    
    ListNode slow = head;
    ListNode fast = head.next;
    
    while (slow != fast) {
        if (fast == null || fast.next == null) return false;
        slow = slow.next;
        fast = fast.next.next;
    }
    
    return true;
}

// ========== 二叉树遍历 ==========
// 前序遍历（递归）
public void preorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    result.add(root.val);
    preorder(root.left, result);
    preorder(root.right, result);
}

// 前序遍历（迭代）
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        result.add(node.val);
        if (node.right != null) stack.push(node.right);
        if (node.left != null) stack.push(node.left);
    }
    
    return result;
}

// ========== LRU 缓存（LinkedHashMap 实现）===========
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder=true 开启访问顺序
        this.capacity = capacity;
    }
    
    public int get(int key) {
        return super.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        super.put(key, value);
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}

// ========== 手写 LRU（HashMap + 双向链表）===========
class LRUCacheManual {
    private final Map<Integer, Node> map;
    private final int capacity;
    private final Node head; // 虚拟头节点
    private final Node tail; // 虚拟尾节点
    
    class Node {
        int key, value;
        Node prev, next;
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    public LRUCacheManual(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.head = new Node(0, 0);
        this.tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        Node node = map.get(key);
        if (node == null) return -1;
        moveToHead(node); // 最近访问，移到头部
        return node.value;
    }
    
    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
        } else {
            node = new Node(key, value);
            map.put(key, node);
            addToHead(node);
            if (map.size() > capacity) {
                Node removed = removeTail();
                map.remove(removed.key);
            }
        }
    }
    
    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
    
    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }
    
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private Node removeTail() {
        Node node = tail.prev;
        removeNode(node);
        return node;
    }
}
```

### [掌握] 13.3 常见算法题

```java
// ========== 二分查找 ==========
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;
}

// ========== 最长不重复子串（滑动窗口）===========
public int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int maxLen = 0, left = 0;
    
    for (int right = 0; right < s.length(); right++) {
        // 如果窗口内有重复字符，移动左指针
        while (window.contains(s.charAt(right))) {
            window.remove(s.charAt(left));
            left++;
        }
        window.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    
    return maxLen;
}

// ========== 两数之和 ==========
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    
    return new int[]{-1, -1};
}
```

---

## 14. AI Agent 辅助开发实践

### [掌握] 14.1 高效 Prompt 工程

```
# ========== 代码生成 Prompt 模板 ==========

## 角色
你是一位资深的 Java 后端工程师，精通 Spring Boot、微服务架构和系统设计。

## 任务
请帮我生成一个 [功能名称] 的完整代码。

## 需求
1. 功能描述：[详细描述]
2. 技术栈：Spring Boot 3 + MyBatis-Plus + Redis
3. 表结构：[附上表结构]
4. 接口清单：
   - POST /api/xxx/create
   - GET /api/xxx/list
   - PUT /api/xxx/{id}
   - DELETE /api/xxx/{id}

## 约束
1. 使用统一的 Result 返回类
2. 参数校验使用 @Valid
3. 遵循 RESTful 风格
4. 包含异常处理

## 输出格式
1. Controller 类
2. Service 接口 + 实现类
3. Mapper 接口
4. Entity/DTO/VO 类
```

### [掌握] 14.2 调试 Prompt

```
# 调试 Prompt 模板

我遇到了以下 Bug：

## 问题描述
[描述问题的表现]

## 复现步骤
1. [步骤1]
2. [步骤2]

## 相关代码
```java
[贴相关代码]
```

## 报错信息
```
[贴报错堆栈]
```

## 尝试过的方案
1. [已尝试的方案和结果]

请分析可能的原因，并提供修复方案。
```

### [掌握] 14.3 架构设计 Prompt

```
# 架构设计 Prompt 模板

## 背景
[描述项目背景]

## 核心需求
1. [需求1]
2. [需求2]

## 非功能需求
1. QPS 峰值：[数字]
2. 数据量级：[数字]
3. 可用性要求：[百分比]

## 要求
请设计方案，包含：
1. 整体架构图（文字描述）
2. 核心数据模型
3. 关键接口设计
4. 性能优化方案
5. 可扩展性考虑
6. 潜在风险与应对
```

---

## 15. 系统设计练习题

### 问题 1：设计一个短链服务

**关键点**：
- 发号器（Snowflake / Redis INCR）
- Base62 编码（0-9a-zA-Z 共 62 位）
- 重定向（301 vs 302）
- 过期清理

```java
// ========== Base62 编码 ==========
public class Base62Converter {
    private static final String BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    
    public static String encode(long value) {
        StringBuilder sb = new StringBuilder();
        while (value > 0) {
            sb.append(BASE62.charAt((int) (value % 62)));
            value /= 62;
        }
        return sb.reverse().toString();
    }
    
    public static long decode(String str) {
        long result = 0;
        for (char c : str.toCharArray()) {
            result = result * 62 + BASE62.indexOf(c);
        }
        return result;
    }
}
```

### 问题 2：设计一个秒杀系统

**关键点**：
- 前端限流（按钮置灰 + 倒计时）
- 后端限流（Sentinel / 令牌桶）
- Redis 预减库存
- RabbitMQ 异步下单
- 数据库乐观锁扣减

```java
// ========== Redis 预减库存 ==========
public boolean tryDecrementStock(Long productId, int count) {
    String key = "seckill:stock:" + productId;
    // Lua 脚本：原子扣减，库存不足返回 -1
    String lua = """
        local stock = redis.call('GET', KEYS[1])
        if not stock or tonumber(stock) < tonumber(ARGV[1]) then
            return -1
        end
        return redis.call('DECRBY', KEYS[1], ARGV[1])
        """;
    Long result = redisTemplate.execute(
        new DefaultRedisScript<>(lua, Long.class),
        List.of(key), String.valueOf(count)
    );
    return result >= 0;
}
```

### 问题 3：设计一个消息推送系统

**关键点**：
- WebSocket 连接管理（userId → sessionId）
- 离线消息存储（Redis List）
- 消息去重（唯一 ID + 布隆过滤器）
- 推送确认（ACK 机制）
- 大规模推送（MQ 广播）

---

## 16. 面试 Checklist

### 技术准备

- [ ] **Java 基础**：HashMap 原理、ConcurrentHashMap 原理、线程池参数、AQS 原理
- [ ] **Spring Boot**：自动配置原理、启动流程、自定义 Starter
- [ ] **Spring Security**：过滤器链、JWT 无感刷新（滑动续期）
- [ ] **MyBatis-Plus**：多租户、分页、条件构造器
- [ ] **MySQL**：B+树索引、最左前缀、MVCC、间隙锁、SQL 优化
- [ ] **Redis**：数据结构、缓存穿透/击穿/雪崩、分布式锁（Redisson）
- [ ] **RabbitMQ**：消息可靠（Confirm/Return）、幂等、顺序消息
- [ ] **微服务**：Nacos 配置热更新、Sentinel 规则持久化、Feign 调用
- [ ] **WebSocket**：双通道、指数退避、断线重连
- [ ] **Vue3**：Composition API、响应式原理（Proxy vs defineProperty）、Pinia

### 项目准备

- [ ] **项目一（王牌）**：
  - JWT 无感刷新完整讲清楚（前端拦截器 + 后端过滤器 + 滑动续期）
  - 订单状态机设计模式
  - 竞价超卖三层防护
  - WebSocket 双通道 + 指数退避 + 轮询保底
- [ ] **项目二**：定时任务数据同步 + 批处理优化
- [ ] **项目三**：Nacos 热更新 + 多租户 + Object.freeze() 优化 + Sentinel 持久化 + try_files
- [ ] **项目四**：Redisson 分布式锁 + RabbitMQ 异步 + AOP 日志 + Stream 并行

### 行为面试

- [ ] **最大挑战**：WebSocket 的可靠性从简单单通道→双通道+指数退避+轮询保底
- [ ] **独立全栈**：一个人做完整项目，前端+后端+数据库+部署
- [ ] **AI 效率**：不只是使用工具，而是理解 Agent 原理，Prompt 工程
- [ ] **学习能力**：自学新技术的例子

### 反问问题

- [ ] 团队当前技术栈和未来规划？
- [ ] 项目的业务规模和技术挑战？
- [ ] 团队对技术成长的支持（Code Review、技术分享等）？

---

## 学习路线图

```
第一阶段：Java 核心（2周）
├── 集合框架源码（HashMap、ConcurrentHashMap）
├── 多线程与线程池
├── JVM 内存模型与 GC
└── 刷算法（链表、二叉树、排序、动态规划）

第二阶段：Spring 全家桶（2周）
├── Spring Boot 自动配置
├── Spring Security + JWT
├── MyBatis-Plus 高级
└── Spring AOP 实战

第三阶段：数据库与缓存（1周）
├── MySQL 索引优化 + EXPLAIN
├── MVCC + 锁机制
├── Redis 数据结构 + 缓存问题
└── Redis 分布式锁 + Lua

第四阶段：微服务与消息（1周）
├── Nacos 配置/注册中心
├── Sentinel 限流熔断
├── RabbitMQ 可靠消息
└── Docker + Jenkins

第五阶段：前端与部署（1周）
├── Vue3 Composition API
├── Pinia 状态管理
├── Nginx 配置
└── 项目打包上线

第六阶段：项目梳理（持续）
├── 每个项目 3 个核心技术点
├── 每个技术点能写代码演示
└── 能讲出为什么这样做（技术选型考量）
```

---

> 📝 **最后提醒**  
> 这份指南覆盖了简历中所有技术点的完整代码和面试回答。  
> 学习时请这样配合：  
> 1. **先理解原理**（为什么这样设计）  
> 2. **再写代码**（手敲每个代码示例）  
> 3. **再讲出来**（对着镜子模拟面试，用自己的话讲述）  
> 4. **再扩展**（基于代码做自己的改进）  
>
> 记住：面试官看重的不是你背了多少，而是**你是否真的理解并能灵活运用**。  
> 祝你面试顺利，拿到心仪的 Offer！💪
