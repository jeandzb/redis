### 背景：  
redis 6之前没有访问控制功能，只有默认账号。即在redis.conf 配置文件中添加如下内容：  
requirepass <password>  
### 配置账号密码:  
1. redis 从6.0开始引入用户名密码认证功能，即ACL (非默认账户的密码)。  
使用acl 命令创建账号密码的相关命令：  
创建一个状态为激活，密码为 strongpassword，允许访问所有 key，允许所有命令的账号密码  
`ACL SETUSER admin on >strongpassword ~* &* +@all`  
列出所有账号  
`ACL USERS`  
查看指定账号的权限  
`ACL GETUSER username`  
删除账号  
`ACL DELUSER username`  
设置缓存最大内存  
`CONFIG SET maxmemory 2gb`  
前面执行的创建账号等命令会临时保存在内存中，重启即失效。  
需要将内存中的修改保存到文件redis.conf，需保持此文件夹可写，会在redis.conf中最后一行生成user 开头的配置。  
`CONFIG REWRITE`  
将内存中的修改保存到文件（如果配置了 aclfile）  
`ACL SAVE`  
从文件重新加载配置（手动改了 .acl 文件后使用）  
`ACL LOAD`

### 实现ACL持久化，有两种方法：  
1. 不配置aclfile  
1.1 通过acl 命令，创建账号密码  
`ACL SETUSER admin on >strongpassword ~* &* +@all`  
将账号密码保存到文件redis.conf  
`CONFIG REWRITE`
2. 配置aclfile  
在redis.conf中添加：aclfile /etc/redis/redis.acl  
执行acl相关命令后，需要执行`ACL SAVE`将内存中的修改保存到文件redis.acl  
当然，也可以直接在redis.acl文件中添加账号密码  
当修改了redis.acl文件后，需要执行`ACL RELOAD`，才会生效

注意：  
1. aclfile和requirepass是互斥的，当两个同时存在是时候，aclfile优先级高
2. 使用aclfile后，默认账号必须要设置密码（protected-mode的配置为yes会拒绝连接）或者直接禁用，在aclfile中配置`user default off`即可禁用

### 在k8s中的应用：
方案1：  
通过文件的方式将aclfile挂载到pod中，并修改redis.conf，将aclfile指向挂载的目录。保持文件可写  

方案2：  
通过configmap或者secret的方式将aclfile挂载到pod中，并修改redis.conf，将aclfile指向挂载的目录。文件只读  
不允许在业务中通过命令修改aclfile  

### 拓展：
1. protected-mode  
默认值：yes，当 Redis 认为自己处于不安全状态时，protected-mode 会触发保护机制，直接切断所有来自外部 IP 的连接。即使你配置了 bind 0.0.0.0  
触发逻辑：
1). 没有通过 requirepass 设置全局密码。（注意：这里只认 requirepass，不认 aclfile 中的用户密码）
2). 没有显式绑定具体的 IP 地址（比如只绑定了 bind 0.0.0.0 这种模糊的地址，或者根本没配置 bind）。
