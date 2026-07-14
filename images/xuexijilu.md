# 学习记录  
### 记录人：  
**邹晓南**  
### 学习日期：  
**2026.07.14**  
### 学习内容  
- [x] Linux入门命令  

## 一、学习过程  
### 1.1今日学习命令清单  
| 常用命令 | 主要功能 |  
| ------- | ------ |  
| pwd     | 查看当前所在路径|  
| ls      | 查看目录      |  
| cd      | 切换目录      |  
| mkdir   | 创建目录      |  
| touch   | 创建文件      |  
| cp      | 复制文件/目录  |  
| mv      | 移动/重命名文件|  
| rm      | 删除文件      |  
| cat     | 查看文件内容   |  
| less    | 分页查看      |  
| find    | 查找文件       |  
| grep    | 搜索文本行     |  
|ip addr| 查看本机网卡及ip信息|    
| ping    | 测试网络联通性  |  
|wget / curl|  下载文件   |  

### 1.2命令学习与实操记录  
**1.2.1终端**  
<img src="2026-07-14 141014.png" width="400">   
**1.2.2查看当前目录**  
`pwd`  
![alt text](<2026-07-14 141036.png>)  
**1.2.3查看文件**  
`ls`   
`ls -a`   
  <img src="2026-07-14 141101.png" width="400">   
**1.2.4切换目录**  
`cd`  
  <img src="2026-07-14 141225.png" width="400">   
**1.2.5创建文件和目录**  
`mkdir`   
`touch`   
<img src="2026-07-14 151637.png" width="400">    
**1.2.6查看文件内容**  
`cat`  `less`  `head`  `tail`  
  <img src="2026-07-14 151655.png" width="400">    
**1.2.7复制、移动、删除**  
`cp`  
    <img src="2026-07-14 151717.png" width="400">    
`mv`  
<img src="2026-07-14 151732.png" width="400">     
`rm`  
<img src="2026-07-14 151744.png" width="400">  
**1.2.8查看系统信息**  
`whoami` `date` `uname -a`   
<img src="2026-07-14 152022.png" width="400">  
**1.2.9网络命令**  
`ip addr`  
<img src="2026-07-14 152045-1.png" width="400">    
`ping`    
<img src="2026-07-14 152102-1.png" width="400">

## 二、遇到的问题及解决方案  
1. **问题**：VSCode Markdown 图片不显示  
**解决**：图片放入虚拟机 `images` 目录，使用相对路径 `![图片](./images/xxx.png)`；VSCode 打开整个文件夹。

2. **问题**：Windows 图片无法传入虚拟机  
**解决**：安装工具并重启  
   ```bash
    sudo apt update
    sudo apt install open-vm-tools open-vm-tools-desktop -y
    sudo reboot  
    ```  

## 三、学习总结  
     今天主要学习了Linux基础命令和Markdown笔记编写，并解决了虚拟机环境下的实操问题。通过练习掌握了pwd、ls、cd、mkdir、touch、cp、mv、rm、cat等常用命令，熟悉了文件与目录的基本操作。同时学会在VSCode中使用Markdown记录学习内容，并成功实现虚拟机与Windows之间图片传输和笔记插图。遇到的图片不显示、文件无法拷贝、路径错误、命令参数遗漏等问题，都已通过安装工具、规范路径、注意命令用法全部解决。整体学习顺利，为后续Linux学习打下了基础。