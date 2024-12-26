# 1.终端配置
## 静态IP和hosts文件
给虚拟机设置静态IP

建议虚拟机用户名和Windows保持一致

并往hosts文件里写入 对应主机名 HOSTNAME 和 IP
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
            },
```

# 2.映射Windows文件夹到 Linux
vm-fuese

# 3.映射Linux根目录到Windows
NFS
# 4. Windows下 添加 文件夹空白处右键在Linux中打开
## 脚本

## 注册表



