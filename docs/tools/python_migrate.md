# python依赖跨平台迁移

0、 创建虚拟环境

virtualenv --python=python3 venv

1、  在虚拟环境中安装依赖

pip install xxxxx

2、导出requirements文件

pip freeze >requirements.txt

3、下载pip依赖到本地

pip download -d package -r requirements.txt --only-binary=:all: --platform linux_x86_64  -i https://pypi.tuna.tsinghua.edu.cn/simple

4、离线安装

pip install --no-index --find-links=/packages/ -r requirements.txt