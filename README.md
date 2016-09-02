# saltstack

学习Saltstack已经有一段时间了，不过现在我还是不知道如何对Saltstack做一个全面的定义。按照大家公认的说法，我们可以这样来定义Saltstack，一个整合了Puppet和 Chef的功能，更加强大，更适合大规模批量管理服务器的自动化工具，基于ZeroMQ通信，使用python开发的简单高可用工具。还是按照之前的老套路，先从安装和入门说起。

### 跟ansible 区别
ansible差不多是基于ssh封装的 slat是要安装agent,然后通宵是zeromq实现的


### 引言:自动化运维 
### 1:saltstack 的基本介绍 
### 2:salt 的安装
* 服务端 
    * 安装
    * 配置文件 
    * 运行 
    * 注意事项
* 客户端 
    * 安装
    * 配置文件
    * 运行
    * 注意事项

### 3:salt 的使用: 
* 基础知识
   * targeting 
   * nodegroup 
   * grains 
   * pillar
* 状态管理 
   * state
   * state 语法
   * state 的逻辑关系 
   * highstate
   * salt schedule 
* 实时管理 
   * cmd.run
   * module 
* 无 master
### 4:salt 开发
* saltclient 管理 salt
  * salt api
*************************************************************
### 引言:自动化运维 运维的工作主要在2方面:
1. 状态的管理
2. 系统性能调优 这里主要简介下运维状态的管理:
对于运维来说,基于状态的配置管理已经向自动化迈进了一大步,以状态为核心的运维,
让状态本身有了可管理性;在运维过程中我们会发现,同样的一个配置,我们会在不同的时间, 不同的地点一次在一次的配置,这个时候,配置管理就有了重复性;有的甚至是原封不动的重 复,而另外一些则是按照一定的规律在发展,这些按照一定规律发展的配置,就是可预测的.综 上我认识的,我们运维工作的本身是可管理,可重复,可预测的.基于这样的理念,我们就可以更 高一步的推进我们的运维自动化,甚至到智能化.

看到这里,我理想中的运维自动化的配置管理平台应该有如下功能: 

1. 对状态的配置及管理(最基本的) 
2. 可以及时的对系统状态进行调整并能看到结果(可以方便的实时升级系统状态)
3. 可以对其本身做方便的第三方管理(借助其 API,直接给状态制定好其发展方向)

#####  加分项: 
1. 开发语言单一
2. 架构简单,灵活 
3. 有不差的安全性 
4. 没有商业版
下面是现有比较有代表性的自动化配置管理工具: 附:以下仅基于开源版本进行介绍

#### 理念 优缺点
puppet 从运维的角度去做配置管理(运维人员做给运维用的) 架构简单,系统比较成
熟/不便于第三方管理

chef 从研发的角度去做配置管理(研发人员做给运维用的) 较便于第三方管理,对
自身(节点,变量,cookbook)的管理较方便,有自己的 dashboard,cookbook 支持版本管理,自从 cookbook 的版本管理/架构复杂,开发语言较多,(安全问题)

* 以上 2 者都比较侧重于状态的管理,对单个或者多个状态的临时调整或者管理较差 2 个都有商业版,让我这个用开源版的很自卑

这里我们也能看到,2 个配置功能都没能达到我理想中的状态,那就暂用 chef 吧,直到有 一天,了解到了 saltstack 看到了这句话:“ 系统配置的自动化不仅可预测,可重复, 还具有可 管理性”(http://wiki.saltstack.cn/reproduction/dive-into-saltstack),这尼玛才是运维自动化的未 来啊,于是毫无节操的开始学习 salt,而且发现越学越喜欢;在我使用 puppet 及 chef 的时 候都没有使用 salt 的感觉,太爽了。所以我这里仅仅介绍几本的语法不涉及实际用例,salt 的安装非常方便,所以你在看本文档的时候希望你能真正的动手去做一下,然后你就会爱上 salt 了

``` bash 
附:如果你会 python,salt 基本是不需要怎么学的,而我正好了解一点 py,不过这最多占我选择 salt 的 20%。
```
### 1:saltstack 的基本介绍
salt 是一个新的基础平台管理工具。很短的时间即可运行并使用起来, 扩展性足以支撑
管理上万台服务器,数秒钟即可完成数据传递. 经常被描述为 Func 加强版+Puppet 精简版。

salt 的整个架构都是基于消息来实现.所以能够提供横强的拓展性,灵活性,实时性;不夸 了,看实际的 slat 是什么样的不过 salt 还是一个很年轻的东西,还有很多地方不够完善,做的不够好,不过我相信这 些都只是时间问题。

* 注:以下文档仅仅为基本内容,相关知识点的深入学习,请看相应文档连接 ##2:salt 的安装

安装有很多途径,作为一个 centos 的用户,我选择 rpm 首先添加 RPM 源:
rpm -ivh http://mirrors.sohu.com/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm

##### 附:实际生产中我是自建源
epel 中关于 salt 的包:

``` bash 
salt-api.noarch : A web api for to access salt the parallel remote execution system salt-master.noarch : Management component for salt, a parallel remote execution
system
salt-minion.noarch : Client component for salt, a parallel remote execution system salt.noarch : A parallel remote execution system
salt-cloud.noarch : Generic cloud provisioning tool
```
1:服务端 1:安装

``` bash 
yum install salt-master 2:配置文件
/etc/salt/master
配置文件选项介绍: http://docs.saltstack.com/ref/configuration/master.html
最基本字段:
interface: 服务端监听 IP
3:运行 调试模式:
salt-master -l debug 后台运行:
salt-master -d
作为 centos 管理员,我选择:
/etc/init.d/salt-master start 

4:注意事项:
1:监听端口
4505(publish_port):salt 的消息发布系统 4506(ret_port):salt 客户端与服务端通信的端口
所以确保客户端能跟服务端的这2个端口通信 
```

#### 安装客户端

```bash 
yum install salt-minion 2:配置文件
/etc/salt/minion
配置文件选项介绍: http://docs.saltstack.com/ref/configuration/minion.html 最基本字段:
master: 服务端主机名
id: 客户端主机名(在服务端看到的客户端的名字)
3:运行 调试模式:
salt-minion -l debug 后台运行:
salt-minion -d
作为 centos 管理员,我选择:
/etc/init.d/salt-minion start 

4:注意事项:
1:minion 默认和主机名 salt 的主机通信 

2:关于配置文件
salt 的配置文件均为 yaml 风格
$key: $value #注意冒号后有一个空格 3:基础知识
1:salt minion 和 master 的认证过程:
(1) minion 在第一次启动时,会在/etc/salt/pki/minion/下自动生成
minion.pem(private key), minion.pub(public key),然后将 minion.pub 发送给 master
(2) master 在接收到 minion 的 public key 后,通过 salt-key 命令 accept minion public key,这样在 master 的/etc/salt/pki/master/minions 下的将会存放以 minion id 命名的
public key, 然后 master 就能对 minion 发送指令了
如下: 
启动服务端:
/etc/init.d/salt-minion restart
启动客户端:
/etc/init.d/salt-minion restart 

服务端查看key:
salt-key
Accepted Keys:
Unaccepted Keys: minion1
Rejected Keys:
服务端接受 key salt-key -a minion1

测试:
salt 'minion1' test.ping minion1:
True
附:salt 更多命令及手册 salt '*' sys.doc

```
### 3:salt 的使用: 1:基础知识

```
1:targeting
salt '*' test.ping
引号中以实现很强大的 minion 的过滤与匹配技术 文档:http://docs.saltstack.com/topics/targeting /compound.html
常用命令:
salt 'shell 正则' 命令
salt -E 'prel 正则'
salt -N $group 命令
salt -L 'server_id1,server_id2,server_id3' 命令
示例:
salt -C 'webserv* and G@os:Debian or E@web-dc1-srv.*' test.ping
2:nodegroup
对 minion 进行分组
文档: http://docs.saltstack.com/topics/targeting/nodegroups.html 在 master 的配置文件中按如下格式定义:
nodegroups:
group1: 'L@foo.domain.com,bar.domain.com,baz.domain.com or
bl*.domain.com'
group2: 'G@os:Debian and foo.domain.com' 在 state 或者 pillar 中引用的时候如下:
base: group1:
- match: nodegroup - webserver

```
3:grains
minion 基本信息的管理 文档:http://docs.saltstack.com/topics/targeting /grains.html 基本使用:

``` bash 
salt '*' grains.ls 查看 grains 分类
salt '*' grains.items 查看 grains 所有信息
salt '*' grains.item osrelease 查看 grains 某个信息
示例:
salt '*' grains.item osrelease minoin1:
osrelease: 6.2
```

### minion 的变量

在用 salt 进行管理客户端的时候或者写 state 的时候都可以引用 grains 的变量 4:pillar

salt 敏感信息的管理,只有匹配到的节点才能看到和使用 文档:http://docs.saltstack.com/topics/tutorials/pillar.html 默认:pillar 数据定义文件存储路径:/srv/pillar 入口文件:/srv/pillar/top.sls
``` bash 
格式: base:
"targeting":
- $pillar
$pillar.sls 基本:
$key: $value state 引用方式:
{{ pillar['$key'] }}
复杂:
users:
thatch: 1000 shouse: 1001 utahdave: 1002 redbeard: 1003
state 引用方式:
{% for user, uid in pillar.get('users', {}).items() %}
{{user}}: user.present:
- uid: {{uid}} {% endfor %}
查看节点的 pillar 数据: salt 'client2' pillar.data
同步 pillar:
salt '*' saltutil.refresh_pillar
附:这里我们可以看到,pallar 中也可以使用 jinja(后面会提到)做一些处理 5:minion
即为 salt 的客户端
2:状态管理 1:state
salt 基于 minion 进行状态的管理 1:state 语法
文档:http://docs.saltstack.com/ref/states/all/index.html
#名字为pillar.sls的文件来存放对匹配到的

{% if grains['os_family'] == 'RedHat' %} - name: vim-enhanced
{% elif grains['os'] == 'Debian' %}
- name: vim-nox
{% elif grains['os'] == 'Ubuntu' %} - name: vim-nox
{% endif %}
- installed
如果是 redhard 系列的就安装 vim-enhanced,如果系统是 Debian 或 者 Ubuntu 就安装 vim-nox
以有多个
附:state 默认使用 jinja(http://jinja.pocoo.org/)的模板语法, 文档地址: http://jinja.pocoo.org /docs/templates/
```
2:state 的逻辑关系: 文档:http://docs.saltstack.com/ref/states/ordering.html

``` bash 
require:依赖某个 state,在运行此 state 前,先运行依赖的 state,依赖可
httpd: pkg:
- installed file.managed:
- name: /etc/httpd/conf/httpd.conf 
- source: salt://httpd/httpd.conf
- require:
- pkg: httpd
```
watch:在某个 state 变化时运行此模块 redis:

``` bash
pkg:
- latest
file.managed:
- source: salt://redis/redis.conf
- name: /etc/redis.conf
- require:
- pkg: redis service.running: 
- enable: True
#state 的名字
```
$state: #要管理的模块类型 - $State states #该模块的状态

``` bash 
- watch:
- file: /etc/redis.conf 
- pkg: redis
```
附:watch 除具备 require 功能外,还增了关注状态的功能

```
order:
优先级比 require 和 watch 低
有 order 指定的 state 比没有 order 指定的优先级高 vim:
pkg.installed: - order: 1
想让某个 state 最后一个运行,可以用 last 3:state 与 minion

将临时给 minoin 加个 state
salt 'minion1' state.sls 'vim' #给 minion1 加一个 vim 的 state 执行该命令后可以立即看到输出结果

2:highstate
给 minion 永久下添加状态
文档: http://docs.saltstack.com/ref/states/highstate.html 默认配置文件:/srv/salt/top.sls
语法:
'*':
- core
- wsproxy 
- /srv/salt/目录结构:
.
├── core.sls ├── top.sls └── wsproxy
├── init.sls ├── websocket.py └── websockify
应用:
salt "minion1" state.highstate

测试模式:
salt "minion1" state.highstate -v test=True 3:salt schedule
默认的 state 只有在服务端调用的时候才执行,很多时候我们希望 minon 自觉 的去保持在某个状态

文档:http://docs.saltstack.com/topics/jobs/schedule.html
cat /srv/pillar/top.sls base:
"*":
- schedule
cat /srv/pillar/schedule.sls schedule:
highstate:
function: state.highstate minutes: 30

如上配置:
minion 会没 30 分钟从 master 拉去一次配置,进行自我配置

3:实时管理 有时候我们需要临时的查看一台或多台机器上的某个文件,或者执行某个命令
1:cmd.run
用法 salt '$targeting' cmd.run '$cmd'
示例:salt '*' cmd.run 'hostname' 执行下这样的命令,马上就感受到效果了,速度还贼快
2:module
同时,salt 也将一些常用的命令做了集成 文档:http://docs.saltstack.com/ref/modules/all/index.html 这里几乎涵盖了我们所有的常用命令
比如:
查看所有节点磁盘使用情况
salt '*' disk.usage 文档:http://docs.saltstack.com/topics/tutorials/quickstart.html
4:无 master
主要应该在测试和 salt 单机管理的时候
```
### 4:salt 开发
1:saltclient 管理 salt
只有才 master 才可以salt 全部用 python,这里也用 python 
文档:http://docs.saltstack.com/ref/python-api.html 

示例:

```bash
import salt.client
client = salt.client.LocalClient()
ret = client.cmd('*', 'cmd.run', ['ls -l'])
print ret
```
2:salt api
salt api 我现在还没用,不过我也没搞定,如果你搞定了,我会非常感谢你能分享
下的。

```
####参考文档:

1:salt 中文 wiki:http://wiki.saltstack.cn/ 很不错的文章:http://wiki.saltstack.cn/reproduction/dive-into-saltstack
2:salt 官网 http://saltstack.com/ 官网文档:http://docs.saltstack.com/


