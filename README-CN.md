# wsl2hosts
[English DOC](README.md)
## 一个bash脚本实现自动更新windows hosts,来映射到WSL的ip地址

### 依赖
只需要安装一个工具,叫gsudo,用来以管理员身份运行命令, 否则没有权限修改windows hosts文件

下载地址:
gerardog/gsudo: A Sudo for Windows  
https://github.com/gerardog/gsudo

### 使用方法

1. 首先需要在windows端安装好gsudo(我是通过scoop 下载gsudo的),   
然后把命令的路径和powershell.exe的目录加到PATH环境变量里,如下:
```bash
export PATH=/mnt/c/Users/pcmsi/scoop/apps/gsudo/current/:$PATH
export PATH=/mnt/c/Windows/System32/WindowsPowerShell/v1.0:$PATH
```

2. 把修改 windows hosts ip的脚本放到函数里: 
(注windows hosts 文件位置为: C:\Windows\System32\drivers\etc\hosts  
在WSl里对应/mnt/c/Windows/System32/drivers/etc/hosts)

```bash
# --------------------------------------------------------------------------------------------------
# Name:        .wsl2hosts.sh
# Version:     1.0.0
# Date:        2023-02-21
# Author:      berndgz inspired by https://github.com/iamqiz/bash-wsl2-host
# Description: bash script to automatically update your Windows hosts file with the WSL2 VM IP addr.
# Requirement: WSL2, WindowsPowerShell, https://github.com/gerardog/gsudo, elevated privileges.
# Note:        gsudo is a sudo equivalent for Windows, with a similar user-experience.
# Usage:       add location from gsudo and WindowsPowerShell to PATH environment variable.
#              add this script to ~/.bashrc and define your dns name in '_wsl_ssh_ip_name' variable.
# --------------------------------------------------------------------------------------------------

# adjust this paths to your environment
export PATH=/mnt/c/Users/your-USERNAME/gsudo/x64/:$PATH
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

  # get current entry from windows hosts
  # _entry=`grep "$_wsl_ssh_ip_name" $_wslhosts`
  # _entry=${_entry/$_win_ssh_ip_addr/$_wsl_ssh_ip_addr}
  # echo "$_entry"

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
  _hosts=`cat $_wslhosts`
  echo "$_hosts"
}

change_wsl_ssh_ip

```

3. 然后把 change_wsl_ssh_ip 函数放到.bashrc 脚本里,让它每次启动都执行

4. 把上一步定义的host名字写到 windows hosts文件里,前面的ip随便写,不写也行

```text
...
123.45.67.89  ubuntu2004.wsl 
...
```

5. 重启WSL看看效果,
在windows cmd里执行 wsl --shutdown 来关闭所有wsl
然后重启wsl
