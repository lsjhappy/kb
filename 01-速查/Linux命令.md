# Linux 命令速查

> 全部是**只读**命令，生产上可以直接跑。

## 先确认自己在哪台机

```bash
hostname -I          # 看 IP，避免在错误的机器上操作
uptime               # 跑了多久、负载多少
who                  # 谁在线
```

> 负载怎么看：`load average` 三个数是 1/5/15 分钟平均。
> **超过 CPU 核数就是过载**。核数：`nproc`

## 看进程和端口

```bash
ps -ef | grep java | grep -v grep        # 所有 Java 进程（带 jar 路径）
ss -lntp                                 # 监听的端口 + 对应进程
ss -lntp | grep 9090                     # 只看某个端口
lsof -p <PID> | head -30                 # 某进程打开了哪些文件
```

## 看资源

```bash
df -h                    # 磁盘（先看这个，最常见）
df -i                    # inode（磁盘没满但写不进去，就是它）
free -h                  # 内存
top -b -n1 | head -20    # CPU 占用前几名
iostat -x 1 3            # 磁盘 IO（需要 sysstat）
```

## 看日志

```bash
tail -f /path/app.log                    # 实时跟
tail -n 200 /path/app.log                # 看最后 200 行
grep -i error /path/app.log | tail -30   # 只看错误
grep "$(date +%F)" /path/app.log | tail  # 只看今天
```

> ⚠️ **不要 `cat` 几百 MB 的日志**，会把终端卡死，也会给机器压力。
> 用 `tail` / `grep` / `less`。

## 找文件和目录

```bash
find /opt -name '*.jar' 2>/dev/null | head -20
find /opt -type d -iname '*log*' 2>/dev/null
du -sh /opt/* 2>/dev/null | sort -h | tail -10   # 哪些目录最占空间
```

## 服务管理（CentOS 7）

```bash
systemctl status <服务名>     # 状态（只读）
systemctl list-units --type=service --state=running   # 在跑的服务
journalctl -u <服务名> -n 100 --no-pager              # 服务日志
```

## 什么时候用什么

| 现象 | 先看 |
|---|---|
| 程序起不来 | `systemctl status` + `journalctl` |
| 连不上 | `ss -lntp` 看端口在不在听 |
| 写不进去 | `df -h`、`df -i` |
| 变慢了 | `top`、`iostat`、`uptime` |
| 不知道装在哪 | `ps -ef` 看启动命令里的路径 |
