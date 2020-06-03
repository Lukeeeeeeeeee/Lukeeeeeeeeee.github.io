---
title: GitHub加载过慢或图片加载失败
date: 2020-06-03 16:42:31
tags: [GitHub]
---

通过 [ip地址查询](https://www.ipaddress.com/) 查看 GitHub 的 ip。

### Mac

在 Mac 下执行 `sudo vim /etc/hosts` 命令修改 hosts 文件。随便按一个键，进入 INSERT 模式，在最后一行添加如下信息：

```
# GitHub Start
140.82.114.4      github.com
140.82.114.20     gist.github.com
140.82.113.4      gist.github.com

151.101.184.133    assets-cdn.github.com
151.101.184.133    raw.githubusercontent.com
151.101.184.133    gist.githubusercontent.com
151.101.184.133    cloud.githubusercontent.com
151.101.184.133    camo.githubusercontent.com
151.101.184.133    avatars0.githubusercontent.com
199.232.68.133     avatars0.githubusercontent.com
199.232.28.133     avatars1.githubusercontent.com
151.101.184.133    avatars1.githubusercontent.com
151.101.184.133    avatars2.githubusercontent.com
199.232.28.133     avatars2.githubusercontent.com
151.101.184.133    avatars3.githubusercontent.com
199.232.68.133     avatars3.githubusercontent.com
151.101.184.133    avatars4.githubusercontent.com
199.232.68.133     avatars4.githubusercontent.com
151.101.184.133    avatars5.githubusercontent.com
199.232.68.133     avatars5.githubusercontent.com
151.101.184.133    avatars6.githubusercontent.com
199.232.68.133     avatars6.githubusercontent.com
151.101.184.133    avatars7.githubusercontent.com
199.232.68.133     avatars7.githubusercontent.com
151.101.184.133    avatars8.githubusercontent.com
199.232.68.133     avatars8.githubusercontent.com
# GitHub End
```

添加完毕后，按 ESC 键退出 INSERT 模式，输入 `:wq` 命令退出。

注意：当出现 GitHub 加载过慢或者图片加载失败时，通过控制台查看该网址并通过上面的 ip地址查询网站 中查看对应的 ip地址，将它添加到 hosts 文件中即可。

### Windows

暂时还没尝试。