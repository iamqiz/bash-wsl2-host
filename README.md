# Bash-wsl2-host
[中文文档](README-CN.md)
## bash script to Automatically update your Windows hosts file with the WSL2 VM IP address 

### Dependencies
the only tool you need to install is "gsudo",it is used to run command with administrator

gerardog/gsudo: A Sudo for Windows  
https://github.com/gerardog/gsudo

### Usage
1. download gsudo in windows,  
then add gsudo and powershell to path:
```bash
export PATH=/mnt/c/Users/pcmsi/scoop/apps/gsudo/current/:$PATH
export PATH=/mnt/c/Windows/System32/WindowsPowerShell/v1.0:$PATH
```

2. define the function to modify windows hosts file (C:\Windows\System32\drivers\etc\hosts in windows 
or /mnt/c/Windows/System32/drivers/etc/hosts in WSL)

```bash
change_wsl_ssh_ip(){
  # define your ip name
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
3. add change_wsl_ssh_ip to .bashrc

4. finaly add your host name to windows hosts, ip can be arbitrary,it will be replaced with real value;  
look like this :
```text
.....
123.45.67.89   ubuntu2004.wsl 
....
```

5. now restart your wsl to check whether the bash work ,
(run wsl --shutdown  to close wsl )

