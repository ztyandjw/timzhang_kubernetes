##安装前准备活动

####机器环境

* 10.0.94.164(kebe-master1)
* 10.0.94.136(kebe-node1)
* 10.0.94.234(kebe-node2)
* 10.0.94.216
* 10.0.94.147
* 10.0.94.245

####主要软件版本

* Kubernetes 1.12.2
* Docker 18.03.1-ce
* Etcd 3.3.10

####系统初始化&全局变量

######主机名
* `hostnamectl set-hostname hostname`
* 修改/etc/hosts文件

######无密码登录其它节点
* `ssh-keygen -t rsa`
* `ssh-copy-id root@${hostname}`

######将可执行文件路径添加到PATH路径下
* echo 'PATH=/opt/k8s/bin:$PATH' >> /root/.bash

######安装软件包
* `yum install -y epel-release`
* `yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp lrzsz`


######关闭防火墙
 `systemctl stop firewalld`
 
 `systemctl disable firewalld`
 
`iptables -F && sudo iptables -X && sudo iptables -F -t nat && sudo iptables -X -t nat`

 `iptables -P FORWARD ACCEPT`

######关闭swap分区
* `swapoff -a`
* `sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`

######关闭selinux
`setenforce 0`

 `grep SELINUX /etc/selinux/config`

######关闭dnsmasq
`service dnsmasq stop`

`systemctl disable dnsmasq`

######设置系统参数
```
   cat > kubernetes.conf <<EOF
   net.bridge.bridge-nf-call-iptables=1
   net.bridge.bridge-nf-call-ip6tables=1
   net.ipv4.ip_forward=1
   vm.swappiness=0
   vm.overcommit_memory=1
   vm.panic_on_oom=0
   fs.inotify.max_user_watches=89100
   EOF
   ```
   
```cp kubernetes.conf  /etc/sysctl.d/kubernetes.conf```

```sysctl -p /etc/sysctl.d/kubernetes.conf```

```mount -t cgroup -o cpu,cpuacct none /sys/fs/cgroup/cpu,cpuacct```


######加载内核模块
`modprobe br_netfilter`

 `modprobe ip_vs`
 
 ######设置系统时区
 `timedatectl set-timezone Asia/Shanghai`
 
 `timedatectl set-local-rtc 0`
 
 `systemctl restart rsyslog`
 
 `systemctl restart crond`
 
  ######创建目录
  
  `mkdir -p /opt/k8s/bin`
  
  `mkdir -p /etc/kubernetes/cert`
  
  `mkdir -p /etc/etcd/cert`
  
  `mkdir -p /var/lib/etcd`
 
##创建CA证书以及秘钥
 
####安装cfssl工具集

`mkdir -p /opt/k8s/cert && cd /opt/k8s`

`wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64`

`mv cfssl_linux-amd64 /opt/k8s/bin/cfssl`

`wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64`

`mv cfssl-certinfo_linux-amd64 /opt/k8s/bin/cfssl-certinfo`

`chmod +x /opt/k8s/bin/*`

`export PATH=/opt/k8s/bin:$PATH`


####创建CA根证书
CA 证书是集群所有节点共享的，只需要创建一个 CA 证书，后续创建的所有证书都由它签名

######创建配置文件
```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF

```

######创建证书请求文件
```
cat > ca-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "4Paradigm"
    }
  ]
}
EOF
```





   
   
 






