# Bash-wsl2-host
## 一个bash脚本实现自动更新windows hosts映射到WSL的ip地址

只需要安装一个工具,叫gsudo,用来以管理员身份运行命令, 否则没有权限修改windows hosts文件

下载地址:
gerardog/gsudo: A Sudo for Windows  
https://github.com/gerardog/gsudo

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
change_wsl_ssh_ip(){
  # define your ip name 这里定义host名字,
  _wsl_ssh_ip_name="ubuntu2004.wsl"
  # get wsl real ip
  _wsl_ssh_ip_addr=$(ip -4 addr show "eth0" | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
  echo "wsl ssh ip addr:[${_wsl_ssh_ip_addr}]"
  grep "${_wsl_ssh_ip_name}" /mnt/c/Windows/System32/drivers/etc/hosts
  # get current ip from windows hosts
  _win_ssh_ip_addr=$(grep -oP "\d+(\.\d+){3}(?=\s*${_wsl_ssh_ip_name})" /mnt/c/Windows/System32/drivers/etc/hosts)
  echo "win ssh ip addr:[${_win_ssh_ip_addr}]"
  # check whether ip change, if change,modify
  if [ "${_win_ssh_ip_addr}" == "${_wsl_ssh_ip_addr}" ]; then
    echo "ssh ip same"
  else
    echo "ssh ip diff, call gsudo powershell to modify windows hosts"
    # the gsudo is used to run command with administrator;
    # the powershell is to replace "<old_ip>   <_wsl_ssh_ip_name>" line with "<new_ip>  <_wsl_ssh_ip_name>" in windows hosts
    gsudo powershell.exe -Command "(Get-Content 'C:\Windows\System32\drivers\etc\hosts') -replace '.*${_wsl_ssh_ip_name}', '${_wsl_ssh_ip_addr}  ${_wsl_ssh_ip_name}'| Set-Content 'C:\Windows\System32\drivers\etc\hosts'"
  fi
}
```
3. 然后把 change_wsl_ssh_ip 函数放到.bashrc 脚本里,让它每次启动都执行


4. 把上一步定义的host名字写到 windows hosts文件里,前面的ip随便写,不写也行

```text
.....
123.45.67.89   ubuntu2004.wsl 
....
```

5. 重启WSL看看效果,
在windows cmd里执行 wsl --shutdown 来关闭所有wsl
然后重启wsl
