# API 

## JSON

**1 读JSON文件**：

```python
rd_json = json.load(open(os.path.join(root, 'TL_train.json')))
```

**2 剪切图片**：

```python
# Improting Image class from PIL module
from PIL import Image

# Opens a image in RGB mode
im = Image.open(r"C:\Users\Admin\Pictures\geeks.png")

# Size of the image in pixels (size of orginal image)
# (This is not mandatory)
width, height = im.size

# Setting the points for cropped image
left = 5
top = height / 4
right = 164
bottom = 3 * height / 4

# Cropped image of above dimension
# (It will not change orginal image)
im1 = im.crop((left, top, right, bottom))

# Shows the image in image viewer
im1.show()
```

# 工具

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

