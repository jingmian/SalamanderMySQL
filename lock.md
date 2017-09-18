# MySQL锁概述

MySQL 两种锁特性归纳：  
表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。  
行级锁：开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率最低，并发度也最高。  
MySQL 不同的存储引擎支持不同的锁机制。  
MyISAM 和 memory 存储引擎采用的是**表级锁**  
InnoDB 存储引擎既支持行级锁，也支持表级锁，但默认情况下采用**行级锁**。  

表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如 web 应用。  
而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用。

# MyISAM的锁

1. MyISAM的表级锁模式
MySQL 的表级锁有两种模式，表共享读锁(table read lock)和表独占写锁（table write lock）。  
对 MyISAM 表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM 表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM  表的读操作和写操作之间，以及写操作之间时串行的。  
当一个线程获得对一个表的写锁户，只有持有锁的线程可以对表进行更新操作，其他线程的读、写操作都会等待，直到锁被释放。

2. 加锁
MyISAM 在执行查询语句（select）前，会自动给涉及的所有表加读锁，在执行更新操作（update、delete、insert等）前，会自动给涉及的表加**写锁**，这个过程并不需要直接用 lock table 命令给 MyISAM 表显示加锁。给 MyISAM 表显式加锁，一般是为了在一定程度模拟事务操作 
MyISAM 在自动加锁的情况下，总是一次获得 sql 语句所需要的全部锁，所以显示锁表的时候，必须同时取得所有涉及表的锁，这也正是 MyISAM 表不会出现死锁（deadlock）的原因。