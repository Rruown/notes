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