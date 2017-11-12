---
layout: post
title: "网站技术架构总结"
subtitle: "最近看完了《大型网站技术架构核心原理与案例分析》这本书，书里主要涉及的都是技术知识而非代码实现，读下来之后感觉对自己的知识面还是有加深的，起到了很好的深化作用，巩固了之前的薄弱环节，而且对于面试时一些架构设计、分布式数据一致性问题都感到有了更好的回答思路，总之确实是一本适合入门的好书。"
date: 2017-11-09
author: "ChenJY"
header-img: "img/websitear.jpg"
catalog: true
tags: 
    - 大型网站技术架构核心原理与案例分析
    - 读后感
---

### 重点网站架构技术
#### 缓存
缓存是改善系统性能的第一手段。缓存之所以有效，因为用户的热点数据往往只集中在某20%上。

* <a href="https://en.wikipedia.org/wiki/Content_delivery_network" target="_blank"><b>CDN</b></a>，全称内容分发网络，实际上是部署在距离用户最近地点的网络服务商，用户的访问请求总是先到达距离最近的网络服务商处，在那里会缓存一些网站的静态资源，既不需要访问数据库就能获取的，这样可以第一时间返回结果给用户，改善用户体验，例如爱奇艺这样的视频网站一定是在全国各地都会有 <a href="https://en.wikipedia.org/wiki/Content_delivery_network" target="_blank"><b>CDN</b></a> 服务器部署，其实用户经常访问的热点内容会缓存在上面。

* <a href="https://en.wikipedia.org/wiki/Reverse_proxy" target="_blank"><b>反向代理</b></a>，这是属于前端架构的一部分，类似于在后台服务器集群与客户端之间架设的新的一台服务器，用于接收用户的请求。这其中，<a href="https://en.wikipedia.org/wiki/Content_delivery_network" target="_blank"><b>反向代理服务器</b></a>有两个基本功能，第一可以用作资源缓存，可以直接将重要的网站静态资源放置在此；第二因为<a href="https://en.wikipedia.org/wiki/Content_delivery_network" target="_blank"><b>反向代理服务器</b></a>特殊的部署位置，因此它还可以充当负载均衡器的作用，可以转发到后台合适的某台服务器上，这台服务器处理完用户请求之后的结果也需要通过<a href="https://en.wikipedia.org/wiki/Content_delivery_network" target="_blank"><b>反向代理服务器</b></a>发送至用户。

* 本地缓存，顾名思义，这是速度最快的访问方式，类似于现在的浏览器也会有缓存的功能。

* 分布式缓存，因为有些网站的缓存数据量太大，单机无法承受，因此搭建缓存集群尤为关键，应用程序需要通过网络通信来访问缓存的数据。

但是缓存也并不是什么时候都适合，例如在数据需要频繁修改的时候，使用缓存可能会造成应用还没来得及读取，真实数据已经改变缓存也已经失效的问题，徒增系统压力；其次，不是访问热点的数据也无需缓存；区分是否需要强一致性的场景也是判断是否适合使用缓存的关键因素；最后，如何提升缓存的可用性需要注意，一旦缓存崩溃那么数据库将直面访问压力，很容易宕机，目前基于 <a href="https://en.wikipedia.org/wiki/Memcached" target="_blank"><b>memcached</b></a> 或者 <a href="https://en.wikipedia.org/wiki/Redis" target="_blank"><b>redis-cluster</b></a> 的缓存集群越来越多的得到应用。

#### 异步
异步抽象出来就是 <a href="https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem" target="_blank"><b>生产者-消费者模式</b></a>，最常见的就是消息队列。异步的操作既可以改善用户体验、加快响应速度，又可以降低服务器的压力、削峰，又可以提高系统可用性。

#### 存储
<a href="https://en.wikipedia.org/wiki/RAID" target="_blank"><b>RAID</b></a> 廉价磁盘冗余阵列是为了改善磁盘访问延迟，增强数据可用性和容错能力。具体有以下几种技术：

* RAID0，写入磁盘是数据分成 N 部分，并发写入磁盘，但不做备份。
* RAID1，写入两块磁盘，相当于在 RAID0 的基础上复制了一块一模一样的。
* RAID3，将数据分出 N-1 份，并发写入 N-1 块磁盘，并在第 N 块磁盘记录校验数据，当前 N-1 块磁盘中数据损坏时，可以用第 N 块恢复，关于这部分有时间我会另开一篇仔细做个总结。
* RAID5，校验数据不写进第 N 块，而是分散的写进所有磁盘，因为单单写入第 N 块的话，这块磁盘容易形成访问热点，出错的概率高

关系型数据库索引一般采用 B+ 树的结构，这个前面的 Blog 中有详细的解释，NoSQL 中一般采用 LSM 树作为主要数据结构，它可以看做是一个 N 阶合并树，数据写操作都会在内存中进行，并且都会创建一个新纪录，这些数据在内存中仍然是一颗排序树，当数据量超过设定的内存或者阈值的时候，会将这棵排序树和磁盘上最新的排序树进行合并，合并过程中会用新数据覆盖旧数据。

### 网站高可用性的保障
#### 失效转移
心跳检测机制可以让负载均衡器发现某一台服务器宕机了，进而将它从服务器列表进行删除，将请求转发至其他服务器上，原来宕机的服务器可以进行自动重启或者人工干预，之后经过测试，状态迁移等操作重新加入集群中工作。

#### Session 管理
这部分在面试中经常被问到，解决的方法有，第一集群广播后迁移 session，第二集群中单独设置 session 服务器，第三采用源地址 Hash 方法一直定位到同一台服务器，第四 session 复制，每台服务器都存有所有的 session，第五还可以采用 Cookie 记录 session，每次都发到服务器端的做法。

#### 服务降级
网站在访问高峰期，服务可能因为大量的并发调用而性能下降，严重时可能导致服务宕机。为了在这期间保证核心应用和功能的正常运行，需要对服务进行降级，可以选择拒绝服务或者关闭服务。
* 拒绝服务：拒绝低优先级的调用，减少服务调用并发数，确保核心应用正常使用；或者也可以随机拒绝部分调用，节约资源，让另一部分请求得以成功。
* 关闭服务：关闭部分不重要的服务，或者服务内部关闭不重要的功能，以节约系统开销，为重要的服务和功能让出资源。

#### 数据备份
冷备，古老的手工复制拷贝技术，廉价但是不能保障数据最终一致性，可用性也比较低。
热备可以分为同步热备和异步热备两种：
* 异步热备：写入一个副本成功即返回，其他副本异步写入，这种情景下，存储服务器分为主存储服务器（Master）和从存储服务器（Slave）
* 同步热备：指多份数据副本写入完成才返回，为了提高性能，可以客户端并发写入多个存储服务器
* 实际业务中，通常还会采用读写分离的方式访问 master-slave 集群，写在 master 上，而从 slave 上读取数据

### 参考资料
* <a href="https://item.jd.com/11322972.html" target="_blank"><b>《大型网站技术架构核心原理与案例分析》</b></a> 李智慧著

### 许可协议
* 本文遵守创作共享 <a href="https://creativecommons.org/licenses/by-nc-sa/3.0/cn/" target="_blank"><b>CC BY-NC-SA 3.0协议</b></a>
* 商业用途转载请联系 Chen.Jiayang[AT]foxmail.com
* 封面图片来自 <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px;" href="https://unsplash.com/@paramir?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Ehud Neuhaus"><span style="display:inline-block;padding:2px 3px;"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white;" viewBox="0 0 32 32"><title></title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px;">Ehud Neuhaus</span></a>


