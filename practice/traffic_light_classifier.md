## 安装Jupyter NoteBook

**1 设置Python：**
```shell
sudo apt install python3-pip python3-dev
```

**2 安装Python**
```shell
sudo apt install python3-pip python3-dev
```d

**3 安装Python virtualenv**
```shell
pip3 install --upgrade pip
pip3 install virtualenv
```

**4 创建Python虚拟环境**
```shell
mkdir notebook
cd notebook
virtualenv jupyterenv
source jupyterenv/bin/activate #load and activate the virtual environment using the following command.
```

**5 安装Jupyter Notebook**
```shell
pip install jupyter
```

**6 运行Jupyter Notebook**
```shell
jupyter notebook
```

## 