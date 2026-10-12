# 02：Linux 常用命令

以下命令都在 SSH 登录服务器后运行。

## 目录和文件

```bash
pwd
ls -lah
cd /var/www
cd ..
cd ~
```

`pwd` 显示当前位置，`ls -lah` 显示详细列表，`cd` 切换目录。

只读查看文件：

```bash
cat 文件名
less 文件名
head -n 30 文件名
tail -n 50 文件名
```

在 `less` 中按 `q` 退出。持续查看日志用 `tail -f 文件名`，按 `Ctrl+C` 停止。

## 系统资源

```bash
uptime
free -h
df -h
du -sh /var/www/*
```

`free -h` 看内存，`df -h` 看分区空间，`du -sh` 看目录占用。磁盘接近 100% 时，服务可能无法写日志或启动。

## 进程、端口和服务

```bash
ps aux
top
ss -lntp
systemctl status nginx
journalctl -u nginx -n 100 --no-pager
```

在 `top` 中按 `q` 退出。以上 `status`、`journalctl` 和 `ss` 用法均为只读检查。

## 修改前先备份

```bash
cp 原文件 原文件.bak
```

必须换成实际文件名。修改 Nginx 后先运行 `nginx -t`，通过后才可重载。

## 高风险操作

谨慎对待 `rm`、`mv`、`chmod`、`chown`、`systemctl stop/restart`、防火墙和 SSH 配置修改，以及下载后直接交给 shell 的脚本。不清楚时先看 `命令 --help` 或 `man 命令`。
