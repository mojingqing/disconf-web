# disconf-web

修改的disconf-web项目不依赖nginx，没有前后端分离。maven项目可直接运行在tomcat上。

新建数据库，运行sql文件夹下的0-init_table.sql，1-init_data.sql文件。

修改配置文件包括：

	- jdbc-mysql.properties (数据库配置)
	- redis-config.properties (Redis配置)
	- zoo.properties (Zookeeper配置)
	- application.properties (应用配置）

都换成自己实际搭建对外开放的地址。

请看开源作者的主页https://github.com/knightliao/disconf 

使用的详细文档：https://disconf.readthedocs.io/zh_CN/latest/

disconf-web 
===========

分布式配置Web平台服务模块

推荐使用最新的Chrome或Firefox浏览.

### 登录页 ###

可以使用 admin   admin 进行登录。

### 主界面 ###
左上角可以选择APP和环境，选择之后，就会在中间出现若干个版本，

选择版本后，就会显示 APP、环境、版本 三个条件下的配置列表：

####表格中 各个列的意义是：

- APP：使用哪个APP，及它的ID
- KEY：配置文件或配置项
- 配置内容：配置文件或配置项在配置中心中的值
- 实例列表：使用此配置文件或配置项的所有实例列表，及每个实例的配置值。如果实例的配置值与配置中心的值不一致，这里会标识出来。
- 修改时间：修改此配置的最后一次时间 
- 操作：个性、删除、下载

####右上角可以

新建配置项、新建配置文件、新建APP

####表格右上方 

可以批量下载所有配置文件至本地，还可以查看ZK上的部署情况。

## How to deploy ##

###安装依赖软件###

- 安装Mysql（Ver 14.12 Distrib 5.0.45, for unknown-linux-gnu (x86_64) using  EditLine wrapper）
- 安装Tomcat（apache-tomcat-7.0.50）
- 安装 zookeeeper （zookeeper-3.3.0）
- 安装 Redis （2.4.5）

### 修改配置 ###
配置文件包括：

	- jdbc-mysql.properties (数据库配置)
	- redis-config.properties (Redis配置)
	- zoo.properties (Zookeeper配置)
	- application.properties (应用配置）

***注意，即使只有一个redis，也应该配置两个redis client，否则将造成内部错误。***

### 上线前的初始化工作 ###

**初始化数据库：**

可以参考 sql/readme.md 来进行数据库的初始化。注意顺序执行
0-init_table.sql        
1-init_data.sql         
201512/20151225.sql
20160701/20160701.sql

里面默认有6个用户（**请注意线上环境删除这些用户以避免潜在的安全问题**）

name | pwd
------- | -------
admin | admin
testUser1 | MhxzKhl9209
testUser2 | MhxzKhl167
testUser3 | MhxzKhl783
testUser4 | MhxzKhl8758
testUser5 | MhxzKhl112

如果想自己设置初始化的用户名信息，可以参考代码来自己生成用户：

    src/main/java/com/baidu/disconf/web/tools/UserCreateTools.java

### 部署War ###
打包，放到tomcat的webapps下


Disconf-client客户端使用说明：

1.	pom.xml文件中导入jar

`<dependency>
	<groupId>com.baidu.disconf</groupId>
	<artifactId>disconf-client</artifactId>
	<version>2.6.36</version>
</dependency>`

2.	在客户端应用的classpath下新增disconf.properties文件

文件示例：

```#是否使用远程配置文件
#true(默认)会从远程获取配置 false直接获取本地配置
disconf.enable.remote.conf=true
#配置服务器的host,用逗号分隔(disconf服务器地址)
disconf.conf_server_host=localhost:8080/disconf
#版本，官方推荐采用X_X_X_X
disconf.version=1_0_0_0
#采用产品线_服务名 格式
disconf.app=Online_wf
#环境
disconf.env=local
disconf.ignore=
#获取远程配置重复次数，默认是3次
disconf.conf_server_url_retry_times=3
#获取远程配置，重试时休眠时间，默认是5秒#
disconf.conf_server_url_retry_sleep_seconds=1
#用户定义的下载文件夹，远程文件下载后会放在这里。
disconf.user_define_download_dir=/workspace/Online_wf/src/main/resources/conf
#下载的文件会被迁移到classpath根路径下，默认是true
disconf.enable_local_download_dir_in_class_path=true
```

3.	在applicationContext.xml 添加Disconf启动支持

```<aop:aspectj-autoproxy proxy-target-class="true" /> 

<!-- 使用disconf必须添加以下配置 -->
<bean id="disconfMgrBean" class="com.baidu.disconf.client.DisconfMgrBean" destroy-method="destroy">
    <!--disconf配置对象扫描包,scanPackage是扫描标注了disconf注解类所在包路径 -->
	<property name="scanPackage" value="com.disconf.config" />
</bean>

<bean id="disconfMgrBean2" class="com.baidu.disconf.client.DisconfMgrBeanSecond"
		init-method="init" destroy-method="destroy">
</bean>```


配置文件注解使用

package com.disconf.config;
import org.springframework.stereotype.Service;
import com.baidu.disconf.client.common.annotations.DisconfFile;

@Service
@DisconfFile(filename = "config-demo.properties")
public class ConfigDemoProperties{

}


4.	文件托管

    <!--需要拉取更新的文件 -->  
	<!-- 使用托管方式的disconf配置(无代码侵入, 配置更改会自动reload) -->
	<bean id="configproperties_disconf"		class="com.baidu.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
		<property name="locations">
			<list>
			<value>classpath:/config-demo.properties</value>       		
			</list>
		</property>
	</bean>

	<!--内容有变动，重新reload -->
	<bean id="propertyConfigurer"		class="com.baidu.disconf.client.addons.properties.ReloadingPropertyPlaceholderConfigurer">
		<property name="ignoreResourceNotFound" value="true" />
		<property name="ignoreUnresolvablePlaceholders" value="true" />
		<property name="propertiesArray">
			<list>
				<ref bean="configproperties_disconf" />
			</list>
		</property>
	</bean>




