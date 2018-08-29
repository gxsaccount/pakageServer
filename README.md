# pakageServer  
启动时启动6个线程：   
RawDataAgent//数据value磁盘存储，将RawDataInfoAgent构造函数注入   
RawDataInfoAgent//数据包头信息存储   
RawDataNetAgent//数据接受，将RawDataAgent构造函数注入  
StaticDataNetAgent//统计数据接收，将StatisticDataAgent构造函数注入   
StatisticDataAgent//统计信息存储，  
DiskAgent//启动磁盘容量监测  
定义Request和Response  
RawDataInfoAgent -> RawDataAgent-> RawDataNetAgent  
StatisticDataAgent -> StaticDataNetAgent  

将这5个线程set给SystemMonister，用SystemMonister启动线程，在接受关闭请求时，SystemMonister调用所有线程的关闭函数  

RawDataNetAgent和StaticDataNetAgent采用netty实现。分别接受从客户端从来的原始数据rawdata和staticdata，  

RawDataNetAgent从c++的客户端接收数据并解析（按照包头解析），将解析的数据放入一个LinkedTransferQueue，用一个AtomicLong 记录data个数。
RawDataInfoAgent从LinkedTransferQueue中取出（drainTo bufferSize个数据到一个set中，遍历set记录每个数据在以时间戳为文件名的文件的offset，并将数据
存在一个ButeBuffer中存入文件。然后将RawdataInfo存入一个LinkedTransferQueue。RawDataInfoAgent从这个LinkedTransferQueue读取数据，add进在一个
TreeMap<分钟精度的时间，RawDataInfo的set>，存入时，判断TreeMap大小，到一定大小时并每次从TreeMap取最老时间的（大小不够的化取最大的一个key对应set）set
由线程池执行存入数据库aerospike中，aerospike查询的key是由包头信息和时间戳信息生成二进制key。

关闭时要注意将将遗留数据全部存完，在关闭各种链接。  


难点/坑：  
解析数据不对：  
linux是小端序，但是java是大端序。  
c++的struct有内存补齐机制，导致java解析时错误。  

包头定义：  
