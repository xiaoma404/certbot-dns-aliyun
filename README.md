# [certbot-dns-aliyun](http://www.madmalls.com/blog/post/get-wildcard-certificate-on-letsencrypt/)

用 Let's Encrypt 官方工具 Certbot 申请通配符证书（Wildcard Certificate）时，只能用 DNS-01 的方式来验证域名所有权，需要在域名下添加一条 DNS TXT 记录。如果要用 certbot renew 命令自动续期的话，就需要自动添加或删除 DNS TXT 记录。官方提供的都是国外的 DNS 服务商的插件，而国内的 Aliyun DNS 也提供了 DNS 云解析管理 API，此工具是用 Go 语言调用 API 实现自动添加和删除 DNS TXT 记录，从而实现自动用 certbot renew 命令续期通配符证书的目的！


# 1. 如何使用

## 1.1 获取 certbot-dns-aliyun 插件

如果你是 Linux 系统，那么只需要克隆本项目到本地后，将我编译好的可执行程序 `certbot-dns-aliyun` 复制到你指定的位置（比如 `/etc/letsencrypt/` 目录下）即可

```bash
[root@CentOS ~]# git clone https://github.com/xiaoma404/certbot-dns-aliyun.git

[root@CentOS ~]# cp certbot-dns-aliyun/certbot-dns-aliyun /etc/letsencrypt/
```

然后，在此程序同样的位置（比如 `/etc/letsencrypt/` 目录下）新建 `config.json` 文件，里面是你的 Aliyun DNS accesskey，格式如下:

```
{
    "domain": "你的主域名xxx.com不要带二级域名"
    "regionID": "你的RegionID",
    "accessKeyID": "你的AccessKeyID",
    "accessKeySecret": "你的AccessKeySecret"
}
```

你也可以查看一下此程序的帮助文档：

```bash
[root@CentOS ~]# /etc/letsencrypt/certbot-dns-aliyun -h
```

## 1.2 申请通配符证书

```bash
certbot certonly \
  --non-interactive \
  --agree-tos \
  --manual-public-ip-logging-ok \
  --manual --preferred-challenges dns-01 \
  --manual-auth-hook "/etc/letsencrypt/certbot-dns-aliyun -o authenticator" \
  --manual-cleanup-hook "/etc/letsencrypt/certbot-dns-aliyun -o cleanup" \
  -d *.madmalls.com -d madmalls.com \
  --server https://acme-v02.api.letsencrypt.org/directory
```

你只需要替换 `-d` 的值即可

> 如果你只是想测试 Certbot 功能，可以指定测试环境的 API: `--server https://acme-staging-v02.api.letsencrypt.org/directory`

