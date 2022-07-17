# python 统一开发环境

# 背景
windows 上 ansible装不上，曲线救国55555

# 优点
统一python 的开发运行环境，避免浪费时间解决环境问题

# 怎么做
1 装一台linux虚拟机
2 安装miniconda
```
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py38_4.8.2-Linux-x86_64.sh
```
3 使用conda创建一个纯净的环境
```
conda create -n venv  --clone base
```
4 使用pycharm配置远程环境
![](https://images.xiaozhuanlan.com/photo/2021/caaff2bbc794e806b0ff5e60357434b2.png)
5 自定义解释器的脚本
```
#!/usr/bin/env bash
source /root/miniconda3/etc/profile.d/conda.sh
conda activate venv
python "$@"   
```
6 配置对应的解释器
![](https://images.xiaozhuanlan.com/photo/2021/f4ed1460c4bc6ff4fe113e0fc3c596c9.png)
7 配置代码检查
在本地的环境安装 flake8
```
pip install flake8 -i  https://pypi.tuna.tsinghua.edu.cn/simple
```
配置pycharm
![](https://images.xiaozhuanlan.com/photo/2021/9b9fa858682e8fc17cefd6af90a57e9a.png)
![](https://images.xiaozhuanlan.com/photo/2021/4dcbd3baa32c49429cdd67b0b470b705.png)
```
program 选本地的python解释器
arguments： -m flake8 --show-source --statistics $ProjectFileDir$
working directory：$ProjectFileDir$
```
![](https://images.xiaozhuanlan.com/photo/2021/c3f3a9ce76133247d0f3fbc8478d86ef.png)
使用 tools中的external tools 做检查
8 使用black做代码格式化
在本地的环境安装 black
```
pip install black -i  https://pypi.tuna.tsinghua.edu.cn/simple
```
配置pycharm
![](https://images.xiaozhuanlan.com/photo/2021/594d47b9f00d9310367ea2a07d7db2bc.png)
![](https://images.xiaozhuanlan.com/photo/2021/355a98851676cbe0c4a6ba2d72c61b1a.png)
```
program 选本地的python解释器
arguments：-m black $FilePath$
working directory：$ProjectFileDir$
```
![](https://images.xiaozhuanlan.com/photo/2021/e29cbf92681a772f6c19a3694932e240.png)
使用black对代码做格式化