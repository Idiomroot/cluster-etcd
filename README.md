目录
一、etcd01节点生成etcd的证书
二、三个etcd节点安装etcd
三、etcd01拷贝证书到其他节点
四、开启集群etcd

节点信息：
etcd01   172.16.3.83
etcd02    172.16.3.90
etcd03    172.16.3.197

一、etcd01节点生成etcd的证书

[root@172-16-3-83 k8s]# git clone https://github.com/Idiomroot/cluster-etcd.git
[root@172-16-3-83 k8s]# cd cluster-etcd
[root@172-16-3-83 etcd]# sh etcd.sh
[root@172-16-3-83 etcd]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
[root@172-16-3-83 etcd]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server

二、三个etcd节点安装etcd
mkdir /data/src/ &&cd /data/src/
wget https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz
mkdir  /etc/etcd/
tar xf etcd-v3.3.10-linux-amd64.tar.gz
mv etcd-v3.3.10-linux-amd64/{etcd,etcdctl} /usr/bin/
mv etcd /etc/etcd
mv etcd.service /usr/lib/systemd/system/etcd.service
设置etcd的api版本
[root@172-16-3-83 ]# export ETCDCTL_API=2

三、etcd01拷贝证书到其他节点

[root@172-16-3-83 ]#scp /data/ssl/etcd/ca*pem root@172.16.3.90:/data/ssl/etcd/
[root@172-16-3-83 ]#scp /data/ssl/etcd/ca*pem root@172.16.3.197:/data/ssl/etcd/
[root@172-16-3-83 ]#scp /data/ssl/etcd/server*pem root@172.16.3.197:/data/ssl/etcd/
[root@172-16-3-83 ]#scp /data/ssl/etcd/server*pem root@172.16.3.90:/data/ssl/etcd/

四、开启集群etcd
systemctl enable etcd
systemctl start etcd

etcd01查看etcd是否健康

[root@172-16-3-83 ]# etcdctl --ca-file=/data/ssl/etcd/ca.pem --cert-file=/data/ssl/etcd/server.pem --key-file=/data/ssl/etcd/server-key.pem --endpoints="https://172.16.3.83:2379,https://172.16.3.90:2379,https://172.16.3.197:2379" cluster-health
member 435d25f5aa61f16b is healthy: got healthy result from https://172.16.3.90:2379
member 470dcdf2c4c2804f is healthy: got healthy result from https://172.16.3.197:2379
member aafdd75d7f990b4e is healthy: got healthy result from https://172.16.3.83:2379
cluster is healthy

注意，节点迁移的步骤

1）停止待迁移节点上的etc进程；
2）将数据目录打包复制到新的节点；
3）更新该节点对应集群中peer url，让其指向新的节点；
4）使用相同的配置，在新的节点上启动etcd进程
