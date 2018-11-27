---
layout: post
title:  "2018-11-25-Beanstalkd 一个高性能、轻量级的分布式内存队列系统【转载】"
date:   2018-11-25 15:00:13 +0000
categories: 战神悟空
---

作者：战神悟空
链接：https://www.jianshu.com/p/391d847dc872
來源：简书



### 介绍
Beanstalkd，一个高性能、轻量级的分布式内存队列系统，最初设计的目的是想通过后台异步执行耗时的任务来降低高容量Web应用系统的页面访问延迟，支持过有9.5 million用户的Facebook Causes应用。后来开源，现在有PostRank大规模部署和使用，每天处理百万级任务。Beanstalkd是典型的类Memcached设计，协议和使用方式都是同样的风格，所以使用过memcached的用户会觉得Beanstalkd似曾相识。



###### 官网

```
https://kr.github.io/beanstalkd/
```





### 安装

###### centos

```bash
yum install beanstalkd --enablerepo=epel
```

######  源码

```bash
tar -zxvf /usr/bin/beanstalkd/beanstalkd-1.10.tar.gz

cd beanstalkd

make install PERFIX=/usr/bin/beanstalkd
```

参考 http://kr.github.io/beanstalkd/download.html



###### 启动

```bash
/usr/bin/beanstalkd -l 0.0.0.0 -p 11300 -b /var/lib/beanstalkd/binlog -F &
```

###### beanstalkd参数

```bash
/usr/bin/beanstalkd -h
Use: /usr/bin/beanstalkd [OPTIONS]

Options:
-b 开启binlog，断电后重启会自动恢复任务。
-f MS fsync最多每MS毫秒
-F 从不fsync（默认）
-l ADDR侦听地址（默认为0.0.0.0）
-p 端口侦听端口（默认为11300）
-u USER成为用户和组
-z BYTES设置最大作业大小（以字节为单位）（默认值为65535）
-s BYTES设置每个wal文件的大小（默认为10485760） （将被舍入到512字节的倍数）
-c 压缩binlog（默认）
-n 不要压缩binlog
-v 显示版本信息
-V 增加冗长度
-h 显示这个帮助
```

###### 配置文件

```
/etc/sysconfig/beanstalkd
```

### 概念

###### 核心概念

```
job：一个需要异步处理的任务，是 Beanstalkd 中的基本单元，需要放在一个 tube 中。

tube：一个有名的任务队列，用来存储统一类型的 job，是 producer 和 consumer 操作的对象。

producer：Job 的生产者，通过 put 命令来将一个 job 放到一个 tube 中。

consumer：Job的消费者，通过 reserve/release/bury/delete 命令来获取 job 或改变 job 的状态。
```





###### job 的生命周期

![](https://github.com//fyflzjz/fyflzjz.github.io/blob/master/image/2018-11-25-001/20181125001-001.png)

​	当producer直接put一个job时，job就处于READY状态，等待consumer来处理，如果选择延迟put，job就先到DELAYED状态，等待时间过后才迁移到READY状态。

​	consumer获取了当前READY的job后，该job的状态就迁移到RESERVED，这样其他的consumer就不能再操作该job。

​	当consumer完成该job后，可以选择delete, release或者bury操作；

​	delete之后，job从系统消亡，之后不能再获取；

​	release操作可以重新把该job状态迁移回READY（也可以延迟该状态迁移操作），使其他的consumer可以继续获取和执行该job；

​	有意思的是bury操作，可以把该job休眠，等到需要的时候，再将休眠的job kick回READY状态，也可以delete BURIED状态的job。

​	正是有这些有趣的操作和状态，才可以基于此做出很多意思的应用，比如要实现一个循环队列，就可以将RESERVED状态的job休眠掉，等没有READY状态的job时再将BURIED状态的job一次性kick回READY状态。

```
READY -    需要立即处理的任务，当延时 (DELAYED) 任务到期后会自动成为当前任务；
DELAYED -  延迟执行的任务, 当消费者处理任务后, 可以用将消息再次放回 DELAYED 队列延迟执行；
RESERVED - 已经被消费者获取, 正在执行的任务。Beanstalkd 负责检查任务是否在 TTR(time-to-run) 内完成；
BURIED -   保留的任务: 任务不会被执行，也不会消失，除非有人把它 "踢" 回队列；
DELETED -  消息被彻底删除。Beanstalkd 不再维持这些消息。
```



###### 一些特性

- 优先级

  任务 (job) 可以有 0~2^32 个优先级, 0 代表最高优先级，默认优先级为1024。

- 持久化

  可以通过binlog将job及其状态记录到文件里面，在Beanstalkd下次启动时可以通过读取binlog来恢复之前的job及状态。
  
- 分布式容错

	分布式设计和Memcached类似，beanstalkd各个server之间并不知道彼此的存在，都是通过client来实现分布式以及根据tube名称去特定server获取job。
	
- 超时控制

	为了防止某个consumer长时间占用任务但不能处理的情况，Beanstalkd为reserve操作设置了timeout时间，如果该consumer不能在指定时间内完成job，job将被迁移回READY状态，供其他consumer执行。
	
### 客户端操作

```
项目地址：https://github.com/pda/pheanstalk/
```

##### 1、向队列中添加job

```php
<?php

//创建队列消息

require_once('./vendor/autoload.php');

use Pheanstalk\Pheanstalk;

$pheanstalk = new Pheanstalk('127.0.0.1',11300);

$tubeName='user_eamil_message_list';

$jobData=array(
    'uid' => time(),
    'email' => 'wukong@qq.com',
    'message' => 'Hello World !!',
    'dtime' => date('Y-m-d H:i:s'),
);

$pheanstalk ->useTube( $tubeName) ->put( json_encode( $jobData));

echo json_encode($jobData).PHP_EOL;
echo 'Success ~~'.PHP_EOL;
```

##### 2、从队列中取出job

```php
<?php


//消费队列消息

require_once('./vendor/autoload.php');

use Pheanstalk\Pheanstalk;

$pheanstalk = new Pheanstalk('127.0.0.1',11300);

$tubeName='user_eamil_message_list';

while(true){
    //获取队列信息,reserve 阻塞获取
    $job = $pheanstalk ->watch($tubeName) ->ignore('default') ->reserve();
    $data=$job->getData();

    //执行相关逻辑代码
    $ret = file_put_contents('./send_mail.log',$data,FILE_APPEND | LOCK_EX);
    if( $ret ){
        echo '执行成功'.PHP_EOL.$data.PHP_EOL;
        $pheanstalk->delete($job);
    }

    //暂停(不可能是百分百的准确，跟系统的调度、CPU时钟周期等有一定关系)
    usleep(500000);
}
```

##### 3、检查服务状态

```php
<?php

//监控服务状态
require_once('./vendor/autoload.php');

use Pheanstalk\Pheanstalk;

$pheanstalk = new Pheanstalk('127.0.0.1',11300);
$isAlive = $pheanstalk->getConnection()->isServiceListening(); 

var_dump($isAlive);

/**

可以开发监控面板，监控数据的，有多少tube，多少队列，多少延迟等等


//查看beanstalkd状态
//var_dump($pheanstalk->stats());

//查看有多少个tube
//var_dump($pheanstalk->listTubes());

//设置要监听的tube
$pheanstalk->watch('test');

//取消对默认tube的监听，可以省略
$pheanstalk->ignore('default');

//查看监听的tube列表
//var_dump($pheanstalk->listTubesWatched());

//查看test的tube当前的状态
//var_dump($pheanstalk->statsTube('test'));

*/
```

###### 

###### 生产消息

```bash
$ php  producer.php 
{"uid":1499569740,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:00"}
Success ~~

php  producer.php 
{"uid":1499569742,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:02"}
Success ~~

 php  producer.php 
{"uid":1499569744,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:04"}
Success ~~
```



##### 消费消息

```bash
$ php consumer.php 
执行成功
{"uid":1499569740,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:00"}
执行成功
{"uid":1499569742,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:02"}
执行成功
{"uid":1499569744,"email":"wukong@qq.com","message":"Hello World !!","dtime":"2017-07-09 11:09:04"}
```



### PHP监控

<https://github.com/kr/beanstalkd/wiki/Tools>
 <https://github.com/ptrofimov/beanstalk_console>
 <https://github.com/jimbojsb/bstools>

参考：
 <https://segmentfault.com/a/1190000002784775>
 http://www.jianshu.com/p/413676e1696f