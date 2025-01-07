*需要有容器环境，此处使用docker*

## 1、配置yum源

```bash
yum install -y yum-utils
yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 2、安装docker

```bash
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```

## 3、启动（现在启动且开机自启）

```bash
systemctl enable docker --now
```

**配置加速（此处不配）**

## 4、修改配置（后续不使用docker可跳过）

*这里额外添加了docker的生产环境核心配置cgroup（exec-opts）*

```bash
tee /etc/docker/daemon.json <<-'EOF'
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
 "max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF

systemctl daemon-reload

systemctl restart docker
#检查配置
docker info
```

## 5、下载nfs镜像

```perl
docker pull k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
```

## 6、打包镜像

```perl
docker save k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2 > nfs-subdir-external-provisioner-v4.0.2.tar
```

## 7、将镜像传出