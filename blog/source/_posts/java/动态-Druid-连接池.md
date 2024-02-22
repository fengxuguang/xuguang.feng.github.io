---
title: 动态 Druid 连接池
tags:
  - java
categories:
  - java
thumbnail: /images/home.jpg
sticky: 99
abbrlink: ecd042ad
date: 2024-02-07 14:56:47
---

## 动态 Druid 连接池

### 背景

 **需求**：因项目业务需求，需要在系统上配置不同数据库，然后进行连接获取数据库相关数据； 原生 JDBC 通过 DriverManager 进行连接每次都要重新进行连接，导致整体查询时间效率变低，故引入 Druid 连接池；
  但是不同类型数据库生成`DruidDataSource`后没有将`DS(DruidDataSource)`进行缓存起来， 本质还是每次连接数据库时还是重新进行数据库连接，问题依然存在。
  所以设计了一个`容器`将`DS`缓存起来，考虑到容器里数量限制，使用`LinkedHashMap`进行存储数据，控制存储`DS`的数量。 另考虑缓存起来的`DS`存在数据库密码修改的情况，支持更新`容器`内`DS`的`密码`，保证 DS 可用。
具体代码如下：

```Java
import com.alibaba.druid.pool.DruidDataSource;
import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang.StringUtils;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Objects;

/**
 * 连接池工具类
 * Created by fengxuguang on 2023/12/28 11:54
 */
@Slf4j
public class ConnectionPoolUtil {
    /**
     * 初始化大小, 设置为 10
     */
    private static final int INITIAL_SIZE = 10;
    /**
     * 最小线程数
     */
    private static final int MIN_IDLE = 10;
    /**
     * 最大线程数
     */
    private static final int MAX_ACTIVE = 200;
    /**
     * 连接等待超时时间, 单位: 毫秒
     */
    private static final int MAX_WAIT = 60_000;
    /**
     * 间隔多久检测一次需要关闭的空闲连接, 单位: 毫秒
     */
    private static final int TIME_BETWEEN_EVICTION_RUNS_MILLIS = 200_000;
    /**
     * 重连次数
     */
    private static final int CONNECTION_ERRORS_RETRY_COUNT = 3;

    /**
     * 配置一个空闲连接在池中最小生存的时间, 单位是毫秒, 设置 5 分钟
     */
    private static final int MIN_EVICT_TABLE_IDLE_TIME_MILLIS = 5 * 60 * 1000;

    /**
     * 连接池容器最大数量
     */
    private static final int CONTAINER_MAX_SIZE = 5;

    /**
     * 连接池容器
     */
    private static final Map<DbSourceKey, DruidDataSource> containers = new ContainerLinkedHashMap(CONTAINER_MAX_SIZE);

    /**
     * 获取 DataSource
     * @param driverName 连接 driver-class
     * @param connStr 连接 url
     * @param username 账号
     * @param password 密码
     * @param validateQuery 校验查询语句
     * @return DruidDataSource
     */
    public synchronized static DruidDataSource getDataSource(String driverName, String connStr, String username, String password,
                                                String validateQuery) {
        try {
            DbSourceKey dbSourceKey = new DbSourceKey(connStr, username, password);
            if (containers.containsKey(dbSourceKey)) {
                return getDruidDataSource(dbSourceKey);
            }

            containers.put(dbSourceKey, createDruidDataSource(driverName, connStr, username, password, validateQuery));

            return containers.get(dbSourceKey);
        } catch (Exception e) {
            log.error("创建数据源连接池失败：{}", e.getMessage(), e);
        }
    }

    /**
     * 创建 DruidDataSource 对象
     * @param driverName driver-class
     * @param connStr 连接 url
     * @param username 账号
     * @param password 密码
     * @param validateQuery 保持连接SQL
     * @return DruidDataSource
     */
    private static DruidDataSource createDruidDataSource(String driverName, String connStr, String username, String password,
                                           String validateQuery) {
        DruidDataSource ds = new DruidDataSource();

        // 基础连接信息
        ds.setDriverClassName(driverName);
        ds.setUrl(connStr);
        ds.setUsername(username);
        ds.setPassword(password);

        // 初始化大小
        ds.setInitialSize(INITIAL_SIZE);
        ds.setMinIdle(MIN_IDLE);
        ds.setMaxActive(MAX_ACTIVE);

        // 连接泄露监测
        ds.setRemoveAbandoned(true);
        ds.setRemoveAbandonedTimeout(30);

        // 配置时间
        // 配置获取连接等待超时的时间
        ds.setMaxWait(MAX_WAIT);
        // 配置间隔多久才进行一次检测, 检测需要关闭的空闲连接, 单位: 毫秒
        ds.setTimeBetweenEvictionRunsMillis(TIME_BETWEEN_EVICTION_RUNS_MILLIS);
        // 设置重连次数
        ds.setBreakAfterAcquireFailure(true);
        ds.setConnectionErrorRetryAttempts(CONNECTION_ERRORS_RETRY_COUNT);

        // 防止过期
        if (StringUtils.isBlank(validateQuery)) {
            validateQuery = "SELECT 'x'";
        } else if ("-1".equals(validateQuery)) {
            validateQuery = null;
        }
        ds.setValidationQuery(validateQuery);
        ds.setTestWhileIdle(true);
        ds.setTestOnBorrow(true);
        ds.setMinEvictableIdleTimeMillis(MIN_EVICT_TABLE_IDLE_TIME_MILLIS);

        return ds;
    }

    /**
     * 根据 dbSourceKey 获取 DruidDataSource
     * @param dbSourceKey dbSourceKey
     * @return DruidDataSource
     */
    private static DruidDataSource getDruidDataSource(DbSourceKey dbSourceKey) {
        if (!containers.containsKey(dbSourceKey)) {
            return null;
        }
        // verify whether the password is changed. if the password is changed, update the password in the DruidDataSource
        if (!dbSourceKey.getPassword().equals(containers.get(dbSourceKey).getPassword())) {
            containers.get(dbSourceKey).setPassword(dbSourceKey.getPassword());
        }
        return containers.get(dbSourceKey);
    }

    /**
     * 容器链表, 用于存储 DruidDataSource, key: DbSourceKey, value: DruidDataSource
     */
    private static class ContainerLinkedHashMap extends LinkedHashMap<DbSourceKey, DruidDataSource> {

        /**
         * 链接最大容量
         */
        private final int maxSize;

        ContainerLinkedHashMap(int maxSize) {
            if (maxSize <= 0) {
                this.maxSize = 1;
            } else {
                this.maxSize = maxSize;
            }
        }

        @Override
        public DruidDataSource get(Object key) {
            return super.get(key);
        }

        @Override
        protected boolean removeEldestEntry(Map.Entry<DbSourceKey, DruidDataSource> eldest) {
            // 超过最大容量, 需要移除旧的数据
            if (containers.size() > maxSize) {
                DruidDataSource druidDataSource = eldest.getValue();
                druidDataSource.close();
            }

            return containers.size() > maxSize;
        }
    }

    /**
     * 存储数据源信息
     */
    private static class DbSourceKey {

        /**
         * 连接 url, 设置后不可修改
         */
        private final String connStr;

        /**
         * 账号, 设置后不可修改
         */
        private final String account;

        /**
         * 密码, 设置后密码可修改
         */
        @Setter
        @Getter
        private String password;

        DbSourceKey(String connStr, String account, String password) {
            this.connStr = connStr;
            this.account = account;
            this.password = password;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            DbSourceKey that = (DbSourceKey) o;
            return Objects.equals(connStr, that.connStr) && Objects.equals(account, that.account);
        }

        @Override
        public int hashCode() {
            return Objects.hash(connStr, account);
        }
    }

}
```

### 扩展知识

1. `ContainerLinkedHashMap`继承`LinkedHashMap`后重写`removeEldestEntry`方法的原因。

```Java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
    // 省略...
} 
```

解释：由于`LinkedHashMap`继承`HashMap`，`LinkedHashMap`在调用`put`方法时，实际是调用`HashMap`的`put`方法，`HashMap`的`put`方法源码如下：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // 省略...
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

    void afterNodeInsertion(boolean evict) { }
}
```

看源码可知，调用`put`方法后接着去调用`putVal`方法，在将数据插入到`HashMap`后，会调用`afterNodeInsertion`方法，默认`HashMap`对此方法不做处理。但是`LinkedHashMap`对`afterNodeInsertion`此方法进行了处理，源码如下：

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{

    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
    
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
}
```

在`afterNodeInsertion`方法内会调用`removeEldestEntry`方法来判断是否删除链表头数据，默认`LinkedHashMap`的`removeEldestEntry`返回`false`，表示不移除链表头元素。

`ContainerLinkedHashMap`继承`LinkedHashMap`后，可以对`removeEldestEntry`方法进行重新，根据链表内的容量来判断是否需要删除链表头元素，以达到控制链表容量的目的。

ps：`removeEldestEntry`方法在`LinkedHashMap`调用时传入的参数为链表的头元素。

源码如下：

```java
private static class ContainerLinkedHashMap extends LinkedHashMap<DbSourceKey, DruidDataSource> {

    /**
     * 链接最大容量
     */
    private final int maxSize;
        
    @Override
    protected boolean removeEldestEntry(Map.Entry<DbSourceKey, DruidDataSource> eldest) {
        // 超过最大容量, 需要移除旧的数据
        if (containers.size() > maxSize) {
            DruidDataSource druidDataSource = eldest.getValue();
            druidDataSource.close();
        }
    
        return containers.size() > maxSize;
    }
}
```
