# 使用nfs文件系统挂载
1. 服务器端挂载配置  https://cloud.tencent.com/developer/article/1626660  
2. 参考文章    ：[How to share a directory on a Linux machine to a Windows machine via NFS?](https://superuser.com/questions/1363062/how-to-share-a-directory-on-a-linux-machine-to-a-windows-machine-via-nfs)
## 挂载相关命令

### Linux side:

1. Install the NFS server and utilities (nfs-utils or your distribution's equivalent).  `sudo apt install nfs-kernel-server`
2. Create the directory `/srv/nfs`.  
`sudo mkdir -p /srv/nfs4/backups`
3. Create a new empty directory under /srv/nfs, e.g. files.
Bind-mount the created directory to the directory containing you want to share, e.g.:  
`sudo mount --bind /home/user/stuff/files /srv/nfs/files`  
Create or edit `/etc/exports`, and add the line:(the above assumes the Windows machine is on the same LAN as the Linux machine, with the subnet having a 192.168.0.0/16 prefix - adjust as needed).
``` 
/srv/nfs 192.168.0.0/16(rw,all_squash,no_subtree_check,anonuid=65534,anongid=65534)
``` 
4. Start the NFS server  
`sudo systemctl start nfs-server.service`

### Windows side:

1. In Control Panel, open Programs and Features, there find Add/remove Windows components, and enable Services for NFS and everything under it.
2. Open a command prompt, and type:
`mount -o anon \\192.168.0.1\srv\nfs\files Z:`
(assuming your Linux machine is at 192.168.0.1).
3. The directory should now be available on the Z: drive. (Type start Z: at the command prompt to open it in Explorer).

### FAQ

#### windows连接挂载显示网络连接失败（错误代码53）
by Ubuntu https://ubuntuforums.org/showthread.php?t=1402655
>NFS server has an option of working in insecure mode (Allowing higher incoming port numbers). Windows NFS client often uses higher port numbers. You can enable this option by adding an option to the share
>Example: /share *(insecure,rw)  
在服务器端exports文件中添加insecure
#### 在windows创建文件显示无权访问
> https://graspingtech.com/mount-nfs-share-windows-10/
1. Open `regedit` by typing it in the search box end pressing Enter.
2. Browse to `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default`.
3. Create a new `New DWORD (32-bit) Value` inside the `Default` folder named `AnonymousUid` and assign the `UID` found on the `UNIX` directory as shared by the NFS system.
4. Create a new `New DWORD (32-bit") Value` inside the `Default` folder named `AnonymousGid` and assign the GID found on the UNIX directory as shared by the NFS system.
5. Restart the NFS client or reboot the machine to apply the changes.

#### Windows输入mount提示：`位于命令管道位置 1 的 cmdlet New-PSDrive 请为以下参数提供值:Name: E PSProvider: C:`
reason：缺少“基于UNIX的应用程序子系统”一项。  
```
New-PSDrive -Name "z" -Root "\\10.147.17.145\srv\nfs" -Persist -PSProvider "FileSystem"
```
参考stackoverflow
> https://stackoverflow.com/questions/57582910/powershell-cant-run-mount
#### 重启nfs服务**  
` sudo /etc/init.d/nfs-kernel-server restart`
#### 验证是否正常使用**  
` sudo cat /proc/fs/nfsd/versions`