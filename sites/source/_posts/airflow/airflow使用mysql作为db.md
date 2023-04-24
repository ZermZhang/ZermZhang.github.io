---
title: airflow使用mysql作为db
tags: ['airflow','调度器']
date: 2023-04-24
categories: ['airflow']
---
> airflow在使用过程中的各种组件是可以进行适当的替换的，方便大家在使用过程中采用比较符合自己技术栈的相关组件。
> 比如，我们可以将airflow模型链接的数据库换成mysql

1. 安装mysql服务
2. 对mysql进行一定的配置
	1. 启动mysql：`mysql`
	2. 常见需要使用的数据库：`create database airflowdb;`
	3. 检查数据库创建是否成功：`show databases;`
	4. 给指定的用户适当的权限
		1. `create user 'airflow'@'%' IDENTIFIED BY '123456';`
		2. `GRANT all privileges on airflowdb.* TO 'airlfow'@'%';`
			1. 将数据库airflowdb的所有权限赋予用户airflow，密码123456，且该用户可以在任何IP段进行操作
		3. 官方文档中提示airflow只需要CREATE和ALTER权限
		4. 但是在实际使用中发现需要如下权限：
			1. CREATE，ALTER，INSERT，UPDATE，INDEX，DEL，SELECT，DROP，ALTER，REFERENCES
	5. 授权检查：`show grants for 'airflow'@'%';`
3. airflow.cfg配置文件调整
	1. sql_alchemy_conn = mysql://airflow:123456@localhost/airflowdb
	2. sql_engine_encoding = ascii
		1. 默认值是utf-8，容易导致prefres超长的问题出现
4. 重新初始化数据库
	`airflow db init`
	如果之前用过sqlite，可以使用`airflow db reset`
	可能出现的问题：
	1. 问题
		 ![image.png](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230424112100.png)
	解决：`apt-get install -y python3-mysqldb`
	2. 问题
		 ![image.png](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230424112148.png)
	解决：创建database的时候指定ascii编码
		`create database airflowdb CHARACTER SET ascii;`