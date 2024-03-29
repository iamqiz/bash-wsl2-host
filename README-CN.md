# wsl2hosts
[English DOC](README.md)
## 一个bash脚本实现自动更新windows hosts,来映射到WSL的ip地址
### 介绍
在windows里若想连接WSL(适用于Windows的Linux子系统),需要通过WSL的ip,但是该ip在每次重启wsl后都会改变,  
解决方法是将wsl的ip跟一个固定名称(比如说ubuntu2004.wsl)绑定,写在windows hosts文件(C:\Windows\System32\drivers\etc\hosts)里,  
然后每次在wsl重启后都自动更新windows hosts文件里的ip为wsl当前的ip, 就可以始终使用 ubuntu2004.wsl 这个名称来ssh连接到WSL.  
windows hosts里会有下面这样一行(这里ip是示例,表示wsl的ip):  
`123.45.67.89  ubuntu2004.wsl`

### 依赖
只需要安装一个工具,叫gsudo,用来以管理员身份运行命令, 否则没有权限修改windows hosts文件

下载地址:
gerardog/gsudo: A Sudo for Windows  
https://github.com/gerardog/gsudo

### 使用方法

1. 首先需要在windows端安装好gsudo(我是通过scoop 下载gsudo的)  
如何安装scoop见:
https://github.com/ScoopInstaller/Scoop#installation  
通过scoop安装gsudo如下:  
`scoop install gsudo`  
安装后在cmd里执行`where gsudo` 查看位置,我的在:  
C:\Users\用户名\scoop\apps\gsudo\current\gsudo

2. 下载本仓库下的bash脚本,内容如下

将你下载的gsudo和powershell.exe对应的wsl路径(不是windows路径)加到脚本里的PATH里 

修改脚本里 _wsl_ssh_ip_name 变量的值为你想要的名字,这里是 "ubuntu2004.wsl"


```bash
#!/bin/bash

# adjust this paths to your environment 下面改成你的gsudo位置; powershell路径一般就是这个路径
export PATH=/mnt/c/Users/用户名/scoop/apps/gsudo/current/:$PATH
export PATH=/mnt/c/Windows/System32/WindowsPowerShell/v1.0/:$PATH

change_wsl_ssh_ip(){
  # define your dns name
  _wsl_ssh_ip_name="ubuntu2004.wsl"

  # set windows hosts file path
  _wslhosts=/mnt/c/Windows/System32/drivers/etc/hosts
  _winhosts='C:\Windows\System32\drivers\etc\hosts'

  # get wsl real ip and ignore wsl-vpnkit
  _wsl_ssh_ip_addr=$(ip -4 addr show "eth0" | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | head -1)
  echo "wsl ssh ip addr:[$_wsl_ssh_ip_addr]"

  # get current ip from windows hosts
  _win_ssh_ip_addr=$(grep -oP "\d+(\.\d+){3}(?=\s*$_wsl_ssh_ip_name)" $_wslhosts)
  echo "win ssh ip addr:[$_win_ssh_ip_addr]"

  # check if ip exists or diff, then modify
  if [[ "$_win_ssh_ip_addr" == "" ]]
  then
    echo "ssh ip is missing, adding to windows hosts"
    gsudo powershell.exe -Command "echo '' '$_wsl_ssh_ip_addr  $_wsl_ssh_ip_name' | out-file -encoding ASCII $_winhosts -append"
  else
    if [[ "$_win_ssh_ip_addr" != "$_wsl_ssh_ip_addr" ]]
    then
      echo "ssh ip diff, modifying windows hosts"
      gsudo powershell.exe -Command "(gc $_winhosts) -replace '$_win_ssh_ip_addr', '$_wsl_ssh_ip_addr' | out-file -encoding ASCII $_winhosts"
    else
      echo "ssh ip ok"
    fi
  fi

  # resulting windows hosts
  echo "now $_wsl_ssh_ip_name 's ip is:"
  grep "$_wsl_ssh_ip_name" $_wslhosts
}

# run the function
change_wsl_ssh_ip

```

3. 在~/.bashrc里执行上面的bash脚本,(路径改成自己的):
```bash
source  /path/to/wsl2hosts.sh
```
这样每次启动wsl时,会自动执行bash脚本,如果ip发生变化,会进行修改

4. 重启WSL看看效果,  
在windows cmd里执行 wsl --shutdown 来关闭所有wsl
然后重启wsl, 控制台会打印win和wsl里的ip信息,如果不一致或不存在,会进行修改 

windows hosts文件里会出现下面的语句,(这里ip地址为示例) 
```text
...
123.45.67.89  ubuntu2004.wsl 
...
```
