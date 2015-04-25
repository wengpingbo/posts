title: 在linux下利用Google的SMTP来发邮件
author: WEN Pingbo <wengpingbo@gmail.com>
date: 2013/05/18
tags: [linux, smtp]
categories: linux
---

在维护服务器的时候，经常需要建立一个服务器错误预警系统，而邮件是一个很好的途径。

在 linux 下，一般是通过 mail 来写邮件，而传递默认使用 sendmail 服务。这样虽然能向外界发送邮件，但邮件不能回复，并且 sendmail 服务要求发送方是系统可识别用户，配置比较麻烦。下面通过使用 Google 的 SMTP 服务器来发送邮件，不但减轻服务器负担，而且可以使用类似的公共邮件地址来作为发送方。

下面所有步骤，全部基于 CentOS 6.3，其他发行版本类似。

## 安装mail

```
yum install mailx -y
```

如果想直接使用 sendmail 来发送邮件，需要启动 sendmail 服务，或者 saslauthd 服务

## 配置 SMTP
如果想利用外部 SMTP 发送邮件，需编辑 /etc/mail.rc，加入以下内容

```
set from=demo@qq.com 
set smtp=smtp.qq.com  
set smtp-auth-user=demo 
set smtp-auth-password=demopass 
set smtp-auth=login
```

但是这个设置只适合那些支持非 SSL 链接的 SMTP 服务器，但对于想 Google 这样，强制使用 SSL 加密连接的，需要加入 SSL 支持。

## 加入 SSL 支持

在裝有 Firefox 的 Linux 电脑, 將 `~/.mozilla/firefox/xxxxxxxx.default/` 的 `cert*.db` 与 `key*.db` 复制到 `~/.mozilla_nss_shared_db`

编辑 /etc/mail.rc，加入以下内容

```sh
set ssl-verify=ignore
set nss-config-dir=~/.mozilla_nss_shared_db
set from="myaccount@gmail.com(myname)"
set smtp=smtps://smtp.gmail.com:465
set smtp-auth=login
set smtp-auth-user=myaccount
set smtp-auth-password=mysecret
```

如果想添加多个帐号，那就这样写配置文件

```sh
account starttls {
	set smtp-use-starttls
	set ssl-verify=ignore
	set nss-config-dir=~/.mozilla_nss_shared_db
	set from="myaccount@my.smtp.host(myname)"
	set smtp=smtp://my.smtp.host:25
	set smtp-auth=login
	set smtp-auth-user=myaccount
	set smtp-auth-password=mysecret
}
account gmail {
	set ssl-verify=ignore
	set nss-config-dir=~/.mozilla_nss_shared_db
	set from="myaccount@gmail.com(myname)"
	set smtp=smtps://smtp.gmail.com:465
	set smtp-auth=login
	set smtp-auth-user=myaccount
	set smtp-auth-password=mysecret
}
```

不过在发送邮件的时候，需要用 -A 参数指定发送帐号，比如 -A gmail。
