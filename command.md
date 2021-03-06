---
title: Client端命令介绍
---

DataHub Client 是 DataHub 的命令行客户端，用来执行 DataHub 相关命令。

| 命令	    | 用途              |
| :---------- | :----------       |
| dp          | Datapool 管理      |
| subs        | Subscrption 管理   |
| login       | 登录到 dataos.io   | 
| pull        | 下载数据           |
| pub         | 发布数据           |
| repo        | Repository 管理   |
| job         | 显示任务列表       |
| ep		  | 设置Entrypoint    |
| logout	  | 登出              |
| help        | 帮助命令          |  


## DataHub Client命令行使用说明

#### NOTE：
- 如果没有额外说明，所有的命令在没有错误发生时，不在终端输出任何信息，只记录到日志中。错误信息会打印到终端。
- 所有的命令执行都会记录到日志中，日志级别分[TRACE] [INFO] [WARNNING] [ERROR] [FATAL]。
- 参数支持全名和简称两种形式，例如--type等同于-t。详情见命令帮助。
- 参数赋值支持空格和等号两种形式，例如--type=file等同于--type file。


### 1. Datapool 相关命令

#### 1.1 列出所有数据池
列出所有数据池（datapool）

	datahub dp
    
输出
	
    {%DPNAME    %DPTYPE}

例子

    $ datahub dp
    DATAPOOL            TYPE
    ------------------------
    dp1                 file 
    dp2                 db2
    dphdfs              hdfs
    dps3                s3
    $ 
    
#### 1.2 列出数据池详情
指定一个datapool，列出此datapool详情。

	datahub dp $DPNAME

输出

    %DPNAME 			         %DPTYPE 		%DPCONN
    {%REPOSITORY/%DATAITEM:%TAG	         %LOCAL_TIME		%T}

例子

    $ datahub dp dp1
    DATAPOOL:dp1                    file                     /var/lib/datahub/dp1
    -------------------------------------------------------------------------------------------------------
    repo1/item1:tag1                2015-10-23 03:57:42       pub		repo1_item1       tag1.txt    		Size:232.00KB
    repo1/item1:tag2                2015-10-23 03:59:49       pub		repo1_item1	  tag2
    repo1/item2:jinrong-40          2015-10-23 04:01:22       pull		item2location	  jinrong_40.txt	金融信息
    cmcc/beijing:jiangsu-lac-ci     2015-11-19 10:57:21       pull		cmcc_beijing 	  jiangsu-lac-ci.txt   位置区编码
    $ 
    
说明：cmcc_beijing为DataItem beijing在Datapool dp1中的位置， jiangsu-lac-ci.txt为Tag存储到dp1中的文件名，“位置区编码”为详细信息。
    
#### 1.3 创建数据池
新建一个datapool。

	datahub dp create $DPNAME [[file://][ABSOLUTE PATH]] | [[s3://][BUCKET]] | [[hdfs://][USERNAME:PASSWORD@HOST:PORT]]
其中：[file://][s3://][hdfs://]为datapool类型；[absolute path]为datapool保存路径（且保存类型必须设置）

输出

	%msg

例子1

在命令中，datapool类型与datapool保存路径均设置：

    $ datahub dp create testdp file:///var/lib/datahub/testdp
    DataHub : Datapool has been created successfully. Name:testdp Type:file Path:/var/lib/datahub/testdp.
    $
  
例子2

在命令中，只设置保存路径:
此时默认为file，且路径为/var/lib/datahub/testdp。

     $ datahub dp create testdp /var/lib/datahub/testdp
     DataHub : Datapool has been created successfully. Name:testdp Type:file Path:/var/lib/datahub/testdp.
     $
     
例子3

在命令中，文件类型和保存路径均不设置：
此时默认为file，且路径为/var/lib/datahub/testdp。

    $ datahub dp create testdp
    DataHub : Datapool has been created successfully. Name:testdp Type:file Path:/var/lib/datahub/testdp.
    $

例子 4

创建datapool类型[s3://]

    $ datahub dp create s3dp s3://mybucket
    DataHub : s3dp already exists, please change another name.
    $
    
说明：mybucket是s3上已存在的bucket。另外，需要在启动daemon的系统中设置环境变量：AWS_SECRET_ACCESS_KEY， AWS_ACCESS_KEY_ID， AWS_REGION。
    
例子 5

创建datapool类型[hdfs://]

    $ datahub dp create hdfsdp hdfs://user123:admin123@x.x.x.x:9000

说明：“hdfs://”后需要接hdfs的连接串。
    
#### 1.4 删除数据池
删除数据池不会删除目标数据池已保存的数据。该datapool有发布的数据项时，不能被删除。删除是在sqlite中标记状态，不真实删除。若datapool中存在已经发布的数据，则不能删除datapool。需要删除datapool下发布的数据后方可删除。。

	datahub dp rm $DPNAME

输出

	%msg

例子

    $ datahub dp rm testdp
    DataHub : Datapool testdp removed successfully!
    $
    
### 2. subs 相关命令
#### 2.1 列出所有已订阅项

	datahub subs 

输出

	REPOSITORY/DATAITEM     TYPE    STATUS
	{%REPOSITORY/%DATAITEM  %TYPE   online/offline}

例子

	$ datahub subs
	REPOSITORY/DATAITEM     TYPE    STATUS
	cmcc/beijing            file    online
	repo1/testing           hdfs    online
        $
        
#### 2.2 列出用户在某个Repository下已订阅的DataItem

	datahub subs $REPO

输出

	REPOSITORY/%DATAITEM    %TYPE        STATUS
	{%REPOSITORY/%DATAITEM  %TYPE        %STATUS}

例子

	$ datahub subs cmcc
	REPOSITORY/DATAITEM      TYPE
	cmcc/Beijing             file
	cmcc/Tianjin             file
	cmcc/Shanghai            file
	$	

    
#### 2.3 列出已订阅 DataItem 详情

	datahub subs $REPOSITORY/$DATAITEM

输出

     REPOSITORY/DATAITEM:TAG                UPDATETIME      COMMENT      STATUS
    {%REPOSITORY/%DATAITEM:%TAGNAME         %UPDATE_TIME    %COMMENT     %STATUS}
    
例子

   	$ datahub subs cmcc/beijing
        REPOSITORY/DATAITEM:TAG          UPDATETIME              COMMENT      STATUS
        cmcc/beijing:chaoyang    	 15:34 Oct 12 2015       600M         NORMAL
        cmcc/beijing:daxing      	 16:40 Oct 13 2015       435M         NORMAL
        cmcc/beijing:shunyi      	 16:40 Oct 14 2015       324M         NORMAL
	cmcc/beijing:haidian     	 16:40 Oct 15 2015       988M         NORMAL
	$
    
### 3. pull 命令
- pull命令用来下载您已订阅的DataItem下的Tag到你所指定的Datapool。

#### 3.1 拉取某个 DataItem 的 Tag。

- pull 一个 Tag ，需指定`$DATAPOOL`, 可再指定`$DATAPOOL`下的子目录`$LOCATION`，默认下载到`$DATAPOOL://$REPOSITORY_$DATAITEM`。 可选参数`[--destname, -d]`命名下载的 Tag [--automatic, -a]自动下载已订阅的Item新增的tag [--cancel, -c]取消自动下载Tag 。

		datahub pull $REPOSITORY/$DATAITEM:$TAG $DATAPOOL[://$LOCATION] [--destname，-d]

输出

	%msg

例子1
在缺省命名参数的情况下：

    $ datahub pull cmcc/beijing:chaoyang dp1://cmccbj
    DataHub : cmcc／beijing:transportation  will be pulled soon and can be found in  /var/lib/datahub/dp1/cmccbj
    $
    
例子2
在有命名参数的情况下：

    $ datahub pull cmcc/beijing:transportation dp1://cmccbj --destname “abc”
    DataHub : cmcc／beijing:transportation  will be pulled into /var/lib/datahub/dp1/cmccbj/
    $
例子3
自动（取消自动）下载：    

     $ datahub pull $REPO/$DATAITEM:$TAG $DATAPOOL[://$LOCATION] [--automatic，-a]
     $ datahub pull $REPO/$DATAITEM:$TAG $DATAPOOL[://$LOCATION] [--cancel，-c]

### 4. login 命令

- login 命令支持被动调用，用于 DataHub client 与 DataHub server 交互时作认证。并将认证信息保存到环境变量，免去后续指令重复输入认证信息。

#### 4.1 登陆到 dataos.io

	datahub login [--user=user]

输出

	%msg
    
例子

        $ datahub login
	login as: datahub
	password: *******
	Error : login failed.
        $
        
### 5. pub 相关命令
 -发布数据前需指定EntryPoint，即提供给外网访问的端口
- pub 分为发布一个 DataItem 和发布一个 Tag 。

#### 5.1 发布一个 DataItem
- 发布 DataItem 必须指定 `$DATAPOOL` 和 `$DATAPOOL` 下的子路径 `$LOCATION` , 可选参数`--accesstype`, `-t=` 指定DataItem属性：public, private, 默认private 。
- 可选参数`--comment`, `-m=` ,描述 DataItem 或者 Tag 。

		datahub pub $REPOSITORY/$DATAITEM $DATAPOOL://$LOCATION --accesstype=public [private]  [--comment, -m]

输出

	Successfully published

例子1

DataItem属性、描述均设置：

    $ datahub pub music_1/migu mydp://dirmigu --accesstype=public --comment="migu music desc"
    DataHub :Successfully published
    $
    
例子2

选择设置DataItem属性，不设置描述：

    $ datahub pub music_1/migu mydp://dirmigu --accesstype=public --comment="migu music desc"
    DataHub : Successfully published
    $
    
例子3

选择设置DataItem描述，不设置DataItem属性（此时属性默认为private）：

    $ datahub pub music_1/migu mydp://dirmigu --comment="migu music desc"
    DataHub :Successfully published
    $
    
例子4

DataItem属性、描述均不设置（此时默认属性为private）：

    $ datahub pub music_1/migu mydp://dirmigu
    DataHub : Successfully published
    $
    
说明：
1、参数支持全名和简称两种形式，例如--type等同于-t。详情见命令帮助。
2、参数赋值支持空格和等号两种形式，例如--type=file等同于--type file

#### 5.2 发布一个 Tag
- 发布 Tag 必须指定 TAGDETAIL , 用来指定 Tag 对应文件名，该文件必须存在于`$DATAPOOL://$LOCATION`内。
- 其中，参数[--comment, -m=]用于描述DataItem或者Tag

		datahub pub $REPOSITORY/$DATAITEM:$Tag $TAGDETAIL --comment=" "

输出

	successfully published

例子1

不设置Tag描述：

    $ datahub pub music_1/migu:migu_user_info migu_user_info.txt
    DataHub:successfully published
    $

例子2

设置Tag描述：

    $ datahub pub music/migu:migu_user migu_user_info.txt --comment=“a”
    DataHub:successfully published
    $

说明 ：
1、参数支持全名和简称两种形式，例如--type等同于-t。详情见命令帮助。
2、参数赋值支持空格和等号两种形式，例如--type=file等同于--type file

### 6. repo 命令

#### 6.1 查询自己创建的和具有写权限的所有 Repository

	datahub repo 

输出

    REPOSITORY
    --------------------
    Location_information                    
    Internet_stats  
    Base_station_location

#### 6.2 查询自己具有写权限的DataItem

	datahub repo Internet_stats

输出

    REPOSITORY/DATAITEM
    --------------------------------
    Internet_stats/Music
    Internet_stats/Books
    Internet_stats/Cars
    Internet_stats/Ecommerce_goods
    Internet_stats/Film_and_television

#### 6.3 查询自己创建的Tag

	datahub repo  Internet_stats/Music

输出

	REPOSITORY/DATAITEM:TAG 		   UPDATETIME  				 COMMENT
    -----------------------------------------------------------------------------------------------
    Internet_stats/Music:music_baidumusic_6008      2016-03-04 09:15:18|6天前 	        百度音乐
    Internet_stats/Music:music_qqmusic_6001         2016-02-03 09:23:30|1个月前  	QQ音乐
    Internet_stats/Music:music_kuwomusic_6005       2016-01-06 09:35:44|2个月前  	酷我音乐


#### 6.4 删除自己创建的DataItem

	datahub repo rm myrepo/myitem

输出

    Datahub : After you delete the DataItem, data could not be recovery, and all tags would be deleted either.
    Are you sure to delete the current DataItem?[Y or N]:Y
    DataHub : OK
    
说明：当此DataItem下有正在生效的订购计划时，会提示资费回退规则。

#### 6.5 删除自己创建的Tag

	datahub repo rm FavouriteMusic/MusicItem:bingyu

输出

	DataHub : After you delete the Tag, data could not be recovery.
	Are you sure to delete the current Tag?[Y or N]:y
	DataHub : OK
        
### 7. job 命令
#### 7.1 job 查看所有任务列表，包括数据下载和发送的任务

	datahub job

输出

	%msg

例子

	$ datahub job
	JOBID       STATUS       DOWN       TOTAL       PERCENT      TAG
	----------------------------------------------------------------------------------
        72cbddes    downloaded   893        893          100.0%      Base_station/Location:location
        $

#### 7.2 job 查看某个任务 id 对应的信息
	
    datahub job &JOBID
    
输出

	%msg 
	
例子

	$ datahub job
	JOBID       STATUS       DOWN       TOTAL       PERCENT     TAG
	-----------------------------------------------------------------------------
	72cbddes   downloaded    893   	    893         100.0%      Base_station/Location:location
        $
        
#### 7.3 job rm 删除某个 job
	
    datahub job rm &JOBID
    
输出

	%msg 
        
#### 8. ep 命令

- 设置DataHub daemon的Entrypoint，作为数据提供方，需要提供可访问的url，供需求方访问，并下载数据。
- 此命令也可以用来查看是否设置了Entrypoint。

#### 9. logout 命令
- logout命令支持被动调用，用于datahub client与datahub server交互时作认证。并将认证信息保存到环境变量，免去后续指令重复输入认证信息。

	datahub logout

输出

	%msg
    
例子

    $ datahub logout
    DataHub : Logout success.
    $ 
    
### 10. help 命令

- help 提供 DataHub 所有命令的帮助信息。

#### 10.1 列出帮助

	datahub help [$CMD] [$SUBCMD]

输出

    Usage of %CMD %SUBCMD
    {  %OPTION=%DEFAULT_VALUE     %OPTION_DESCRIPTION}

例子

    $datahub help dp
    Usage: 
      datahub dp [DATAPOOL]
    List all the datapools or one datapool

    Usage of datahub dp create:
      datahub dp create DATAPOOL [file://][ABSOLUTE PATH]
      e.g. datahub dp create dptest file:///home/user/test
           datahub dp create dptest /home/user/test
    Create a datapool

    Usage of datahub dp rm:
      datahub dp rm DATAPOOL
    Remove a datapool
    $



## DataHub 应用场景举例

### 1. publish 数据

- 发布数据是数据提供方行为。

在 DataHub 产品中，数据组织划分为 Repository 、DataItem 、Tag ，其中 Repository 是数据仓库， DataItem 是一个数据项，包含一个主题的数据， Tag 是一个具体的数据记录。

提供方发布数据前要在本地基于已有数据建立一个 Datapool，发布这个 Datapool 里面的数据。

假设在`/home/myusr/data/topub`目录下存在若干文件，现在想把这些文件发布为一些 tags ，等待需求方来下载。
操作过程如下：

	1) datahub dp create mydatapool file:///home/myusr/data

以上命令创建了一个Datapool，类型是file，路径是`/home/myusr/data`。

	2) datahub pub myrepo/myitem mydatapool://topub --accesstype=public --comment="my test item "

发布一个名称为 myitem 的 DataItem ，所属 Repository 是 myrepo ，对应 mydatapool 的子目录 topub ，即待发布数据存在于`/home/myusr/data/topub`中。

>>>>注意：在发布 DataItem 之前，可以在其对应的目录里创建、编译三个文件：sample.md, meta.md, price.cfg。

这三个文件的作用分别是：
* sample.md 用于保存 markdown 格式的样例数据，如果没有此文件，程序会读取此目录下的一个 tag 文件的前十行，作为样例数据，发布到 dataitem 的详情里。
* meta.md 用于保存 markdown 格式的元数据。
* price.cfg 用于保存 json 格式的资费计划，用来明确此 DataItem 的资费。格式如下：

```markdown
{
"price":[
                {
                    "times":10,
                    "money": 5,
                    "expire":30
                },
                {
                    "times": 100,
                    "money": 45,
                    "expire":30
                },
                {
                    "times":500,
                    "money": 400,
                    "expire":30
                }
        ]
}
```

	3) datahub pub myrepo/myitem:mytag test.txt

发布一名称为 mytag 的 Tag ，所属 DataItem 是 myitem ，对应数据文件是 `/home/myusr/data/topub/test.txt`。

### 2. 下载数据

- 下载数据是数据需求方的行为。

需求方用户登录 http://hub.dataos.io ，查看、搜索 Repository 、DataItem ，然后订购自己所需的 DataItem 。订购成功后，在 Tag 详情页面，点击复制，复制 Tag 全名，即可在客户端下载此 DataItem 下的 Tag 所对应的数据。

DataHub Client 操作如下：

   1）da tahub dp create mydp file:///home/usr/data/itempull 

以上命令创建了一个名为 mydp 的 Datapool ，类型是 file ，路径是`/home/myusr/data/itempull`, 用于存储即将下载 的数据。
	
    2）datahub pull repotest/itemtest:tagtest mydp://mydir1 –d tagdestname.txt

下载一个 Tag 对应的数据到 mydp 中，子路径是 mydir1。
