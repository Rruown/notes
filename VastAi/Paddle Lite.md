# Paddle Lite

## 概要

## paddle lite是一组工具，以实现在多端设备（移动端设备、物联网设备等）上运行机器学习模型

## 开发流程

### 生成paddle lite模型

使用paddle lite opt工具将paddle模型转换成paddle lite模型

```bash
pip install paddlelite // 安装paddle lite
paddle_lite_opt --model_dir=./mobilenet_v1 \   // 模型路径
                --optimize_out_type=naive_buffer \
                --optimize_out=./mobilenet_v1_op  // 输出的paddle lite文件
```

### 运行推断

使用`Paddle Lite API`加载、运行Paddle Lite模型，支持 [Java](https://paddle-lite.readthedocs.io/zh/latest/user_guides/java_demo.html)、[C++ API](https://paddle-lite.readthedocs.io/zh/latest/api_reference/cxx_api_doc.html?highlight=C%2B%2B)、[Python](https://paddle-lite.readthedocs.io/zh/latest/user_guides/python_demo.html)

Paddle Lite API库可以通过[官方预测库下载（支持Android/iOS/x86/macOS 平台）](https://paddle-lite.readthedocs.io/zh/latest/quick_start/release_lib.html)、[Docker开发环境编译](https://paddle-lite.readthedocs.io/zh/develop/source_compile/docker_env.html)获得

#### **1. 配置参数**

##### `CXXConfig`

```c++
#include <paddle_api.h>
class CxxConfig;
```

`CxxConfig` 用来配置构建 Full feature predictor 的配置信息，如 protobuf 格式的模型地址、能耗模式、工作线程数、place 信息等等

设置/获取模型文件夹，**加载非combined模型**

```c++
void set_model_dir(const std::string& x);  

const std::string& model_dir();
```

设置/获取模型文件，**加载combined模型**

```c++
void set_model_file(const std::string& path);

std::string model_file();
```



##### `MobileConfig`

```c++
#include <paddle_api.h>
class MobileConfig;
```

`MobileConfig` 用来配置构建light weight predictor 的配置信息，如 NaiveBuffer 格式的模型地址、模型的内存地址（从内存加载模型时使用）、能耗模式、工作线程数等等。

> 注意：输入的模型需要使用 [Model Optimize Tool](https://paddle-lite.readthedocs.io/zh/develop/user_guides/model_optimize_tool.html)转化为 NaiveBuffer 格式的优化模型。

设置模型文件

```c++
void set_model_from_file(const std::string& x);
```



#### 2.创建Predictor

```c++
#include <paddle_api.h>

template <typename ConfigT>
std::shared_ptr<PaddlePredictor> CreatePaddlePredictor(const ConfigT&);
```

`CreatePaddlePredictor` 用来根据 `Config` 构建预测器`PaddlePredictor`对象。

#### 3.执行Predictor

```c++
#include <paddle_api.h>

class PaddlePredictor;
```

`PaddlePredictor` 是 Paddle Lite 的预测器，由 `CreatePaddlePredictor` 根据 `Config` 进行创建。用户可以根据 PaddPredictor 提供的接口设置**输入数据、执行模型预测、获取输出**以及获得当前使用 lib 的版本信息等。



**1.通过PaddlePredictor的GetInput接口获取输入的Tensor**

```c++
std::unique_ptr<Tensor> input_tensor(std::move(predictor->GetIntput(x)));
```

**2.调整Tensor的维度以适应输入数据的维度**

```c++
void Tensor::Resize(const shape_t &shape);
input_tensor->Resize({1, 2, 3, 4});
```

**3.向Tensor中填充输入数据**

获取Tensor底层数据的常量指针（n维的shape，底层使用一维数组），根据传入的不同模型类型获取相应数据。

```c++
template <typename T> const T* data() const;
```

设置Tensor底层数据

```c++
template <typename T> T* mutable_data() const;

auto* data = input_tensor->mutable_data<float>();
// 设置 Tensor 数据
for (int i = 0; i < ShapeProduction(input_tensor->shape()); ++i) {
  data[i] = 1;
}
```

**4.运行predictor**

```c++
predictor->Run();
```

**5.获取predictor的输出结果**

通过PaddlePredicator的GetOutput接口获取输出Tensor

```c++
std::unique_ptr<Tensor> out_tensor(std::move(predictor->GetOutput(x)));
```

获取输出Tensor底层数据的常量指针

```c++
auto* output = output_tensor->data<float>();
```

