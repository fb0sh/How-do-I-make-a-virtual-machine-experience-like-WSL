# 1.终端配置
## 静态IP和hosts文件
给虚拟机设置静态IP

建议虚拟机用户名和Windows保持一致

并往hosts文件里写入 对应主机名 HOSTNAME 和 IP

已安装vmtools
## ssh服务器配置
```bash
openssh-server
```
## 免密认证

生成RSA 密钥对 设置为空密码
```bash
ssh-keygen -t rsa
```
使用ssh-copy-id 复制到虚拟机 
```bash
ssh-copy-id HOSTNAME
```

## Windows Terminal配置
使用WindowsTerminal


打开 设置>打开JSON文件

"profiles"  > "list" 添加profile
```json
{
                "commandline": "ssh HOSTNAME",
                "guid": "{eadf7af9-2574-41c2-840f-b0afa915ce18}",
                "hidden": false,
                "icon": "C:\\Users\\USERNAME\\Pictures\\icons8-fedora-96.png",
                "name": "Fedora"
}
```

# 2.映射Windows文件夹到 Linux
## 配置Vmware 文件夹共享
![image](https://github.com/user-attachments/assets/2893c69f-d718-4e4e-9799-ca23acdd3eea)
## 创建对应磁盘文件夹
```bash
sudo mkdir /mnt/c /mnt/d /mnt/p
```
## 创建自动挂载服务
### 脚本
`/mnt/mount-win.sh`
```bash
#!/bin/bash
vmhgfs-fuse .host:/C /mnt/c -o allow_other
vmhgfs-fuse .host:/D /mnt/d -o allow_other
vmhgfs-fuse .host:/P /mnt/p -o allow_other
exit 0
```
### 服务文件
`/etc/systemd/system/mount_win.service`
```bash
[Unit]
Description=Mount VMware Shared Folder at Boot
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/mnt/mount-win.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
### 启动挂载服务
```bash
sudo systemctl daemon-reload
sudo systemctl start mount_win.service
sudo systemctl enable mount_win.service
```

# 3.映射Linux根目录到Windows
## 安装Samba
`sudo dnf install samba samba-client samba-common`
## 配置Samba
`/etc/samba/smb.conf`
添加以下内容
```bash
[root]
        comment = Share For Windows
        path = /
        browsable = yes
        read only = no
        guest ok = yes
        public = yes
        create mask = 0775
        directory mask = 0775
```
## 启动Samba服务
```bash
  sudo smbpasswd -a fb0sh
  sudo smbpasswd -e fb0sh

  sudo systemctl restart smb
  sudo systemctl restart nmb
  sudo firewall-cmd --permanent --add-service=samba
  sudo firewall-cmd --reload
```
## 映射为Windows网络驱动器

![image](https://github.com/user-attachments/assets/29aae32c-10e8-4d73-bd34-542fbdf3a53e)


# 4. Windows下 添加 文件夹空白处右键在Linux中打开
## 脚本
```powershell
# HKEY_CLASSES_ROOT\Directory\Background\shell\OpenInFedora\command
# cmd.exe /k powershell -NoProfile -ExecutionPolicy Bypass -File "P:\vms\Fedora\open_path.ps1" -WindowsPath "%V"
param (
    [string]$WindowsPath
)
function Convert-PathToLinuxStyle {
    param (
        [string]$path
    )
    
    # 将路径中的反斜杠替换为正斜杠
    $path = $path -replace '\\', '/'

    # 将驱动器字母转换为小写并替换冒号
    $driveLetter = ($path -split ':')[0].ToLower()
    
    # 替换路径中的驱动器字母部分
    $linuxPath = $path -replace "^[a-zA-Z]:", "/mnt/$driveLetter"
    
    return $linuxPath
}
# 将 Windows 路径转换为 WSL 格式
$WSLPath =Convert-PathToLinuxStyle -path $WindowsPath

# 输出转换后的路径（用于调试）
Write-Host "Converted Path: $WSLPath"

# 执行 SSH 命令
ssh -t HOSTNAME "cd $WSLPath && /usr/bin/bash"
```
## 注册表
```cmd
HKEY_CLASSES_ROOT\Directory\Background\shell\OpenInFedora\
HKEY_CLASSES_ROOT\Directory\Background\shell\OpenInFedora\command
cmd.exe /k powershell -NoProfile -ExecutionPolicy Bypass -File "P:\vms\Fedora\open_path.ps1" -WindowsPath "%V"
```


![image](https://github.com/user-attachments/assets/7c0002cb-3de8-4437-b0a1-74840aeab0c7)

# 开机自启动 虚拟机 后台
`autostart.bat`
```bash
"C:\Program Files (x86)\VMware\VMware Workstation\vmrun.exe" -T ws start "P:\vms\Fedora\Fedora.vmx" nogui
```

创建其快捷方式， 并使用Win+r 输入shell:startup 将快捷方式拖动进去
