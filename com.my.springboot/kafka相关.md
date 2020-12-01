#

##topic主题
    类似于关系型数据库的表
    topic是逻辑概念

##partition分区
    分区具体在服务器上面表现起初就是一个目录，一个topic下有多个分区，这些分区会存储到不同的服务器上面。
    分区其实就是在不同的主机上建了不同的目录，这些分区主要的信息就存在了.log文件里。跟数据库里的分区差不多，是为了提高性能
    partition就是分布式的存储单元，是保证海量数据处理的基础
    分区会有单点故障问题，我们为每个分区设置副本数，分区编号从0开始
    
##producer生产者


##consumer消费者


##message消息


##集群架构

##replica副本
    为保证数据安全，每个partition会设置多个副本replica
    每个副本都有角色之分，选取一个副本作为leader，其余的作为follower，我们的生产者在发送数据的时候，是直接发送到leader partition中，
        然后follower partition会去leader那里自行同步数据，消费者消费数据的时候，也是从leader那去消费
    
##consumer group 消费者组
    消费数据时会在代码里面指定一个group.id,这个id代表的是消费组的名字，而且这个group.id就算不设置，系统也会默认设置
    一般的消息系统，只要有一个消费者去消费了消息系统里面的数据，那么其余的所有消费者都不能再去消费这个数据
    kafka中的处理：在consumera去消费了一个topica里面的数据，再让consumerb也去消费topica的数据，是消费不到的。但我们再consumerc中重新指定另外的group.id，
        consumerc是可以消费到topica的数据的，而consumerd也是消费不到的。kafka中，不同组可以有唯一的一个消费者去消费同一主题的数据
    消费者组让多个消费者并行消费信息而存在，而且他么不会消费到同一个消息，consumera，b，c是不会相互干扰的    
    一个分区不会让消费者组里面的多个消费者去消费，但是在消费者不饱和的情况下，一个消费者可以去消费多个分区的数据
##controller
    在大数据分布式文件系统里，95%的都是主从式的架构，个别是对等式的架构比如elasticsearch
    kafka也是主从式的架构，主节点就叫controller，其余的是从节点，controller是需要和zookeeper进行配合管理整个kafka集群
    
#与springboot的整合使用
    引入依赖：
             <!-- kafka -->
             <dependency>
                 <groupId>org.springframework.kafka</groupId>
                 <artifactId>spring-kafka</artifactId>
             </dependency>
    配置文件：
        topinfo:
             # kafka集群配置 ,bootstrap-servers 是必须的
           kafka:
              # 生产者的kafka集群地址
              bootstrap-servers:  192.168.90.225:9092,192.168.90.226:9092,192.168.90.227:9092 
              producer: 
                 topic-name:  topinfo-01
                 
              consumer:
                 group-id:  ci-data
    添加属性配置类：
```
        @ConfigurationProperties(prefix = "topinfo.kafka")
        @Component
        public class KafKaConfiguration {
        
            /**
             * @Fields bootstrapServer : 集群的地址
             */
            private String bootstrapServers;
        
            private Producer producer;
        
            private Consumer consumer;
        
            public String getBootstrapServers() {
                return bootstrapServers;
            }
        
            public void setBootstrapServers(String bootstrapServers) {
                this.bootstrapServers = bootstrapServers;
            }
        
            public Producer getProducer() {
                return producer;
            }
        
            public void setProducer(Producer producer) {
                this.producer = producer;
            }
        
            public Consumer getConsumer() {
                return consumer;
            }
        
            public void setConsumer(Consumer consumer) {
                this.consumer = consumer;
            }
        
        }
``` 
    添加kafka配置类                        


