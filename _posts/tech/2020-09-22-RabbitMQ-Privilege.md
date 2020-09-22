---
layout: post
title: RabbitMq权限管理
category: 技术
tags: RabbitMq权限管理
description: RabbitMq权限管理
---

## 概述
 在RabbitMQ中，用户是访问控制（Access Control）的基本单元，包含用户角色与用户权限二个部分。用户角色是RabbitMQ预置的用户权限组，设置某个角色即获得对应的用户组的权限，使单个用户可以跨越多个vhost进行授权。用户权限是权限控制则是以vhost为单位的，对exchange，queue的操作权限控制，包括配置权限，读写权限。配置权限。
## RabbitMQ用户角色
RabbitMQ角色分为五大类：
•	超级管理员(administrator)
    可登陆管理控制台(启用management plugin的情况下)，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
•	监控者(monitoring)
    可登陆管理控制台(启用management plugin的情况下)，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
•	策略制定者(policymaker)
    可登陆管理控制台(启用management plugin的情况下), 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
•	普通管理者(management)
    仅可登陆管理控制台(启用management plugin的情况下)，无法看到节点信息，也无法对策略进行管理。
•	默认角色(none)
    无法登陆管理控制台，通常就是普通的生产者和消费者。

## RabbitMQ用户权限

用户权限指的是用户对exchange，queue的操作权限，包括配置权限，读写权限。配置权限会影响到exchange，queue的声明和删除。

读写权限影响到从queue里取消息，向exchange发送消息以及queue和exchange的绑定(bind)操作。例如： 将queue绑定到某exchange上，需要具有queue的可写权限，以及exchange的可读权限；向exchange发送消息需要具有exchange的可写权限；从queue里取数据需要具有queue的可读权限。详细请参考官方文档中"How permissions work"部分。
        
AMQP协议中并没有指定权限在vhost级别还是在服务器端级别实现，由具体的应用自定义。在RabbitMQ中，权限控制则是以vhost为单位的。当创建一个用户时，用户通常会被指派给至少一个vhost，并且只能访问被指派vhost内的队列、交换器以及绑定关系。因此，RabbitMQ中的授予权限是指在vhost级别对用户而言的权限授予。

相关的授予权限命令为：rabbitmqctl set_permissions [-p vhost] {user} {conf} {write} {read}。其中各个参数的含义为：

vhost：授予用户访问权限的vhost名称，可以缺省，即vhost为”/”。
user：可以访问指定vhost的用户名称。
conf：一个用于匹配用户在哪些资源上拥有可配置权限的正则表达式。
write：一个用于匹配用户在哪些资源上拥有可写权限的正则表达式。
read：一个用于匹配用户在哪些资源上有用可读权限的正则表达式。
注：可配置指的是队列和交换器的创建以及删除之类的操作；可写指的是发布消息；可读指与消息有关的操作，包括读取消息以及清空整个队列等。

## 结论与建议
建议创建三个权限账号

•	超级管理员账户，一般情况下，不建议使用，仅在删除queue，添加、修改策略等特殊场景下使用。

•	进程读写账户，设置为默认角色(none)并赋予读、写、配置权限。

•	监控账户，设置为监控者角色并赋予读权限，禁止配置、写权限。


附RabbitMQ权限管理命令：（rabbitmq权限检测仅发生在连接建立时，如果权限发生改变，client需重启才能生效）

    角色管理：
        (1) 创建用户(默认角色为空)
            rabbitmqctl  add_user  Username  Password
        (2) 设置用户角色
            rabbitmqctl  set_user_tags  User  Tag
            User为用户名， Tag为角色名(对应于上面的administrator，monitoring，policymaker，management)。
            也可以给同一用户设置多个角色，例如:
            rabbitmqctl  set_user_tags  User  monitoring  policymaker
        (3) 查看所有用户及其角色
            rabbitmqctl list_users
    权限管理
        (1) 设置用户权限
            rabbitmqctl  set_permissions  -p  VHostPath  User  ConfP  WriteP  ReadP
            如：开启vhost1配置、读、写权限 rabbitmqctl  set_permissions -p vhost1 user1 '.*' '.*' '.*' 
            仅开启vhost1读权限 rabbitmqctl  set_permissions -p vhost1 user1 '^$' '^$' '.*' 
        (2) 查看(指定hostpath)所有用户的权限信息
            rabbitmqctl  list_permissions  [-p  VHostPath]
        (3) 查看指定用户的权限信息
            rabbitmqctl  list_user_permissions  User
        (4) 清除用户的权限信息
            rabbitmqctl  clear_permissions  [-p VHostPath]  User
