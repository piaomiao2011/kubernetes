一、机器规划  

使用五台机子部署k8s集群，规划如下：  
master节点3台（同时也是etcd节点）  
node节点2台  
ip分配如下：  
ip:192.168.10.101，主机名：k8s-etcd01  
ip:192.168.10.102，主机名：k8s-etcd02  
ip:192.168.10.103，主机名：k8s-etcd03  
ip:192.168.10.104，主机名：k8s-node01  
ip:192.168.10.105，主机名：k8s-node02   
注意：  
1、全部机子关闭防火墙、关闭selinux  
2、全部机子时间要同步  
3、全部机子的hosts文件添加以下内容：  
192.168.10.101 k8s-master01   k8s-master01.logmm.com  k8s-etcd01.logmm.com  k8s-etcd01  myk8s-api.logmm.com  
192.168.10.102 k8s-master02   k8s-master02.logmm.com  k8s-etcd02.logmm.com  k8s-etcd02  
192.168.10.103 k8s-master03   k8s-master03.logmm.com  k8s-etcd03.logmm.com  k8s-etcd03  
192.168.10.104 k8s-node01     k8s-node01.logmm.com  
192.168.10.105 k8s-node02     k8s-node02.logmm.com  
二、etcd集群  
1、etcd01、02、02、03三个节点都安装etcd  

为了方便，已经在etcd01节点上配置好ansible了，使用andible批量安装。如果没有ansible，则需在3个etcd节点上手动执行yum install etcd -y命令安装etcd。  
```
[root@k8s-etcd01 ~]# ansible etcd -m yum -a "name=etcd state=present"
```  
2、制作etcd、k8s证书  

 在etcd01节点上安装git，使用git克隆证书制作的文件使用git克隆证书制作的文件  
```
[root@k8s-etcd01 ~]# yum install git -y
[root@k8s-etcd01 ~]# git clone https://github.com/iKubernetes/k8s-certs-generator
```  
（1）生成etcd CA及相关组件的证书及私钥  
生成证书的过程中要输入域名，这里使用logmm.com  
```
[root@k8s-etcd01 ~]# cd k8s-certs-generator/
[root@k8s-etcd01 k8s-certs-generator]# bash gencerts.sh etcd
Enter Domain Name [ilinux.io]: logmm.com
Generating RSA private key, 4096 bit long modulus
..................................++
.........++
e is 65537 (0x10001)
Generating RSA private key, 2048 bit long modulus
........................+++
...........................................+++
e is 65537 (0x10001)
Generating etcd/pki/peer.csr
Generating RSA private key, 2048 bit long modulus
.....................+++
.....................+++
e is 65537 (0x10001)
Generating etcd/pki/server.csr
Generating RSA private key, 2048 bit long modulus
.......+++
...........................+++
e is 65537 (0x10001)
Generating etcd/pki/apiserver-etcd-client.csr
Generating RSA private key, 2048 bit long modulus
.........+++
.....................................................+++
e is 65537 (0x10001)
Generating etcd/pki/client.csr
Generating etcd/pki/peer.crt
Using configuration from openssl.conf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4096 (0x1000)
        Validity
            Not Before: Dec 14 16:21:49 2018 GMT
            Not After : Dec 11 16:21:49 2028 GMT
        Subject:
            commonName                = etcd
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Cert Type: 
                SSL Server
            Netscape Comment: 
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier: 
                82:E1:12:62:F7:7B:C2:09:C9:3F:99:7C:57:18:37:DB:82:E6:95:DB
            X509v3 Authority Key Identifier: 
                keyid:6E:1E:B6:94:16:EC:F0:66:3C:21:DD:C4:07:FB:94:9D:8A:8D:A8:65
                DirName:/CN=etcd-ca
                serial:D3:79:41:93:D6:FF:D8:CD

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Subject Alternative Name: 
                DNS:*.logmm.com
Certificate is to be certified until Dec 11 16:21:49 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
Generating etcd/pki/server.crt
Using configuration from openssl.conf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4097 (0x1001)
        Validity
            Not Before: Dec 14 16:21:50 2018 GMT
            Not After : Dec 11 16:21:50 2028 GMT
        Subject:
            commonName                = etcd
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Cert Type: 
                SSL Server
            Netscape Comment: 
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier: 
                B7:54:A2:7B:4C:10:49:8B:8F:44:B7:26:B1:2E:FC:69:53:4E:38:76
            X509v3 Authority Key Identifier: 
                keyid:6E:1E:B6:94:16:EC:F0:66:3C:21:DD:C4:07:FB:94:9D:8A:8D:A8:65
                DirName:/CN=etcd-ca
                serial:D3:79:41:93:D6:FF:D8:CD

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Subject Alternative Name: 
                DNS:*.logmm.com
Certificate is to be certified until Dec 11 16:21:50 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
Generating etcd/pki/apiserver-etcd-client.crt
Using configuration from openssl.conf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4098 (0x1002)
        Validity
            Not Before: Dec 14 16:21:50 2018 GMT
            Not After : Dec 11 16:21:50 2028 GMT
        Subject:
            commonName                = etcd
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Cert Type: 
                SSL Client
            Netscape Comment: 
                OpenSSL Generated Client Certificate
            X509v3 Subject Key Identifier: 
                F6:02:2A:00:C6:EA:92:F3:49:B5:A3:A7:34:3E:F2:E8:77:8C:01:A7
            X509v3 Authority Key Identifier: 
                keyid:6E:1E:B6:94:16:EC:F0:66:3C:21:DD:C4:07:FB:94:9D:8A:8D:A8:65

            X509v3 Key Usage: critical
                Digital Signature, Non Repudiation, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
Certificate is to be certified until Dec 11 16:21:50 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
Generating etcd/pki/client.crt
Using configuration from openssl.conf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4099 (0x1003)
        Validity
            Not Before: Dec 14 16:21:50 2018 GMT
            Not After : Dec 11 16:21:50 2028 GMT
        Subject:
            commonName                = etcd
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Cert Type: 
                SSL Client
            Netscape Comment: 
                OpenSSL Generated Client Certificate
            X509v3 Subject Key Identifier: 
                65:48:18:41:64:04:42:71:B8:4F:A0:06:4A:5D:53:23:55:7D:27:D6
            X509v3 Authority Key Identifier: 
                keyid:6E:1E:B6:94:16:EC:F0:66:3C:21:DD:C4:07:FB:94:9D:8A:8D:A8:65

            X509v3 Key Usage: critical
                Digital Signature, Non Repudiation, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
Certificate is to be certified until Dec 11 16:21:50 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
[root@k8s-etcd01 k8s-certs-generator]# 
```  
（2）生成kubernetes CA及相关组件的证书及私钥  
域名为：logmm.com  
集群名：myk8s  
master节点名称：k8s-master01、k8s-master02、k8s-master03  
```
[root@k8s-etcd01 k8s-certs-generator]# bash gencerts.sh k8s
Enter Domain Name [ilinux.io]: logmm.com
Enter Kubernetes Cluster Name [kubernetes]: myk8s
Enter the IP Address in default namespace 
  of the Kubernetes API Server[10.96.0.1]: 
Enter Master servers name[master01 master02 master03]: k8s-master01  k8s-master02 k8s-master03
Generating CA key and self signed cert.
Generating RSA private key, 4096 bit long modulus
。。。
```  
生成的证书如下：  

etcd相关证书：  
```
[root@k8s-etcd01 k8s-certs-generator]# tree etcd
etcd
├── patches
│   └── etcd-client-cert.patch
└── pki
    ├── apiserver-etcd-client.crt
    ├── apiserver-etcd-client.key
    ├── ca.crt
    ├── ca.key
    ├── client.crt
    ├── client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key

2 directories, 11 files
[root@k8s-etcd01 k8s-certs-generator]#
```  
k8s证书：  
```
[root@k8s-etcd01 k8s-certs-generator]# tree kubernetes/
kubernetes/
├── CA
│   ├── ca.crt
│   └── ca.key
├── front-proxy
│   ├── front-proxy-ca.crt
│   ├── front-proxy-ca.key
│   ├── front-proxy-client.crt
│   └── front-proxy-client.key
├── ingress
│   ├── ingress-server.crt
│   ├── ingress-server.key
│   └── patches
│       └── ingress-tls.patch
├── k8s-master01
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
├── k8s-master02
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
├── k8s-master03
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
└── kubelet
    ├── auth
    │   ├── bootstrap.conf
    │   └── kube-proxy.conf
    └── pki
        ├── ca.crt
        ├── kube-proxy.crt
        └── kube-proxy.key

16 directories, 80 files
[root@k8s-etcd01 k8s-certs-generator]#
```  
将生成的etcd、k8s证书复制到/etc/etcd/pki目录中  
```
[root@k8s-etcd01 k8s-certs-generator]# cp -rp etcd /etc/etcd/pki
[root@k8s-etcd01 k8s-certs-generator]# mv /etc/etcd/pki/pki/* /etc/etcd/pki/
[root@k8s-etcd01 k8s-certs-generator]# rm -rf /etc/etcd/pki/pki/
```  
同时把/etc/etcd/pki复制到etcd02、etcd03节点的/etc/etcd目录中：  
```
[root@k8s-etcd01 k8s-certs-generator]# scp -rp /etc/etcd/pki/ 192.168.10.102:/etc/etcd/   
[root@k8s-etcd01 k8s-certs-generator]# scp -rp /etc/etcd/pki/ 192.168.10.103:/etc/etcd/    
[root@k8s-etcd01 k8s-certs-generator]# 
```  
3、etcd集群配置模版  

在etcd01节点上git克隆：https://github.com/iKubernetes/k8s-bin-inst  
```
[root@k8s-etcd01 ~]# git clone https://github.com/iKubernetes/k8s-bin-inst
```  
修改etcd.conf文件  
```
[root@k8s-etcd01 ~]# cd k8s-bin-inst/
[root@k8s-etcd01 k8s-bin-inst]#vim etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/k8s.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="https://192.168.10.101:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.101:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="k8s-etcd01.logmm.com"
ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://k8s-etcd01.logmm.com:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://k8s-etcd01.logmm.com:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="k8s-etcd01.logmm.com=https://k8s-etcd01.logmm.com:2380,k8s-etcd02.logmm.com=https://k8s-etcd02.logmm.com:2380,k8s-etcd03.logmm.com=https://k8s-etcd03.logmm.com:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
ETCD_CERT_FILE="/etc/etcd/pki/server.crt"
ETCD_KEY_FILE="/etc/etcd/pki/server.key"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_TRUSTED_CA_FILE="/etc/etcd/pki/ca.crt"
ETCD_AUTO_TLS="false"
ETCD_PEER_CERT_FILE="/etc/etcd/pki/peer.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/pki/peer.key"
ETCD_PEER_CLIENT_CERT_AUTH="true"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/pki/ca.crt"
ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
```  
将此文件复制到/etc/etcd/目录中，同时复制到etcd02、etcd03节点的/etc/etcd/  
```
[root@k8s-etcd01 k8s-bin-inst]# cp etcd/etcd.conf /etc/etcd/
[root@k8s-etcd01 k8s-bin-inst]# scp etcd/etcd.conf 192.168.10.102:/etc/etcd/
etcd.conf                                                                    100% 1982   273.3KB/s   00:00    
[root@k8s-etcd01 k8s-bin-inst]# scp etcd/etcd.conf 192.168.10.103:/etc/etcd/
etcd.conf                                                                    100% 1982    75.8KB/s   00:00    
[root@k8s-etcd01 k8s-bin-inst]# 
```  
etcd02节点修改etcd.conf文件：  
```
[root@k8s-etcd02 ~]# vim /etc/etcd/etcd.conf 
ETCD_LISTEN_PEER_URLS="https://192.168.10.102:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.102:2379"
ETCD_NAME="k8s-etcd02.logmm.com"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://k8s-etcd02.logmm.com:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://k8s-etcd02.logmm.com:2379"
```  
只修改上面这几项，其他的不变。  

etcd03节点修改etcd.conf文件：  
```
[root@k8s-etcd03 ~]# vim /etc/etcd/etcd.conf 
ETCD_LISTEN_PEER_URLS="https://192.168.10.103:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.103:2379"
ETCD_NAME="k8s-etcd03.logmm.com"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://k8s-etcd03.logmm.com:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://k8s-etcd03.logmm.com:2379"
```  
只修改上面这几项，其他的不变。  
4、启动etcd服务  

分别启动etcd01、02、03节点的etcd服务  
```
[root@k8s-etcd01 ~]# systemctl start etcd
[root@k8s-etcd02 ~]# systemctl start etcd
[root@k8s-etcd03 ~]# systemctl start etcd
[root@k8s-etcd01 ~]# systemctl enable etcd
[root@k8s-etcd02 ~]# systemctl enable etcd
[root@k8s-etcd03 ~]# systemctl enable etcd
```  
查看一下集群是否健康：  
```
[root@k8s-etcd01 ~]# etcdctl --key-file=/etc/etcd/pki/client.key --cert-file=/etc/etcd/pki/client.crt --ca-file=/etc/etcd/pki/ca.crt --endpoints="https://k8s-etcd01.logmm.com:2379" cluster-health
member 97f4299c17715a9 is healthy: got healthy result from https://k8s-etcd03.logmm.com:2379
member 3c596c9b8fc553d8 is healthy: got healthy result from https://k8s-etcd02.logmm.com:2379
member e65386fe0aaf90fc is healthy: got healthy result from https://k8s-etcd01.logmm.com:2379
cluster is healthy
[root@k8s-etcd01 ~]# 
```  
OK，集群处于健康状态。  
三、kubernetes集群  
1、下载k8s二进制文件  

kubernetes托管在：https://github.com/kubernetes  

相关版本：https://github.com/kubernetes/kubernetes/releases  

这里使用最新版本：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#downloads-for-v1131  

etcd01节点下载1.13.1版本的服务端二进制文件：  
```
[root@k8s-etcd01 ~]# curl -O https://dl.k8s.io/v1.13.1/kubernetes-server-linux-amd64.tar.gz
```  
解压到/usr/local/目录中  
```
[root@k8s-etcd01 ~]# tar xf kubernetes-server-linux-amd64.tar.gz -C /usr/local/
```  
配置环境变量：  
```
[root@k8s-etcd01 ~]# vim /etc/profile.d/k8s.sh
export PATH=$PATH:/usr/local/kubernetes/server/bin
[root@k8s-etcd01 ~]# source /etc/profile.d/k8s.sh
[root@k8s-etcd01 ~]#
```  
2、k8s证书  

在前面中，k8s证书已经生成好。  
```
[root@k8s-etcd01 k8s-certs-generator]# pwd
/root/k8s-certs-generator
[root@k8s-etcd01 k8s-certs-generator]# tree kubernetes/
kubernetes/
├── CA
│   ├── ca.crt
│   └── ca.key
├── front-proxy
│   ├── front-proxy-ca.crt
│   ├── front-proxy-ca.key
│   ├── front-proxy-client.crt
│   └── front-proxy-client.key
├── ingress
│   ├── ingress-server.crt
│   ├── ingress-server.key
│   └── patches
│       └── ingress-tls.patch
├── k8s-master01
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
├── k8s-master02
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
├── k8s-master03
│   ├── auth
│   │   ├── admin.conf
│   │   ├── controller-manager.conf
│   │   └── scheduler.conf
│   ├── pki
│   │   ├── apiserver.crt
│   │   ├── apiserver-etcd-client.crt
│   │   ├── apiserver-etcd-client.key
│   │   ├── apiserver.key
│   │   ├── apiserver-kubelet-client.crt
│   │   ├── apiserver-kubelet-client.key
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── front-proxy-ca.crt
│   │   ├── front-proxy-ca.key
│   │   ├── front-proxy-client.crt
│   │   ├── front-proxy-client.key
│   │   ├── kube-controller-manager.crt
│   │   ├── kube-controller-manager.key
│   │   ├── kube-scheduler.crt
│   │   ├── kube-scheduler.key
│   │   ├── sa.key
│   │   └── sa.pub
│   └── token.csv
└── kubelet
    ├── auth
    │   ├── bootstrap.conf
    │   └── kube-proxy.conf
    └── pki
        ├── ca.crt
        ├── kube-proxy.crt
        └── kube-proxy.key

16 directories, 80 files
[root@k8s-etcd01 k8s-certs-generator]#
[root@k8s-etcd01 k8s-certs-generator]# cd kubernetes/
[root@k8s-etcd01 kubernetes]# ls
CA  front-proxy  ingress  k8s-master01  k8s-master02  k8s-master03  kubelet
[root@k8s-etcd01 kubernetes]# 
```  
将master01的此证书复制到/etc/kubernetes/pki目录中：  
```
[root@k8s-etcd01 ~]# mkdir /etc/kubernetes
[root@k8s-etcd01 ~]# cp -rp k8s-certs-generator/kubernetes/k8s-master01/* /etc/kubernetes/
```  
3、模版文件  

前面中已经git clone模版文件了：https://github.com/iKubernetes/k8s-bin-inst  

1、复制k8s-bin-inst/master/etc/kubernetes/目录中的所有文件到/etc/kubernetes/  
```
[root@k8s-etcd01 ~]# cp k8s-bin-inst/master/etc/kubernetes/* /etc/kubernetes/
[root@k8s-etcd01 ~]# 
```  
/etc/kubernetes/apiserver文件内容：  
```
[root@k8s-etcd01 ~]# cat /etc/kubernetes/apiserver 
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--advertise-address=0.0.0.0"

# The port on the local server to listen on.
KUBE_API_PORT="--secure-port=6443 --insecure-port=0"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=https://k8s-etcd01.logmm.com:2379,https://k8s-etcd02.logmm.com:2379,https://k8s-etcd03.logmm.com:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.96.0.0/12"

# default admission control policies
KUBE_ADMISSION_CONTROL="--enable-admission-plugins=NodeRestriction"

# Add your own!
KUBE_API_ARGS="--authorization-mode=Node,RBAC \
    --client-ca-file=/etc/kubernetes/pki/ca.crt \
    --enable-bootstrap-token-auth=true \
    --etcd-cafile=/etc/etcd/pki/ca.crt \
    --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt \
    --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key \
    --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt \
    --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key \
    --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \
    --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt \
    --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key \
    --requestheader-allowed-names=front-proxy-client \
    --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \
    --requestheader-extra-headers-prefix=X-Remote-Extra- \
    --requestheader-group-headers=X-Remote-Group \
    --requestheader-username-headers=X-Remote-User\
    --service-account-key-file=/etc/kubernetes/pki/sa.pub \
    --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
    --tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
    --token-auth-file=/etc/kubernetes/token.csv"
[root@k8s-etcd01 ~]# 
```  
2、复制k8s-bin-inst/master/unit-files/kube-*文件到 /usr/lib/systemd/system/目录中  
```
[root@k8s-etcd01 ~]# cp k8s-bin-inst/master/unit-files/kube-* /usr/lib/systemd/system/
[root@k8s-etcd01 ~]# 
```  
执行：  
```
[root@k8s-etcd01 ~]# systemctl daemon-reload 
```  
4、创建k8s运行的用户和工作目录  

用户为：kube  

工作目录：/var/run/kubernets  
```
[root@k8s-etcd01 ~]# useradd -r kube
[root@k8s-etcd01 ~]# mkdir /var/run/kubernetes
[root@k8s-etcd01 ~]# chown kube.kube /var/run/kubernetes
[root@k8s-etcd01 ~]# 
```  
5、启动apiserver服务  
```
[root@k8s-etcd01 ~]# systemctl daemon-reload 
[root@k8s-etcd01 ~]# systemctl start kube-apiserver.service
[root@k8s-etcd01 ~]# systemctl enable kube-apiserver.service 
```  
6、配置kubectl  

这个跟使用kubeadm初始化集群的最后的操作一样  
```
[root@k8s-etcd01 ~]# mkdir $HOME/.kube
[root@k8s-etcd01 ~]# cp /etc/kubernetes/auth/admin.conf $HOME/.kube/config
[root@k8s-etcd01 ~]#
```  
7、创建ClusterRoleBinding，授予用户相应操作所需要的权限  

命令：kubectl create clusterrolebinding system:bootstrapper --group=system:bootstrappers --clusterrole=system:bootstrapper  

或者： kubectl create clusterrolebinding system:bootstrapper --user=system:bootstrapper --clusterrole=system:node-bootstrapper  
```
[root@k8s-etcd01 ~]# kubectl create clusterrolebinding system:bootstrapper --user=system:bootstrapper --clusterrole=system:node-bootstrapper
clusterrolebinding.rbac.authorization.k8s.io/system:bootstrapper created
[root@k8s-etcd01 ~]# 
```  
8、启动kube-controller-manager服务    

controller-manager配置文件：  
```
[root@k8s-etcd01 ~]# cat /etc/kubernetes/controller-manager 
###
# The following values are used to configure the kubernetes controller-manager

# defaults from config and apiserver should be adequate

# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS="--bind-address=127.0.0.1 \
    --allocate-node-cidrs=true \
    --authentication-kubeconfig=/etc/kubernetes/auth/controller-manager.conf \
    --authorization-kubeconfig=/etc/kubernetes/auth/controller-manager.conf \
    --client-ca-file=/etc/kubernetes/pki/ca.crt \
    --cluster-cidr=10.244.0.0/16 \
    --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
    --cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
    --controllers=*,bootstrapsigner,tokencleaner \
    --kubeconfig=/etc/kubernetes/auth/controller-manager.conf \
    --leader-elect=true \
    --node-cidr-mask-size=24 \
    --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \
    --root-ca-file=/etc/kubernetes/pki/ca.crt \
    --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
    --use-service-account-credentials=true"
[root@k8s-etcd01 ~]# 
```  
启动服务：  
```
[root@k8s-etcd01 ~]# systemctl start kube-controller-manager.service
[root@k8s-etcd01 ~]# systemctl enable kube-controller-manager.service 
```  
9、schedule服务  

配置文件：  
```
[root@k8s-etcd01 ~]# cat /etc/kubernetes/scheduler 
###
# kubernetes scheduler config

# default config should be adequate

# Add your own!
KUBE_SCHEDULER_ARGS="--address=127.0.0.1 \
    --kubeconfig=/etc/kubernetes/auth/scheduler.conf \
    --leader-elect=true"
[root@k8s-etcd01 ~]#
```  
启动服务：  
```
[root@k8s-etcd01 ~]# systemctl start kube-scheduler.service
[root@k8s-etcd01 ~]# systemctl enable kube-scheduler.service
[root@k8s-etcd01 ~]# systemctl status kube-scheduler.service
● kube-scheduler.service - Kubernetes Scheduler Plugin
   Loaded: loaded (/usr/lib/systemd/system/kube-scheduler.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2018-12-15 13:45:06 CST; 48s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 3232 (kube-scheduler)
   CGroup: /system.slice/kube-scheduler.service
           └─3232 /usr/local/kubernetes/server/bin/kube-scheduler --logtostderr=true --v=0 --address=127.0.0.1 --kubeconfig=/etc/kubernetes/auth/scheduler.conf --leader-...

Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:08.700285    3232 server.go:150] Version: v1.13.1
Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:08.700377    3232 defaults.go:210] TaintNodesByCondition is enabled, PodToleratesNodeTaints predic... mandatory
Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: W1215 13:45:08.704996    3232 authorization.go:47] Authorization is disabled
Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: W1215 13:45:08.705017    3232 authentication.go:55] Authentication is disabled
Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:08.705030    3232 deprecated_insecure_serving.go:49] Serving healthz insecurely on 127.0.0.1:10251
Dec 15 13:45:08 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:08.705571    3232 secure_serving.go:116] Serving securely on [::]:10259
Dec 15 13:45:09 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:09.709171    3232 controller_utils.go:1027] Waiting for caches to sync for scheduler controller
Dec 15 13:45:09 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:09.809275    3232 controller_utils.go:1034] Caches are synced for scheduler controller
Dec 15 13:45:09 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:09.809335    3232 leaderelection.go:205] attempting to acquire leader lease  kube-system/kube-scheduler...
Dec 15 13:45:09 k8s-etcd01 kube-scheduler[3232]: I1215 13:45:09.846569    3232 leaderelection.go:214] successfully acquired lease kube-system/kube-scheduler
Hint: Some lines were ellipsized, use -l to show in full.
[root@k8s-etcd01 ~]#
```  
至此，第一个master节点配置成功。  

使用同样的方法部署master02、master03节点。  

但要注意的是，配置文件要做相应修改。  
四、node节点配置  
1、安装docker  

这里安装18.06.1版本  
```
[root@k8s-node01 ~]#  curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@k8s-node01 ~]# yum list docker-ce --showduplicates | sort -r
[root@k8s-node01 ~]# yum install docker-ce-18.06.1.ce-3.el7 -y
```  
2、kubelet配置  

（1）在master01节点上已经下载好了node节点的kubelet证书相关文件，复制到node节点的/etc/kubernetes/目录中  
```
[root@k8s-etcd01 ~]# scp -rp k8s-certs-generator/kubernetes/kubelet/* 192.168.10.104:/etc/kubernetes/
[root@k8s-etcd01 ~]# scp -rp k8s-certs-generator/kubernetes/kubelet/* 192.168.10.105:/etc/kubernetes/
```  
（2）把配置文件复制到node节点：  
```
[root@k8s-etcd01 ~]# scp k8s-bin-inst/nodes/etc/kubernetes/* 192.168.10.104:/etc/kubernetes/
[root@k8s-etcd01 ~]# scp k8s-bin-inst/nodes/etc/kubernetes/* 192.168.10.105:/etc/kubernetes/
[root@k8s-etcd01 ~]# scp k8s-bin-inst/nodes/unit-files/*  192.168.10.104:/usr/lib/systemd/system
[root@k8s-etcd01 ~]# scp k8s-bin-inst/nodes/unit-files/*  192.168.10.105:/usr/lib/systemd/system
[root@k8s-node01 ~]# systemctl daemon-reload
[root@k8s-node02 ~]# systemctl daemon-reload
```  
（3）复制k8s-bin-inst/nodes/var/lib/文件到node节点/var/lib/目录中  
```
[root@k8s-etcd01 ~]# scp -rp k8s-bin-inst/nodes/var/lib/kube* 192.168.10.104:/var/lib/
[root@k8s-etcd01 ~]# scp -rp k8s-bin-inst/nodes/var/lib/kube* 192.168.10.105:/var/lib/
[root@k8s-etcd01 ~]# 
```  
kubelet配置文件：  
```
[root@k8s-node01 ~]# cat /etc/kubernetes/kubelet 
###
# kubernetes kubelet config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# Add your own!
KUBELET_ARGS="--network-plugin=cni \
    --config=/var/lib/kubelet/config.yaml \
    --kubeconfig=/etc/kubernetes/auth/kubelet.conf \
    --bootstrap-kubeconfig=/etc/kubernetes/auth/bootstrap.conf"
[root@k8s-node01 ~]# 
```  
/var/lib/kubelet/config.yaml文件：  
```
[root@k8s-node01 ~]# cat  /var/lib/kubelet/config.yaml 
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: cgroupfs
cgroupsPerQOS: true
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
configMapAndSecretChangeDetectionStrategy: Watch
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuCFSQuotaPeriod: 100ms
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: false
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kind: KubeletConfiguration
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeLeaseDurationSeconds: 40
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
port: 10250
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
[root@k8s-node01 ~]#
```  
（4）创建/etc/kubernetes/manitests目录  
```
[root@k8s-node01 ~]# mkdir /etc/kubernetes/manitests
```  
（5）CNI插件  

下载地址：https://github.com/containernetworking/plugins/releases  
```
[root@k8s-node01 ~]# wget https://github.com/containernetworking/plugins/releases/download/v0.7.4/cni-plugins-amd64-v0.7.4.tgz
```  
解压：要解压到/opt/cni/bin/目录  
```
[root@k8s-node01 ~]# mkdir /opt/cni/bin -p
[root@k8s-node01 ~]# tar -xf cni-plugins-amd64-v0.7.4.tgz -C /opt/cni/bin/
```  
（6）下载k8s的node端文件  

下载地址：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#downloads-for-v1131  
```
[root@k8s-node01 ~]# wget https://dl.k8s.io/v1.13.1/kubernetes-node-linux-amd64.tar.gz
```  
解压到/usr/local/目录中  
```
[root@k8s-node01 ~]# tar xf kubernetes-node-linux-amd64.tar.gz -C /usr/local/
[root@k8s-node01 ~]#
```  
（7）启动kubelet服务  
```
[root@k8s-node01 ~]# systemctl restart  kubelet.service
```  
（8）证书签证  

在master01节点上对node节点进行签证。  

查看一下node节点的csr请求：  
```
[root@k8s-etcd01 ~]# kubectl get csr
NAME                                                   AGE     REQUESTOR             CONDITION
node-csr-ALHXXR_zVFd0VD5Cei3DvkwyNE-H6NrwKQjlWFqgg60   4m28s   system:bootstrapper   Pending
node-csr-qf7FjC2aebcAhT0dbv7psO7kKDnbUAb08TBhmzTtWlc   8m      system:bootstrapper   Pending
[root@k8s-etcd01 ~]# 
```  
签证：  
```
[root@k8s-etcd01 ~]# kubectl certificate approve node-csr-ALHXXR_zVFd0VD5Cei3DvkwyNE-H6NrwKQjlWFqgg60
certificatesigningrequest.certificates.k8s.io/node-csr-ALHXXR_zVFd0VD5Cei3DvkwyNE-H6NrwKQjlWFqgg60 approved
[root@k8s-etcd01 ~]# kubectl certificate approve node-csr-qf7FjC2aebcAhT0dbv7psO7kKDnbUAb08TBhmzTtWlc
certificatesigningrequest.certificates.k8s.io/node-csr-qf7FjC2aebcAhT0dbv7psO7kKDnbUAb08TBhmzTtWlc approved
[root@k8s-etcd01 ~]#
```  
查看一下：  
```
[root@k8s-etcd01 ~]# kubectl get csr
NAME                                                   AGE     REQUESTOR             CONDITION
node-csr-ALHXXR_zVFd0VD5Cei3DvkwyNE-H6NrwKQjlWFqgg60   8m14s   system:bootstrapper   Approved,Issued
node-csr-qf7FjC2aebcAhT0dbv7psO7kKDnbUAb08TBhmzTtWlc   11m     system:bootstrapper   Approved,Issued
[root@k8s-etcd01 ~]# 
```  
状态为：Approved，证书已颁发  

此时查看集群的节点：  
```
[root@k8s-etcd01 ~]# kubectl get nodes
NAME         STATUS     ROLES    AGE     VERSION
k8s-node01   NotReady   <none>   2m9s    v1.13.1
k8s-node02   NotReady   <none>   2m16s   v1.13.1
[root@k8s-etcd01 ~]#
```  
此时node01、node02已经添加进集群了，但是状态是NotReady。  

（9）ipvs模块加载  

创建/etc/sysconfig/modules/ipvs.modules文件  
```
[root@k8s-node01 ~]# vim /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
for i in $(ls $ipvs_mods_dir | grep -o "^[^.]*");do
   /sbin/modinfo -F filename $i &> /dev/null
  if [ $? -eq 0 ];then
    /sbin/modprobe $i
  fi
done
[root@k8s-node01 ~]# sh +x /etc/sysconfig/modules/ipvs.modules
[root@k8s-node01 ~]# bash /etc/sysconfig/modules/ipvs.modules
```  
查看一下ipvs模块  
```
[root@k8s-node01 ~]# lsmod | grep ip_vs
ip_vs_wlc              12519  0 
ip_vs_sed              12519  0 
ip_vs_pe_sip           12740  0 
nf_conntrack_sip       33860  1 ip_vs_pe_sip
ip_vs_nq               12516  0 
ip_vs_lc               12516  0 
ip_vs_lblcr            12922  0 
ip_vs_lblc             12819  0 
ip_vs_ftp              13079  0 
ip_vs_dh               12688  0 
ip_vs_sh               12688  0 
ip_vs_wrr              12697  0 
ip_vs_rr               12600  1 
ip_vs                 141473  25 ip_vs_dh,ip_vs_lc,ip_vs_nq,ip_vs_rr,ip_vs_sh,ip_vs_ftp,ip_vs_sed,ip_vs_wlc,ip_vs_wrr,ip_vs_pe_sip,ip_vs_lblcr,ip_vs_lblc
nf_nat                 26787  3 ip_vs_ftp,nf_nat_ipv4,nf_nat_masquerade_ipv4
nf_conntrack          133053  8 ip_vs,nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_sip,nf_conntrack_ipv4
libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
[root@k8s-node01 ~]# 
```  
模块加载成功。  

node节点安装ipvsadm：  
```
[root@k8s-node01 ~]# yum install ipvsadm -y
[root@k8s-node02 ~]# yum install ipvsadm -y
```  
（10）启动kube-proxy服务  
```
[root@k8s-node01 ~]# systemctl start kube-proxy
```  
（11）安装flannel插件  

地址：https://github.com/coreos/flannel  

在master01上执行命令：  
```
[root@k8s-etcd01 ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```  
（12）查看集群的node节点  
```
[root@k8s-etcd01 ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-node01   Ready    <none>   18m   v1.13.1
k8s-node02   Ready    <none>   18m   v1.13.1
[root@k8s-etcd01 ~]# 
[root@k8s-etcd01 ~]# kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-n5pnn   1/1     Running   0          3m19s
kube-flannel-ds-amd64-nwd6p   1/1     Running   0          3m19s
[root@k8s-etcd01 ~]#
```  
OK，两个节点都是Ready状态。  

至此node节点部署成功。  
五、coredns配置  

1、在msater01（etcd01)节点部署coredns  

以pod的形式  
```
[root@k8s-etcd01 ~]# mkdir coredns && cd coredns
[root@k8s-etcd01 coredns]# wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
[root@k8s-etcd01 coredns]# wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/deploy.sh
[root@k8s-etcd01 coredns]# bash deploy.sh -i 10.96.0.10 -r "10.96.0.0/12" -s -t coredns.yaml.sed | kubectl apply -f -
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.extensions/coredns created
service/kube-dns created
[root@k8s-etcd01 coredns]# 
```  
查看一下：  
```
[root@k8s-etcd01 coredns]# kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
coredns-7bb49b45c8-cxlt8      1/1     Running   0          60s
coredns-7bb49b45c8-ffr4t      1/1     Running   0          60s
kube-flannel-ds-amd64-n5pnn   1/1     Running   0          15m
kube-flannel-ds-amd64-nwd6p   1/1     Running   0          15m
[root@k8s-etcd01 coredns]#
```  
coredns的pod创建成功。  

2、node节点的ipvs规则如下：  
```
[root@k8s-node01 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.10.101:6443          Masq    1      2          0         
TCP  10.96.0.10:53 rr
  -> 10.244.0.2:53                Masq    1      0          0         
  -> 10.244.2.2:53                Masq    1      0          0         
TCP  10.96.0.10:9153 rr
  -> 10.244.0.2:9153              Masq    1      0          0         
  -> 10.244.2.2:9153              Masq    1      0          0         
UDP  10.96.0.10:53 rr
  -> 10.244.0.2:53                Masq    1      0          0         
  -> 10.244.2.2:53                Masq    1      0          0         
[root@k8s-node01 ~]# 
```  
