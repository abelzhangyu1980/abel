system design study

  1. SCALE FROM ZERO TO MILLIONS OF USERS

        browser      --------> dns lookup
        mobile app    ------->  dns lookup
                      --------> single web server   <---> CRUD database   --> sql database (rdbms)  
                                                                          -->nosql database (key-value/graph/column/document)
                                                                          
        Non-relational databases might be the right choice if: 
         a. app requires super-low latency
         b. data are unstructured, or you do not have any relational data
         c. only need to serialize and deserialize data (JSON, XML, YAML, etc.
         d. need to store a massive amount of data.

Vertical scaling vs horizontal scaling
   vertical scaling -->scal up via adding more memory/cpu resources on the same server   ---hardware limit, single point of failure. no failover and redundancy. 
   horizontal scaling --> scal out --> more computational nodes.
   
   
   so add a load balancer -> web server 1
                          -> web server 2
   
   load balancer will expose public ip and it uses private ip to communiate with web servers.
   
   but how about database layer as it becomes a single point of failure.
   
   add master/slave  --- database replication
   
   web server1
   web server2 ----> write to master
               ----> read to slaves //read write separation.  one master can have multiple read-only slaves and use db replication to sync data.
               
   better performance as master only handles update while slaves proceed queries in parallel
   high availability. if one slave down, no impact, if master is down, elect new master./prompt new master and some auto recovery script...
   
   next is to host static file in CDN. ---caching..
   reduce web server and database workload.
     A CDN is a network of geographically dispersed servers used to deliver static content. CDN servers cache static content like images, videos, CSS, JavaScript files, etc
     To minimize the distance between the visitors and your website’s server, a CDN stores a cached version of its content in multiple geographical locations (a.k.a., points of presence, or PoPs). Each PoP contains a number of caching servers responsible for content delivery to visitors within its proximity.
   
   or Memcached cache. use api to detemine if object is in cache (code level )
   
   first check if object in cache or not. if yes, return it directly. otherwise read from database, store in cache and return.
   web server --> cache server
               --> db server
               
   when to use cache ---> read frequently but modify infrequently. 
                          expiration policy
                          consistency  --This involves keeping the data store and the cache in sync. it can happen as data update on database and cache won't happen in a single transaction.
                          mitigate failures --- spof of cache layer
                          eviction policy --Once the cache is full, any requests to add items to the cache might cause existing items to be removed
                                 Least-recently-used --LRU  Least Frequently Used (LFU) or First in First Out (FIFO) 
                                       LRU -- like a stack. latest accessed will be on the top. and bottom will be evicted.
                                       LFU --- take access time into account. keep track of how many times the cache request has been used.  It requires three data structures. One is a hash table that is used to cache the key/values so that given a key we can retrieve the cache entry at O(1). The second one is a double linked list for each frequency of access.
                          
                          
   next --> stateless web layer
      Now it is time to consider scaling the web tier horizontally. For this, we need to move state (for instance user session data) out of the web tier. A good practice is to store session data in
the persistent storage such as relational database or NoSQL. Each web server in the cluster can access state data from databases. This is called stateless web tier.
      sticky session can route the same requests to the same server. however it adds difficulty on adding/removing server and failover.
      
      then we can add web servers into a auto-scale group and then use non-sql db to persist stateful data.
      
      so far so good, how about expanding business across geographical areas... multiple data centres.
          how to direct traffic to the nearest data center depending on where a user is located.
          data synchronization
          test and deployment
          

database sharding. distribute data to seversal sub databases based on sharding key.
  resharding data if a single shard could no longer hold more data due to rapid growth. 2) Certain shards might experience shard exhaustion faster than others due to uneven data distribution
  celebrity problem or hotkey problem.
  Join and de-normalization:
  
  summary:
• Keep web tier stateless
• Build redundancy at every tier
• Cache data as much as you can
• Support multiple data centers
• Host static assets in CDN
• Scale your data tier by sharding
• Split tiers into individual services
• Monitor your system and use automation tools

power of two:

rate limiter  (client vs server side) cilent side, use timer , server side, at api gateway. at web server/app server..
Algorithms for rating limiting 
1. token bucket
   The token bucket algorithm takes two parameters:
     • Bucket size: the maximum number of tokens allowed in the bucket
     • Refill rate: number of tokens put into the bucket every second
2. leaking bucket.
     . bucket size. implemented at FIFO queue. 
       fixed processing rate. / Outflow rate / fixed rate. 
3, Fixed window counter
    . • The algorithm divides the timeline into fix-sized time windows and assign a counter for each window. and each request increases the counter.   Once the counter reaches the pre-defined threshold, new requests are dropped until a new
time window starts.
       . A major problem with this algorithm is that a burst of traffic at the edges of time windows could cause more requests than allowed quota to go through. spike in traffic within two connecting windows is an issue
          as total number of second half of current window and first half of next window could exceed the allowed quota.
       
 4. sliding window log algorithm
      to fix fixed window counter issue. hence we check the number_of_requests made in last time_window_sec seconds. So this process of checking for a fixed window of time_window_sec seconds on every request, makes this approach a sliding window where the fixed window of size time_window_sec seconds
      is moving forward with each request. 每个新的请求到达后，检查从这个时间往前的窗口（一分钟）有多少个请求，如果超过则拒绝当前请求。如果有过期的则删除。
      比如允许一分钟内有两个请求。
      在1:00:01的时候收到第一个请求。
      时间戳 1:00:01 添加到log里面，最近一秒只有一个请求，允许。
      1:00:30收到第二个请求，合法

      1:00:01
      1:00:30

      1:00:50收到第三个请求，最近一分钟（12:59:50-1:00:50)）之间有三个请求，拒绝。

     1:01：40秒收到第四个请求，往前看一分钟，1:00:40 - 1:01:40之间有两个（加上当前），合法 。 同时删除窗口之外的两个请求。

    pros: Rate limiting implemented by this algorithm is very accurate. In any rolling window,requests will not exceed the rate limit.
    cons: The algorithm consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory.

  5. Sliding window counter algorithm
     The sliding window counter algorithm is a hybrid approach that combines the fixed window counter and sliding window log
      Assume the rate limiter allows a maximum of 7 requests per minute, and there are 5 requests in the previous minute and 3 in the current minute. For a new request that arrives at a 30%
position in the current minute, the number of requests in the rolling window is calculated using the following formula:
• Requests in current window + requests in the previous window * overlap percentage of the rolling window and previous window
  • It only works for not-so-strict look back window. It is an approximation of the actual rate
because it assumes requests in the previous window are evenly distributed


DESIGN CONSISTENT HASHING
serverIndex = hash(key) % N, where N is the size of the server pool. This approach works well when the size of the server pool is fixed, and the data distribution
is even. However, problems arise when new servers are added, or existing servers are removed.
Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on
average, where k is the number of keys, and n is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be
remapped       

哈希算法将任意长度的二进制值映射为较短的固定长度的二进制值，这个小的二进制值称为哈希值。哈希值是一段数据唯一且极其紧凑的数值表示形式。
普通的hash算法在分布式应用中的不足：

比如，在分布式的存储系统中，要将数据存储到具体的节点上，如果我们采用普通的hash算法进行路由，将数据映射到具体的节点上，如key%N，key是数据的key，N是机器节点数，如果有一个机器加入或退出这个集群，则所有的数据映射都无效了，如果是持久化存储则要做数据迁移，如果是分布式缓存，则其他缓存就失效了。
假设有这么一种场景：我们有三台缓存服务器分别为：node0、node1、node2，有3000万个缓存数据需要存储在这三台服务器组成的集群中，希望可以将这些数据均匀的缓存到三台机器上，你会想到什么方案呢？
我们可能首先想到的方案是：取模算法hash（key）%N，即：对缓存数据的key进行hash运算后取模，N是机器的数量；运算后的结果映射对应集群中的节点。具体如下图所示：
![Alt text](c1.png?raw=true "Title")
如上图所示，首先对key进行hash计算后的结果对3取模，得到的结果一定是0、1或者2；然后映射对应的服务器node0、node1、node2，最后直接找对应的服务器存取数据即可。
通过取模算法将每个数据请求都均匀的分散到了三个不同的服务器节点上，看起来很完美！但是，在分布式集群系统的负载均衡实现上，这种模型在集群扩容和收缩时却有一定的局限性：因为在生产环境中根据业务量的大小，调整服务器数量是常有的事，而服务器数量N发生变化后hash（key）%N计算的结果也会随之变化！导致整个集群的缓存数据必须重新计算调整，进而导致大量缓存在同一时间失效，造成缓存的雪崩，最终导致整个缓存系统的不可用，这是不能接受的。为了解决优化上述情况，一致性哈希算法应运而生。

一致性hash算法
均衡性(Balance)
平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。
单调性(Monotonicity)
单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲区加入到系统中，那么哈希的结果应能够保证原有已分配的内容可以被映射到新的缓冲区中去，而不会被映射到旧的缓冲集合中的其他缓冲区。（这段翻译信息有负面价值的，当缓冲区大小变化时一致性哈希(Consistent hashing)尽量保护已分配的内容不会被重新映射到新缓冲区。）
分散性(Spread)
在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。
负载(Load)
负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。

接下来说一下具体的设计：

2.1环形hash空间
按照常用的hash算法来将对应的key哈希到一个具有2^32次方个节点的空间中，即0 ~ (2^32)-1的数字空间中。现在我们可以将这些数字头尾相连，想象成一个闭合的环形
2.2映射服务器节点
将各个服务器使用Hash进行一个哈希，具体可以选择服务器的ip或唯一主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。假设我们将四台服务器使用ip地址哈希后在环空间的位置如下：
  一个闭环，然后用hash将机器放到闭环上。
2.3映射数据
现在我们将objectA、objectB、objectC、objectD四个对象通过特定的Hash函数计算出对应的key值，然后散列到Hash环上,然后从数据所在位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器。
    
![Alt text](consistent%20hash.png?raw=true "Title")
但是有一个很致命的问题，如果节点数量发生了变化 （新增，删除）），也就是在对系统做扩容或者缩容时，必须迁移改变了映射关系的数据，否则会出现查询不到数据的问题。

一致哈希算法也用了取模运算，但与哈希算法不同的是，哈希算法是对节点的数量进行取模运算，而一致哈希算法是对 2^32 进行取模运算，是一个固定的值。（hash是一个integer，最大就是 2^32）
Consistent Hashing is a distributed hashing scheme that operates independently of the number of servers or objects in a distributed hash table

A hash function is a function that maps one piece of data—typically describing some kind of object, often of arbitrary size—to another piece of data, typically an integer, known as hash code, or simply hash.

我们可以把一致哈希算法是对 2^32 进行取模运算的结果值组织成一个圆环，就像钟表一样，钟表的圆可以理解成由 60 个点组成的圆，而此处我们把这个圆想象成由 2^32 个点组成的圆，这个圆环被称为哈希环，如下图：

一致性哈希要进行两步哈希：

第一步：对存储节点进行哈希计算，也就是对存储节点做哈希映射，比如根据节点的 IP 地址进行哈希；
第二步：当对数据进行存储或访问时，对数据进行哈希映射；
所以，一致性哈希是指将「存储节点」和「数据」都映射到一个首尾相连的哈希环上。

问题来了，对「数据」进行哈希映射得到一个结果要怎么找到存储该数据的节点呢？

答案是，映射的结果值往顺时针的方向的找到第一个节点，就是存储该数据的节点。
![Alt text](h0.png?raw=true "Title")
![Alt text](h1.png?raw=true "Title")
![Alt text](h2.png?raw=true "Title")
![Alt text](h3.png?raw=true "Title")
![Alt text](h4.png?raw=true "Title")


一致性哈希算法虽然减少了数据迁移量，但是存在节点分布不均匀的问题。

![Alt text](h5.png?raw=true "Title")

因此，在一致哈希算法中，如果增加或者移除一个节点，仅影响该节点在哈希环上顺时针相邻的后继节点，其它数据也不会受到影响。

如何通过虚拟节点提高均衡度？
要想解决节点能在哈希环上分配不均匀的问题，就是要有大量的节点，节点数越多，哈希环上的节点分布的就越均匀。

但问题是，实际中我们没有那么多节点。所以这个时候我们就加入虚拟节点，也就是对一个真实节点做多个副本。

具体做法是，不再将真实节点映射到哈希环上，而是将虚拟节点映射到哈希环上，并将虚拟节点映射到实际节点，所以这里有「两层」映射关系。

比如对每个节点分别设置 3 个虚拟节点：

对节点 A 加上编号来作为虚拟节点：A-01、A-02、A-03
对节点 B 加上编号来作为虚拟节点：B-01、B-02、B-03
对节点 C 加上编号来作为虚拟节点：C-01、C-02、C-03
引入虚拟节点后，原本哈希环上只有 3 个节点的情况，就会变成有 9 个虚拟节点映射到哈希环上，哈希环上的节点数量多了 3 倍。

上面为了方便你理解，每个真实节点仅包含 3 个虚拟节点，这样能起到的均衡效果其实很有限。而在实际的工程中，虚拟节点的数量会大很多，比如 Nginx 的一致性哈希算法，每个权重为 1 的真实节点就含有160 个虚拟节点。

另外，虚拟节点除了会提高节点的均衡度，还会提高系统的稳定性。当节点变化时，会有不同的节点共同分担系统的变化，因此稳定性更高。
![Alt text](h6.png?raw=true "Title")


一致性Hash算法实现
6.1 数据节点
首先定义一个节点类，实现数据节点的功能，具体代码如下：
public class Node {
    private static final int VIRTUAL_NODE_NO_PER_NODE = 200;
    private final String ip;
    private final List<Integer> virtualNodeHashes = new ArrayList<>(VIRTUAL_NODE_NO_PER_NODE);
    private final Map<Object, Object> cacheMap = new HashMap<>();

    public Node(String ip) {
        Objects.requireNonNull(ip);
        this.ip = ip;
        initVirtualNodes();
    }


    private void initVirtualNodes() {
        String virtualNodeKey;
        for (int i = 1; i <= VIRTUAL_NODE_NO_PER_NODE; i++) {
            virtualNodeKey = ip + "#" + i;
            virtualNodeHashes.add(HashUtils.hashcode(virtualNodeKey));
        }
    }

    public void addCacheItem(Object key, Object value) {
        cacheMap.put(key, value);
    }


    public Object getCacheItem(Object key) {
        return cacheMap.get(key);
    }


    public void removeCacheItem(Object key) {
        cacheMap.remove(key);
    }


    public List<Integer> getVirtualNodeHashes() {
        return virtualNodeHashes;
    }

    public String getIp() {
        return ip;
    }
}

接下来实现核心功能：一致性哈希算法，主要使用java的TreeMap类，实现哈希环和哈希查找的功能。具体代码如下所示：
public class ConsistentHash {
    private final TreeMap<Integer, Node> hashRing = new TreeMap<>();

    public List<Node> nodeList = new ArrayList<>();

    /**
     * 增加节点
     * 每增加一个节点，就会在闭环上增加给定虚拟节点
     * 例如虚拟节点数是2，则每调用此方法一次，增加两个虚拟节点，这两个节点指向同一Node
     * @param ip
     */
    public void addNode(String ip) {
        Objects.requireNonNull(ip);
        Node node = new Node(ip);
        nodeList.add(node);
        for (Integer virtualNodeHash : node.getVirtualNodeHashes()) {
            hashRing.put(virtualNodeHash, node);
            System.out.println("虚拟节点[" + node + "] hash:" + virtualNodeHash + "，被添加");
        }
    }

    /**
     * 移除节点
     * @param node
     */
    public void removeNode(Node node){
        nodeList.remove(node);
    }

    /**
     * 获取缓存数据
     * 先找到对应的虚拟节点，然后映射到物理节点
     * @param key
     * @return
     */
    public Object get(Object key) {
        Node node = findMatchNode(key);
        System.out.println("获取到节点:" + node.getIp());
        return node.getCacheItem(key);
    }

    /**
     * 添加缓存
     * 先找到hash环上的节点，然后在对应的节点上添加数据缓存
     * @param key
     * @param value
     */
    public void put(Object key, Object value) {
        Node node = findMatchNode(key);

        node.addCacheItem(key, value);
    }

    /**
     * 删除缓存数据
     */
    public void evict(Object key) {
        findMatchNode(key).removeCacheItem(key);
    }


    /**
     *  获得一个最近的顺时针节点
     * @param key 为给定键取Hash，取得顺时针方向上最近的一个虚拟节点对应的实际节点
     *      * @return 节点对象
     * @return
     */
    private Node findMatchNode(Object key) {
        Map.Entry<Integer, Node> entry = hashRing.ceilingEntry(HashUtils.hashcode(key));
        if (entry == null) {
            entry = hashRing.firstEntry();
        }
        return entry.getValue();
    }
}

如上所示，通过TreeMap的ceilingEntry() 方法，实现顺时针查找下一个的服务器节点的功能。
哈希计算方法比较常见，网上也有很多计算hash 值的函数。示例代码如下：
public class HashUtils {

    /**
     * FNV1_32_HASH
     *
     * @param obj
     *         object
     * @return hashcode
     */
    public static int hashcode(Object obj) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        String str = obj.toString();
        for (int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;

        if (hash < 0)
            hash = Math.abs(hash);
        //System.out.println("hash computer:" + hash);
        return hash;
    }
}

Find affected keys
When a server is added or removed, a fraction of data needs to be redistributed. How can we find the affected range to redistribute the keys?


CAP theorem
CAP theorem states it is impossible for a distributed system to simultaneously provide more
than two of these three guarantees: consistency, availability, and partition tolerance. Let us
establish a few definitions.
Consistency: consistency means all clients see the same data at the same time no matter which node they connect to.
Availability: availability means any client which requests data gets a response even if some of the nodes are down.
Partition Tolerance: a partition indicates a communication break between two nodes.
Partition tolerance means the system continues to operate despite network partitions. CAP theorem states that one of the three properties must be sacrificed to support 2 of the 3
properties as shown in Figure 6-1.

一致性指“all nodes see the same data at the same time”，即所有节点在同一时间的数据完全一致。一致性是因为多个数据拷贝下并发读写才有的问题，因此理解时一定要注意结合考虑多个数据拷贝下并发读写的场景。

对于一致性，可以分为从客户端和服务端两个不同的视角。

客户端
从客户端来看，一致性主要指的是多并发访问时更新过的数据如何获取的问题。

服务端
从服务端来看，则是更新如何分布到整个系统，以保证数据最终一致。

对于一致性，可以分为强/弱/最终一致性三类

从客户端角度，多进程并发访问时，更新过的数据在不同进程如何获取的不同策略，决定了不同的一致性。

强一致性
对于关系型数据库，要求更新过的数据能被后续的访问都能看到，这是强一致性。

弱一致性
如果能容忍后续的部分或者全部访问不到，则是弱一致性。

最终一致性
如果经过一段时间后要求能访问到更新后的数据，则是最终一致性。

可用性指“Reads and writes always succeed”，即服务在正常响应时间内一直可用。

好的可用性主要是指系统能够很好的为用户服务，不出现用户操作失败或者访问超时等用户体验不好的情况。可用性通常情况下可用性和分布式数据冗余，负载均衡等有着很大的关联。

分区容错性指“the system continues to operate despite arbitrary message loss or failure of part of the system”，即分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性或可用性的服务。

作为一个分布式系统，它和单机系统的最大区别，就在于网络，现在假设一种极端情况，N1和N2之间的网络断开了，我们要支持这种网络异常，相当于要满足分区容错性，能不能同时满足一致性和响应性呢？还是说要对他们进行取舍。假设在N1和N2之间网络断开的时候，有用户向N1发送数据更新请求，那N1中的数据V0将被更新为V1，由于网络是断开的，所以分布式系统同步操作M，所以N2中的数据依旧是V0；这个时候，有用户向N2发送数据读取请求，由于数据还没有进行同步，应用程序没办法立即给用户返回最新的数据V1，怎么办呢？

![Alt text](cap1.png?raw=true "Title")

有二种选择，第一，牺牲数据一致性，响应旧的数据V0给用户；第二，牺牲可用性，阻塞等待，直到网络连接恢复，数据更新操作M完成之后，再给用户响应最新的数据V1。

这个过程，证明了要满足分区容错性的分布式系统，只能在一致性和可用性两者中，选择其中一个。

. Quorum consensus 
    A quorum is the minimum number of members that must be present at any meeting to consider the proceedings of the meeting valid.



