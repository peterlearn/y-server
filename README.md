### 目录结构说明
* data 存放数值配置导出的erl文件
* sql   存放数据库建表、改表结构的SQL脚本
* deps 项目依赖的第三方库
* ebin 存放项目编译后的beam文件
* etc  存放项目的配置文件
* include 存放头文件
* log     存放日志文件
* priv    Web管理后台的根目录
* proto   存放protobuf协议描述文件
* robot   存放机器人测试程序相关代码
* src     存放项目各模块源代码
    admin    管理后台
    common   公用工具方法库
    core     监控数、网络等基础功能
    fight    战斗模块
    id_mgr   全局唯一ID管理模块
    log_mgr  运营日志管理模块
    memcache 缓存管理模块
    role     角色模块
    scene    场景模块
* tools   存放项目中使用的各种脚本工具

### 软件环境
* erlang R18以上
* mysql
* python 
* nodejs
    1. 内网npm源设置
        在host中加入以下配置:
        172.16.10.69 cdn.npm.taobao.org 
        172.16.10.69 registry.npm.taobao.org
    2. npm配置
        NPM config set strict-ssl false
        NPM config set registry http://registry.NPM.taobao.org/
    3. 安装protobufjs
        npm install protobufjs@6 -g

### 网络
###### 协议
    网络层支持TCP和WebSocket,应用层协议使用ProtoBuf进行序列化
    WebSocket协议详情可参考 https://github.com/zhangkaitao/websocket-protocol

* 应用层协议格式(web socket的payload) Module+Message=Cmd PB=application data
    Len(32bit) + Seq(16bit) + Module(8bit) + Message(8bit) + PB
* PB
    客户端=>服务端的消息名以 c2s 开头
    服务端=>客户端的消息名以 s2c 开头

###### 断线
* 客户端主动断开连接
    mmo_network进程退出，玩家进程收到EXIT消息
* 定时心跳检测
    心跳连续超时N次，玩家进程收到keepalive_timeout消息
    断线后执行network_detached方法，延迟关闭玩家进程
* 运营后台踢人
* 停服维护前踢人

### 定时器
* 玩家定时器
    lib_proc_timer实现的低精度定时器
* cron
    类似操作系统crontab的定时器，每分钟检测一次，触发时间可配置
    例如隔天刷新，每天凌晨0点触发一次lib_misc:next_day_refresh/0调用，可在此函数中添加各个模块隔天刷新的回调接口

### 配置文件
* etc/mmo.rc
    环境变量配置
* etc/mmo.config   
    节点应用相关配置信息,如数据库配置、TCP网络配置
* etc/common.config
    游戏的一些通用配置,如账号服务器接口地址、平台登录接口地址等
* etc/game.config
    游戏的一些个性化配置,如果是否开启GM指令等
    游戏节点启动时会将common.config与game.config中的配置项进行合并(相同的配置项在common.config与game.config中都存在,则以game.config中的值为准),
    然后编译为game_config_mod模块,可通过?CONFIG(Key) 或者 ?CONFIG(Key, DefaultVal) 来获取配置项的值

### 运营日志
    每个日志表对应一个serv_log进程, serv_log负责写日志到数据库中。当serv_logs收到写日志请求后，先缓存日志信息，然后判断缓存的日志记录数大于一定数量，
    或者超过一定时间后再批量将日志信息写入数据库,以减轻写运营日志的压力。

### 更新说明
    tools目录下存放了启动服务器、停止服务器、更新数据库、编译协议文件等脚本工具,
    linux系统中, 在项目根目录下执行svn up可从SVN服务器获取最新的代码,然后运行make rebuild即可编译项目

* tools/start.sh  启动服务器  
* tools/stop.sh   关闭服务器  
* tools/db\_update.sh 更新数据库  

### 发版说明
##### 举例发版：trunk版本
1. 机器172.16.11.207，cd到发版源码目录：/data/update_server/h5/trunk
2. 更新代码：svn up
2. 编译代码：make rebuild
3. 执行发版脚本：h5_faban.sh
4. 到蓝海网页点发版：https://dts-lan.gz4399.com/release/make  
\---------------------------------------------------------------------
5. 切换到外网发版机，机器121.201.47.73，关闭要更新的服
6. cd到发版机目录：/data/server/CN/stable/web_4399/s0
7. 同步文件：bitmap, ebin, include, sql 到各个服根目录
8. 数据库更新（无更新忽略这步）：
    * 更新sql：  
        game库执行：sql/update\_game.sql  
        log库执行：sql/update\_log.sql  
    * 数据转换：  
        cd到个服tools目录，执行脚本：version\_transfer.sh
9. 启动服务器

### 蓝海流程说明
* 例行维护-后端：同步服务器目录文件：bitmap ebin include sql，更新数据库，重启服务器 [**要审核**]
* 闪更-后端：同_例行维护-后端_，区别：[**不能更新数据库，无需审核**]
* 热更-后端ebin：可以指定ebin tools下的文件更新到队列服务器

