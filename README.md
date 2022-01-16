# testing
BBR
- BBR是GOOGLE推出的拥塞控制算法，核心大体为：遇到丢包时，不单纯考虑“丢包率”这一个变量来控制速率，这种算法目前相比于Cubix协议能更好地提升传输速率。

直接开启BBR
- 1.自带的内核版本是Linux 4.9及以上的系统已经内置BBR但默认为关闭状态。
- 2.Ubuntu 19.04等已经默认开启BBR，不再需要手动开启。
- 3.CentOS 8.0+, Debian 9.5+, Ubuntu 18.04/18.10+ 等近几年的需要通过以下命令开启：
  ```
  echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
  echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
  sysctl -p
  ```
  
  重启，有bbr输出开启成功
  ```
  lsmod | grep bbr
  ```
  
老系统开启BBR
- CentOS 6.0/7.0, Debian 8, Ubuntu 16.04/16.10 等再老一点的系统建议为升级至最新版，如果实在不想升级，可以升级Linux内核版本，参考秋水逸冰的脚本：
  ```
  wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
  ```

  安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
  ```
  lsmod | grep bbr
  ```

注意
- BBR只针对TCP进行加速，对UDP不奏效。
- 可以不开启，但是大多数情况下建议开启。针对国内腾讯云/阿里云等1Mbps小宽带则可以不开启。
- 腾讯云5M固定宽带开启BBR后通过HTTPS下载文件最高速度无法达到带宽的理论最大值，关闭后则可以达到理论最大值（512kb/s）。
- BBR是单边加速算法，仅在服务器端安装即可成功加速。
- 类似于KVM、HyperV及VMWare的那种可以自定义内核的虚拟机才可以开启BBR，如果你的VPS是基于OpenVZ架构的，可能无法直接开启BBR，需要通过别的方法才能启动BBR；如果是基于XEN架构的VPS，请先确定是否可以自定义内核，因为有些Xen架构的VPS是不允许自定义内核的，通过这个命令可以看到内核版本：uname -r，若小于4.9，则无法开启。
- 配置了BBR之后，如果不想要了需要卸载的话，可以删除/etc/sysctl.conf上面添加的net.core.default_qdisc=fq和net.ipv4.tcp_congestion_control=bbr，然后重启。
  ```
  vim /etc/sysctl.conf
  reboot
  ```
  
- bbr消耗的流量微乎其微，大约占到正常流量的5%左右。
- 可以在开启BBR的同时启动TCP Fast Open：
  ```
  echo "net.ipv4.tcp_fastopen=3" >> /etc/sysctl.conf && sysctl -p
  ```
  其中参数3的含义：1 开启客户端，2 开启服务端，3 都开启。
- BBR默认同时开启IPv4和IPv6的算法
