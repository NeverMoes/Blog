# 分布式 ID 生成

[toc]

## 为什么使用分布式 ID

### 分布式 ID 是什么

一般是在数据分库分表的情况下，需要一个 ID 作为主键表示。显然一般的数据库自增 ID 满足不了这样的需求。

即需要一个**全局唯一的分布式ID**。

### 满足条件

#### 定性

- 全局唯一：基本要求，主键的定义。
- 高性能：生成速度要快，不能成为业务瓶颈。
- 高可用：完全可用是不可能的，但是要接近 100% 的可用性。
- 易用性：在系统设计和实现上要尽可能的简单。
- 单调递增：最好能够像主键 ID 一样**接近**单调递增。

#### 定量

* 是否需要联网
* 长度
* 生成速度
* 算法复杂度
* 是否单调
* 碰撞概率



## 

32x16=512 (bit)

## 基于算法

### UUID



### Snowflake （雪花算法）

![](https://i.loli.net/2020/05/14/hkaLnYfpd9S2QBc.png)

Snowflake 算法由 twitter 开源，这个算法会生成一个 `Long` 类型（64位）的 ID，分为四段，每一段的含义为：

1. 正负位(1 bit)：一般都为正数，所以基本不用。
2. 时间戳(41bit)：毫秒单位，是一个（当前时间戳 - 某个固定时间戳）的差值。
   * 因为 41 位只能表示 69 年的时间，这样可以缩短表示时间戳位数的长度。
3. 机器 ID(10 bit)：可以灵活配置，用机房和机器的组合 ID 都可以。
4. 序列号(12 bit)：自增值，支持同一个节点同时生成 4096 个 ID。

Java 版本的实现

```java
/**
 * https://github.com/beyondfengyu/SnowFlake
 */
public class SnowFlakeShortUrl {

    /**
     * 起始的时间戳
     */
    private final static long START_TIMESTAMP = 1480166465631L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12;   //序列号占用的位数
    private final static long MACHINE_BIT = 5;     //机器标识占用的位数
    private final static long DATA_CENTER_BIT = 5; //数据中心占用的位数

    /**
     * 每一部分的最大值
     */
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_DATA_CENTER_NUM = -1L ^ (-1L << DATA_CENTER_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATA_CENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTAMP_LEFT = DATA_CENTER_LEFT + DATA_CENTER_BIT;

    private long dataCenterId;  //数据中心
    private long machineId;     //机器标识
    private long sequence = 0L; //序列号
    private long lastTimeStamp = -1L;  //上一次时间戳

    private long getNextMill() {
        long mill = getNewTimeStamp();
        while (mill <= lastTimeStamp) {
            mill = getNewTimeStamp();
        }
        return mill;
    }

    private long getNewTimeStamp() {
        return System.currentTimeMillis();
    }

    /**
     * 根据指定的数据中心ID和机器标志ID生成指定的序列号
     *
     * @param dataCenterId 数据中心ID
     * @param machineId    机器标志ID
     */
    public SnowFlakeShortUrl(long dataCenterId, long machineId) {
        if (dataCenterId > MAX_DATA_CENTER_NUM || dataCenterId < 0) {
            throw new IllegalArgumentException("DtaCenterId can't be greater than MAX_DATA_CENTER_NUM or less than 0！");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("MachineId can't be greater than MAX_MACHINE_NUM or less than 0！");
        }
        this.dataCenterId = dataCenterId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     */
    public synchronized long nextId() {
        long currTimeStamp = getNewTimeStamp();
        if (currTimeStamp < lastTimeStamp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currTimeStamp == lastTimeStamp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currTimeStamp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastTimeStamp = currTimeStamp;

        return (currTimeStamp - START_TIMESTAMP) << TIMESTAMP_LEFT //时间戳部分
                | dataCenterId << DATA_CENTER_LEFT       //数据中心部分
                | machineId << MACHINE_LEFT             //机器标识部分
                | sequence;                             //序列号部分
    }
    
    public static void main(String[] args) {
        SnowFlakeShortUrl snowFlake = new SnowFlakeShortUrl(2, 3);

        for (int i = 0; i < (1 << 4); i++) {
            //10进制
            System.out.println(snowFlake.nextId());
        }
    }
}
```

### uid-generator （百度）

```java
LoadingCache<Key, Item> items = CacheBuilder.newBuilder()  
            .maximumSize(1000)
             .expireAfterWrite(1000)
            .build(
                new CacheLoader<Key, Item>() {
                    public Item load(Key key) throws Exception {
                        return getCacheFromRedis(key);
                    }
                });
```



## 参考

[一口气说出9种分布式ID生成方式](https://zhuanlan.zhihu.com/p/107939861)

