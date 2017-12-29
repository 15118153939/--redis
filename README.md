# --redis
一些学习笔记，方便日后查看
#1．Linux安装Redis
```
Redis官网(http://www.redis.io/download)下载安装包，如：redis-4.0.6.tar.gz。
-->放入/opt目录下，解压：tar -zxvf redis-4.0.6.tar.gz 解压后出现文件夹：redis-4.0.6
-->进入redis-4.0.6目录：cd  redis-4.0.6 
-->执行命令：make（如果报jemalloc的错误,则加参数为：make MALLOC=libc）
-->执行make install
-->查看默认安装目录：cd /usr/local/bin
```

#2．Linux启动，关闭，连接客户端
```
redis-server 启动redis服务
ps -ef | grep redis 查看redis进程
redis-cli -h 127.0.0.1 -p 6379 -a pwd 连接redis，quit关闭连接(不关闭redis服务),shutdown(关闭redis服务)
如果是本机则不用带 -h，如果没有密码则不用带-a
redis-cli -p 6379 注：redis默认不是以守护进程启动的,不带端口号的则为默认的6379。
redis-cli --raw 中文以原编码格式显示，不会乱码。
```
3．Linux修改redis配置
```
以自己的配置文件启动redis。例如：
-->新建一个文件夹，路径为： /myredis
-->新建配置文件：touch redis.conf 
-->编辑配置文件，加入自己所需要的配置信息: vi redis.conf 
-->以指定的配置文件启动redis: redis-server /myredis/redis.conf 
```
常用配置项：
```
daemonize yes   	yes以守护进程方式启动redis
port 6379			端口号，默认6379
bind 127.0.0.1		绑定主机地址
timeout 300		客户端闲置多久后关闭(0为不关闭)
dbfilename dump.rdb 本地数据库文件名，默认dump.rdb
dir ./	      		指定本地数据库存放目录
pidfile /var/run/redis.pid 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
loglevel verbose	指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
logfile stdout		日志记录方式
databases 16		设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
save <seconds> <changes>	多长时间内，有多少次更新操作，就将数据同步到数据文件
Redis默认配置文件中提供了三个条件：
```
```
save 900 1
       save 300 10
       save 60 10000
分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改
rdbcompression yes 	存储至本地数据库时是否压缩数据，默认为yes
slaveof <masterip> <masterport>	设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
masterauth <master-password>	master服务设置了密码保护时，slav连接master的密码
requirepass 12345	设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
maxclients 128	设置同一时间最大客户端连接数，默认无限制
maxmemory <bytes>	指定Redis最大内存限制
appendonly no	指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
appendfilename appendonly.aof	指定更新日志文件名，默认为appendonly.aof
appendfsync everysec	指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折衷，默认值）
include /path/to/local.conf	指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
```

4．linux中Redis常用操作命令
```
key : 非二进制安全的字符类型,像“my key”和“mykey\n”包含空格和换行的是不允许的
Values ： Strings，Lists，Sets，Sorted sets，Hash
set key value 		设值(1表示成功，0表示失败)
get key			取值	
setnx key value 	如果key已经存在，返回0
mget key1 key2 ... keyN 一次获取多个key的值
mset key1 value1 ... keyN valueN 一次设置多个key的值，成功返回1表示所有的值都设置了，失败返回0表示没有任何值被设置
msetnx key1 value1 ... keyN valueN 同上，但是不会覆盖已经存在的key
getset key value  设置key的值，并返回key的旧值，如果key不存在返回nil
EXITS key 		是否存在(1存在，0不存在)
del key1 key2 ....keyN  删除给定key,返回删除key的数目
keys *			匹配某key名
type key 			返回给定key的value类型
rename oldkey newkey 原子的重命名一个key,如果newkey存在，将会被覆盖，返回1表示成功，0失败。失败可能是oldkey不存在或者和newkey相同
renamenx oldkey newkey 同上，但是如果newkey存在返回失败
dbsize 返回当前数据库的key数量
expire key seconds 为key指定过期时间，单位是秒。返回1成功，0表示key已经设置过过期时间或者不存在
ttl key 返回设置了过期时间的key的剩余过期秒数， -1表示key不存在或者没有设置过过期时间
flushdb 删除当前数据库中所有key,此方法不会失败。慎用
flushall 删除所有数据库中的所有key，此方法不会失败。更加慎用
```
5．Jedis连接操作Redis
要引入 redis.clients和commons-pool的jar包，可通过maven添加相关依赖
操作redis主要三种模式：
Jedis (JedisPool连接池)：单服务器模式。
ShardedJedis (ShardedJedisPool连接池)：分片模式，一种轻量级集群，是redis没有集群功能之前客户端实现的一个数据分布式方案。使用的是JedisShardInfo的instance的顺序或者name来做的一致性哈希。
JedisCluster (Factory实现)：集群模式，redis3.0提供集群之后，提供的集群方案。 使用的是CRC16算法来做的哈希槽。不支持带密码连接redis，如果要带密码连接则要改源码。
 spring-data-redis整合JedisPool:
核心操作类是RedisTemplate<K, V>，RedisCacheManager,CompositeCacheManager等，不支持集群
JedisPool方式就不写了，一般都用集群(连接池)
方法一：XML注入ShardedJedisPool
```
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxActive" value="${redis.conf.maxActive}" />
		<property name="maxIdle" value="${redis.conf.maxIdle}" />
		<property name="maxWait" value="${redis.conf.maxWait}" />
		<property name="testOnBorrow" value="${redis.conf.testOnBorrow}" />
	</bean>
	<bean id="shardedJedisPool" class="redis.clients.jedis.ShardedJedisPool">
		<constructor-arg index="0" ref="jedisPoolConfig" />
		<constructor-arg index="1">
			<list>
				<bean class="redis.clients.jedis.JedisShardInfo">
					<constructor-arg index="0" value="${redis.host1}" />
					<constructor-arg index="1" value="${redis.port1}" type="int" />
					<constructor-arg index="2" value="${redis.timeout1}" />
					<property name="password" value="${redis.password1}"></property>
				</bean>
				<bean class="redis.clients.jedis.JedisShardInfo">
					<constructor-arg index="0" value="${redis.host2}" />
					<constructor-arg index="1" value="${redis.port2}" type="int" />
					<constructor-arg index="2" value="${redis.timeout2}" />
					<property name="password" value="${redis.password2}"></property>
				</bean>
			</list>
		</constructor-arg>
	</bean>
  ```
 JedisPoolConfig.maxActive: 可用连接实例的最大数目，默认值为8,-1表示不限制
 JedisPoolConfig.maxIdle: 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8
 JedisPoolConfig.maxWait: 等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时   
 JedisPoolConfig.maxActive: true获取 jedis实例时，提前进行validate,得到的jedis实例均是可用的
 (新版本中maxIdle改名为maxIdle，maxWait改名为maxWaitMillis)
 ShardedJedisPool的下标为1的构造方法参数中，有多少个集群服务器，就配多少个JedisShardInfo
 JedisShardInfo.构造方法参数0,1,2分别为:  ip，端口，超时时间毫秒
 JedisShardInfo.password: 如果有密码则配

然后可以java代码中注入ShardedJedisPool 连接池实例，并通过连接池获取 ShardedJedis 实例使用和回收
@Resource(name="shardedJedisPool")
private ShardedJedisPool shardedJedisPool;

方法二：代码中配置ShardedJedisPool 
```
	JedisPoolConfig config = new JedisPoolConfig();
	config.setMaxTotal(MAX_ACTIVE);
	config.setMaxIdle(MAX_IDLE);
	config.setMaxWaitMillis(MAX_WAIT);
	config.setTestOnBorrow(TEST_ON_BORROW);
	List<JedisShardInfo> jedisShardInfoList = new ArrayList<JedisShardInfo>();
	JedisShardInfo jedisShardInfo1 = new JedisShardInfo(ADDR, PORT, TIMEOUT);
	jedisShardInfo1.setPassword(AUTH); //没密码则不用带AUTH参数
	jedisShardInfoList.add(jedisShardInfo1);//有多少个服务器就带多少个
	shardedJedisPool = new ShardedJedisPool(config, jedisShardInfoList);	
```
方法三：SpringBoot方式配置ShardedJedisPool
第一步：application.yml中配置连接属性：

```
redisConfigure:
  maxTotal: 500
  maxIdle: 200
  maxWaitMillis: 10000
  testOnBorrow: true
  jedisShardInfoList[0]:
    addr: 192.168.1.188
    port: 6379
    timeout: 10000
password: 123456
  jedisShardInfoList[1]:
    addr: 192.168.1.188
    port: 6379
    timeout: 10000
    password: 123456
  ```
#...有多少服务器集群就带多少个
第二步：创建对应的属性类(两个嵌套)，将application.yml中的属性注入
```
@Component("redisConfigure")
@ConfigurationProperties(prefix = "redisConfigure")
public class RedisConfigure {
	private int maxTotal;
	private int maxIdle;
	private int maxWaitMillis;
	private boolean testOnBorrow;
	private List<JedisSharedInfoBean> jedisShardInfoList;
	
	public int getMaxTotal() {
		return maxTotal;
	}
	public void setMaxTotal(int maxTotal) {
		this.maxTotal = maxTotal;
	}
	public int getMaxIdle() {
		return maxIdle;
	}
	public void setMaxIdle(int maxIdle) {
		this.maxIdle = maxIdle;
	}
	public int getMaxWaitMillis() {
		return maxWaitMillis;
	}
	public void setMaxWaitMillis(int maxWaitMillis) {
		this.maxWaitMillis = maxWaitMillis;
	}
	public boolean getTestOnBorrow() {
		return testOnBorrow;
	}
	public void setTestOnBorrow(boolean testOnBorrow) {
		this.testOnBorrow = testOnBorrow;
	}
	public List<JedisSharedInfoBean> getJedisShardInfoList() {
		return jedisShardInfoList;
	}
	public void setJedisShardInfoList(List<JedisSharedInfoBean> jedisShardInfoList) {
		this.jedisShardInfoList = jedisShardInfoList;
	}
}
public class JedisSharedInfoBean {
	private String addr;
	private int port;
	private int timeout;
	private String password;
	
	public String getAddr() {
		return addr;
	}
	public void setAddr(String addr) {
		this.addr = addr;
	}
	public int getPort() {
		return port;
	}
	public void setPort(int port) {
		this.port = port;
	}
	public int getTimeout() {
		return timeout;
	}
	public void setTimeout(int timeout) {
		this.timeout = timeout;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
}
```
第二步：初始化连接池并创建Redis工具类(连接池在第一次被使用时初始化)
(静态工具类中注入bean最好自定义一个类实现ApplicationContextAware接口来获取)
```
public class RedisUtil {
	private static ShardedJedisPool POOL = null; // 集群连接池
	/*在多线程环境同步初始化集群连接池/
	private static synchronized void poolInit() {
		if (POOL != null) {
			return;
		}
		// 读取配置文件中的redis属性
		RedisConfigure redisConfigure = SpringContexts.getBean("redisConfigure", RedisConfigure.class);
		
		JedisPoolConfig config = new JedisPoolConfig();
		config.setMaxTotal(redisConfigure.getMaxTotal());
		config.setMaxIdle(redisConfigure.getMaxIdle());
		config.setMaxWaitMillis(redisConfigure.getMaxWaitMillis());
		config.setTestOnBorrow(redisConfigure.getTestOnBorrow());
		
		//若干个redis集群服务器
		List<JedisShardInfo> jedisShardInfoList = new ArrayList<JedisShardInfo>();
		List<JedisSharedInfoBean> jedisSharedInfoBeanList = redisConfigure.getJedisShardInfoList();
		if (jedisSharedInfoBeanList != null && !jedisSharedInfoBeanList.isEmpty()) {
			for (JedisSharedInfoBean temp : jedisSharedInfoBeanList) {
				JedisShardInfo jedisShardInfo = new JedisShardInfo(temp.getAddr(), temp.getPort(),
						temp.getTimeout());
				if (temp.getPassword() != null && !"".equals(temp.getPassword())) {
					jedisShardInfo.setPassword(temp.getPassword());
				}
				jedisShardInfoList.add(jedisShardInfo);
			}
		}
		POOL = jedisShardInfoList.isEmpty() ? null : new ShardedJedisPool(config, jedisShardInfoList);
	}
	
	/**同步获取ShardedJedis实例*/
	public synchronized static ShardedJedis getJedis() {
		if (POOL == null) {
			poolInit();
		}
		ShardedJedis shardedJedis = null;
		try {
			if (POOL != null) {
				shardedJedis = POOL.getResource();
			}
		} catch (Exception e) {
			POOL.returnBrokenResource(shardedJedis);
		}
		return shardedJedis;
	}

	/** 释放非正常的jedis */
	public static void returnBrokenResource(final ShardedJedis shardedJedis) {
		if (shardedJedis != null && POOL != null) {
			POOL.returnBrokenResource(shardedJedis);
		}
	}
	/** 释放正常的jedis资源*/
	public static void returnResource(final ShardedJedis shardedJedis) {
		if (shardedJedis != null && POOL != null) {
			POOL.returnResource(shardedJedis);
		}
	}
	/**取值*/
	public static String get(String key) {
		ShardedJedis shardedJedis = null;
		String value = null;
		try {
			shardedJedis = getJedis();
			if (shardedJedis == null || !shardedJedis.exists(key)) {
				return null;
			}
			value = shardedJedis.get(key);
		} catch (Exception e) {
			POOL.returnBrokenResource(shardedJedis);
		} finally {
			returnResource(shardedJedis);
		}
		return value;
	}
	/**设值带过期时间*/
	public static void set(String key, int seconds, String value) {
		ShardedJedis shardedJedis = null;
		try {
			shardedJedis = getJedis();
			shardedJedis.setex(key, seconds, value);
		} catch (Exception e) {
			POOL.returnBrokenResource(shardedJedis);
		} finally {
			returnResource(shardedJedis);
		}
	}

	/**设值不带过期时间*/
	public static void set(String key, String value) {
		ShardedJedis shardedJedis = null;
		try {
			shardedJedis = getJedis();
			shardedJedis.set(key, value);
		} catch (Exception e) {
			POOL.returnBrokenResource(shardedJedis);
		} finally {
			returnResource(shardedJedis);
		}
	}
}
/**spring ApplicationContext工具,用来获取bean，componet注解不能少*/
@Component
public class SpringContexts implements ApplicationContextAware {
  private static ApplicationContext context;

  /**获取spring 管理的bean  */
  public static <T> T getBean(String name, Class<T> requiredType) {
    return context.getBean(name, requiredType);
  }
  public static ApplicationContext getContext() {
    return context;
  }
  @Override
  public void setApplicationContext(ApplicationContext contex) throws BeansException {
    SpringContexts.context = contex;
  }
}
```
5.2 JedisCluster的使用
  只列举xml配置方法
第一步：xml注入
```
    <!-- 引入配置文件 -->  
    <bean id="propertyConfigurer"  
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="location" value="classpath:redis.properties" />  
    </bean>  
    <!-- jedis 配置-->  
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig" >  
        <!--最大空闲数-->  
        <property name="maxIdle" value="${redis.maxIdle}" />  
        <!--最大建立连接等待时间-->  
        <property name="maxWaitMillis" value="${redis.maxWait}" />  
        <!--是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个-->  
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />  
        <property name="maxTotal" value="${redis.maxTotal}" />  
        <property name="minIdle" value="${redis.minIdle}" />  
    </bean >  
    <bean id="jedisCluster"  class="com.xiaomu.redis.JedisClusterFactory" >  
        <property name="addressConfig">  
            <value>classpath:redis.properties</value>  
        </property>  
        <property name="addressKeyPrefix" value="cluster" />   <!--  属性文件里  key的前缀 -->  
        <property name="timeout" value="300000" />  
        <property name="maxRedirections" value="6" />  
        <property name="genericObjectPoolConfig" ref="poolConfig" />  
    </bean >  
```
第二步：配置redis.properties

```
redis.maxIdle=100  
redis.maxActive=300  
redis.maxWait=1000  
redis.timeout=100000  
redis.maxTotal=1000  
redis.minIdle=8  
redis.testOnBorrow=true 
cluster1.host.port=119.254.166.136:7031
cluster2.host.port=119.254.166.136:7032
cluster3.host.port=119.254.166.136:7033
cluster4.host.port=119.254.166.136:7034
cluster5.host.port=119.254.166.136:7035
cluster6.host.port=119.254.166.136:7036 
```

第三步：创建与xml对应的Factory类,实现FactoryBean<JedisCluster>, InitializingBean
```
public class JedisClusterFactory implements FactoryBean<JedisCluster>, InitializingBean {  
    private Resource addressConfig;  
    private String addressKeyPrefix ;  
    private JedisCluster jedisCluster;  
    private Integer timeout;  
    private Integer maxRedirections;  
    private GenericObjectPoolConfig genericObjectPoolConfig;  
    private Pattern p = Pattern.compile("^.+[:]\\d{1,5}\\s*$");  

    @Override  
    public JedisCluster getObject() throws Exception {  
        return jedisCluster;  
    }  
    @Override  
    public Class<? extends JedisCluster> getObjectType() {  
        return (this.jedisCluster != null ? this.jedisCluster.getClass() : JedisCluster.class);  
    }  
    @Override  
    public boolean isSingleton() {  
        return true;  
    }
    private Set<HostAndPort> parseHostAndPort() throws Exception {  
        try {  
            Properties prop = new Properties();  
            prop.load(this.addressConfig.getInputStream());  

            Set<HostAndPort> haps = new HashSet<HostAndPort>();  
            for (Object key : prop.keySet()) {  

                if (!((String) key).startsWith(addressKeyPrefix)) {  
                    continue;  
                }  

                String val = (String) prop.get(key);  

                boolean isIpPort = p.matcher(val).matches();  

                if (!isIpPort) {  
                    throw new IllegalArgumentException("ip 或 port 不合法");  
                }  
                String[] ipAndPort = val.split(":");  

                HostAndPort hap = new HostAndPort(ipAndPort[0], Integer.parseInt(ipAndPort[1]));  
                haps.add(hap);  
            }  
            return haps;  
        } catch (IllegalArgumentException ex) {  
            throw ex;  
        } catch (Exception ex) {  
            throw new Exception("解析 jedis 配置文件失败", ex);  
        }  
    }  
    @Override  
    public void afterPropertiesSet() throws Exception {  
        Set<HostAndPort> haps = this.parseHostAndPort();  
        jedisCluster = new JedisCluster(haps, timeout, maxRedirections,genericObjectPoolConfig);  
    }  
    public void setAddressConfig(Resource addressConfig) {  
        this.addressConfig = addressConfig;  
    }  
    public void setTimeout(int timeout) {  
        this.timeout = timeout;  
    }  
    public void setMaxRedirections(int maxRedirections) {  
        this.maxRedirections = maxRedirections;  
    }  
    public void setAddressKeyPrefix(String addressKeyPrefix) {  
        this.addressKeyPrefix = addressKeyPrefix;  
    }  
    public void setGenericObjectPoolConfig(GenericObjectPoolConfig genericObjectPoolConfig) {  
        this.genericObjectPoolConfig = genericObjectPoolConfig;  
    }  

}  
```

第四步：使用xml中配置的factory的bean名获取JedisCluster实例，并使用

```
@Autowired  
private JedisCluster jedisCluster; 
    jedisCluster.set("name", "啊芝");
    String val = jedisCluster.get("name");
    
5.3 Jedis或ShardedJedis实例的使用

ShardedJedis 使用方法和Jedis 基本一样，但ShardedJedis 不支持批量设置，如mget、mset、brpop

public class TestRedis {
	public static void main(String[] args) {
		ShardedJedis jedis = RedisUtil.getJedis(); //通过某种连接池方式获取实例
		boolean flag = jedis.exists("user");	//是否存在
	}
	/**Jedis操作String	 */
	public static void fun1(ShardedJedis jedis) {
		jedis.del("name");
		jedis.set("name", "张三");		//设值
		jedis.append("name", "三");		//拼接
           //jedis.mset("name","liuling","age","23","qq","476777XXX");//同时设置多个值（ShardedJedis不支持，Jedis支持）
		System.out.println(jedis.get("name")); //取值
	}
	/**Jedis操作List	 */
	public static void fun2(ShardedJedis jedis) {
		jedis.del("list");
		jedis.rpush("list", "value1");	//插入到右侧
		jedis.lset("list", 1, "vv");	// 设置指定下标的值
           //jedis.lpush("list", "value3");	//插入到左侧
		List<String> list = jedis.lrange("list", 0, 2);	//取值,key，起始下标,结束下标（-1代表所有）		
	}
	/**Jedis操作set	 */
	public static void fun3(ShardedJedis jedis) {
		jedis.del("user");
		jedis.sadd("user", "liuling"); // 添
		jedis.srem("user", "who"); // 移除set中某个元素
		System.out.println(jedis.smembers("user"));// 获取所有加入的value
		System.out.println(jedis.sismember("user", "who"));	// 是否是user集合的元素
		System.out.println(jedis.scard("user"));// 返回集合的元素个数
	}
	/**Jedis操作Map*/
	public static void fun4(ShardedJedis jedis) {
		jedis.del("testMap");
		// -----添加数据----------
		Map<String, String> map = new HashMap<String, String>();
		map.put("name", "xinxin");
		jedis.hmset("testMap", map);
		// 第一个参数是存入redis中map对象的key，后面跟的是放入map中的对象的key，后面的key可以跟多个，是可变参数
		List<String> rsmap = jedis.hmget("testMap", "name", "age", "qq");

		// 删除map中的某个键值
		jedis.hdel("testMap", "age");
		System.out.println(jedis.hmget("testMap", "age")); // 因为删除了，所以返回的是null
		System.out.println(jedis.hlen("testMap")); // 返回key为user的键中存放的值的个数2
		System.out.println(jedis.hkeys("testMap"));// 返回map对象中的所有key
		System.out.println(jedis.hvals("testMap"));// 返回map对象中的所有value
		Iterator<String> iter = jedis.hkeys("testMap").iterator();
		while (iter.hasNext()) {
			String key = iter.next();
			System.out.println(key + ":" + jedis.hmget("user", key));
		}
	}
}
```


