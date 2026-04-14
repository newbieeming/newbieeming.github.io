---
title: linux工具配置
date: 2026-01-18 20:48
categories:
  - linux
  - ssh
---

#### ssh配置

```shell
# 安装ssh
sudo apt install openssh-server
# 开机自欺服务
sudo systemctl enable ssh
# 重启ssh服务
sudo systemctl restart ssh
```

#### 配置静态ip

```shell
# 启用systemd-networkd
sudo systemctl start systemd-networkd
# 安装vim
sudo apt-get install vim-gtk
# 安装
sudo apt install net-tools
# 查询网卡名称
ip a
# 进入配置文件夹
cd /etc/netplan
# 备份文件
sudo cp 01-network-manager-all.yaml 01-network-manager-all.yaml.bak
# 编辑文件
sudo vi 01-network-manager-all.yaml
### 具体内容如下： (网关可以去设置新建动态网络)
#-------------------------------------------------------------------------------
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:            # 网卡名称（用 `ip a` 确认）
      dhcp4: false    # 禁用 DHCP
      addresses:
        - 192.168.159.188/24  # 静态 IP + 子网掩码
      routes:
        - to: default
          via: 192.168.159.2   # 默认网关
      nameservers:
        addresses:
          - 192.168.159.2
# -------------------------------------------------------------------------------
# Ubuntu18
#-------------------------------------------------------------------------------
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:            # 网卡名称（用 `ip a` 确认）
      dhcp4: false    # 禁用 DHCP
      addresses:
        - 192.168.159.188/24  # 静态 IP + 子网掩码
      gateway4: 192.168.159.2
      nameservers:
        addresses:
          - 192.168.159.2
# -------------------------------------------------------------------------------
# 生效修改
sudo netplan apply
# 查询是否生效
ifconfig
```

