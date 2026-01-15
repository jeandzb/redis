1, 镜像必须使用quay.io/opstree仓库中的，否则会报错
2, 默认的redis.conf会自动生成aclfile的配置：
aclfile /etc/redis/user.acl ，所以“user.acl”文件名是固定的。
在使用configmap和secret是key的名称必须是：user.acl

## 连接方式
1, 集群模式下，客户端连接redis集群类似与直连服务端，所以集群外部访问redis,有2种方式：
方案1, 使用云厂商的loadbalancer，或者在私网环境搭建metalb，模拟loadbalancer
方案2, 使用Predixy服务，做代理，但是此代理并非Nginx



## 依赖说明
redis-cluster需要依赖redis-operator，operator管理集群的相关资源


