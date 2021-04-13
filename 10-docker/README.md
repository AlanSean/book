# Docker

## 前置设置

1.在启用或者关闭windows功能中，勾选
   1. Hyper-V
   2. Windows虚拟机监控程序平台
   3. 适用于Linux的windows子系统
   4. 虚拟机平台
   5. 开启cpu虚拟化(自行百度 ，不同主板 操作不同)

2.安装wsl2
   1. 检查运行 WSL 2 的要求

         若要更新到 WSL 2，需要运行 Windows 10。

         对于 x64 系统：版本 1903 或更高版本，采用 内部版本 18362 或更高版本。

         对于 ARM64 系统：版本 2004 或更高版本，采用 内部版本 19041 或更高版本。

         低于 18362 的版本不支持 WSL 2。 使用 Windows Update 助手更新 Windows 版本。
   2. 启用虚拟功能

      打开 PowerShell
      ```
         dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
      ```
   3. 下载 Linux 内核更新包

      - 查看自己的计算机类型，PowerShell运行`systeminfo | find "System Type"` 

      - [适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
      - [ARM的64包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi)
   4. PowerShell运行`wsl --set-default-version 2` ，将 WSL 2 设置为默认版本


##  安装docker
 - [windows-docker下载](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

1. 安装完成后，右击docker -> 属性 -> 兼容性 -> 已管理员启动(勾选) -> 应用，然后启动

2. 设置镜像加速器， 设置  -> Docker Engie 
      ```   
         {
            "registry-mirrors": [这里写你的镜像加速器],
            "insecure-registries": [],
            "debug": false,
            "experimental": false,
            "features": {
               "buildkit": true
            }
            }

      ```

## docker 常用命令

```

docker run [OPTIONS] IMAGE [COMMOND] [ARGS...]

# OPTIONS 说明
   --name：容器名字
   --rm：容器停止自动删除容器
   -i：--interactive,交互式启动
   -t：--tty，分配终端
   -v：--volume,挂在数据卷
   -d：--detach，后台运行
```

```