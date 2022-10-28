# Paddle Lite

## 概要

paddle lite是一组工具，以实现在多端设备（移动端设备、物联网设备等）上运行机器学习模型

## docker环境准备

```Bash
#从 Dockerhub 直接拉取 Docker 镜像
docker pull paddlepaddle/paddle-lite:2.0.0_beta

#启动容器
docker run -itd --privileged\
  --name zxli_paddlelite \
  -v $PWD/Paddle-Lite:/Paddle-Lite \
  --net=host \
  paddlepaddle/paddle-lite:2.0.0_beta /bin/bash
```

## 源码编译

```Bash
# 下载 Paddle Lite 源码并切换到发布分支，如 develop
git clone https://github.com/PaddlePaddle/Paddle-Lite.git
cd Paddle-Lite && git checkout develop
# (可选) 删除 third-party 目录，编译脚本会自动从国内 CDN 下载第三方库文件# rm -rf third-party
./lite/tools/build_linux.sh --arch=x86 --with_extra=ON --with_log=ON --with_exception=ON --with_nnadapter=ON

#./lite/tools/build_linux.sh --arch=x86 --with_extra=ON --with_log=ON --with_exception=ON --with_nnadapter=ON --nnadapter_with_vastai=ON 
```

## 基础编译参数

Paddle Lite 仓库中`./lite/tools/build_linux.sh`脚本文件用于构建 linux 版本的编译包，通过修改`build_linux.sh`脚本文件中的参数，可满足不同场景编译包的构建需求，常用的基础编译参数如下表所示： 有特殊硬件需求的编译参数见后文。

## 开发流程

### 生成paddle lite模型

使用paddle lite opt工具将paddle模型转换成paddle lite模型

```bash
pip install paddlelite // 安装paddle lite
paddle_lite_opt --model_dir=./mobilenet_v1 \   // 模型路径
                --optimize_out_type=naive_buffer \
                --optimize_out=./mobilenet_v1_op  // 输出的paddle lite文件
```

或者使用full模型到处lite模型

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



**1）.通过PaddlePredictor的GetInput接口获取输入的Tensor**

```c++
std::unique_ptr<Tensor> input_tensor(std::move(predictor->GetIntput(x)));
```

**2）.调整Tensor的维度以适应输入数据的维度**

```c++
void Tensor::Resize(const shape_t &shape);
input_tensor->Resize({1, 2, 3, 4});
```

**3）.向Tensor中填充输入数据**

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

**4）.运行predictor**

```c++
predictor->Run();
```

**5）.获取predictor的输出结果**

通过PaddlePredicator的GetOutput接口获取输出Tensor

```c++
std::unique_ptr<Tensor> out_tensor(std::move(predictor->GetOutput(x)));
```

获取输出Tensor底层数据的常量指针

```c++
auto* output = output_tensor->data<float>();
```

## 底层源码

> Paddle Lite 方案架构

<img src="..\images\paddle_lite_with_nnadapter.jpg" alt="Paddle_Lite是实现架构" style="zoom: 80%;" />

### Paddle数据类型

#### Scope

> Scope是一个树形结构，树中每个节点都有一个map记录`变量名->Variable`的映射。



<img src="..\images\image-20221011112944460.png" alt="image-20221011141726663" style="zoom:80%;" />

**API：**

```c++
/// 创建一个孩子节点，返回孩子节点
Scope &Scope::NewScope() const;

/// 查找当前节点的变量
Variable *Scope::FindLocalVar(const std::string &name) const；
    
///向父节点方向查找变量
Variable *Scope::FindVar(const std::string &name) const；

/** 
 * 向父节点方向查找变量
 * 如果找到直接返回
 * 如果没有找到，在当前节点创建一个新Var返回
 */
Variable *Scope::Var(const std::string &name) 
...
```



**在Paddle-Lite中，Scope有两种：**

- `scope_`是根节点的**Scope**，用于存放持久化变量，包括`feed`变量和`fetch`变量。以及模型的权重参数等
- `exec_scope`是`scope_`的子节点，存放模型计算的中间变量

#### Variable

> 底层数据是`Any`类型，`Variable`能以任意类型的方式读取变量和修改变量
>
> [Any](https://dmlc-core.readthedocs.io/en/latest/doxygen/any_8h_source.html)是dml-core中的实现，dml-core是构建机器学习的一组工具包
>
> `Any`是一个**容器可以存放任意的数据类型**

**API:**

```c++
/// 以只读方法读取数据
template <typename T>
const T& Get() const；

/// 读写数据
template <typename T>
T* GetMutable()；

// 类型判断
template <typename T>
bool IsType()；
```



#### Tensor

底层是一个`buffer`（void类型数组）



### Paddle Lite网络结构描述

#### ProgramDesc

一个program由若干个block组成

#### BlockDesc

block由若干个op和var组成

var中存放了op输入输出变量的描述

#### OpDesc

op由输入域和输出域以及类型组成。Op的输入和输出都是key-value格式的map，每个Op不同的实现绑定到不同的key，如`feed_op`输入key是`X`，输出key是`Out`

**成员变量：**

```c++
// op的类型名
std::string type_;
// 输入的变量集合
std::map<std::string, std::vector<std::string>> inputs_;
// 输出的变量集合
std::map<std::string, std::vector<std::string>> outputs_;

// op的属性， k-v存放
// 如果op有kKernelTypeAttrz属性，则将取出值解析出kernel信息（kernel type、place、alias）
std::map<std::string, Any> attrs_;
std::map<std::string, AttrType> attr_types_;
```

**Op有三个特殊的类型：**

- `while`
- `conditional_block`
- `subgraph`





#### VarDesc

var由名称和类型组成



### `PaddlePredictor`

#### Lite API

> [paddle::lite_api::paddle_api.h](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/api/paddle_api.h)定义了PaddlePredictor的Lite接口，支持多种硬件包括ARM、X86、OpenCL、CUDA等

##### Tensor 

##### PaddlePredictor 

`CreatePaddlePredictor` 根据 `Config` 构建预测器。

> CreatePaddlePredictor有两个特化版本，根据不同的Config实例化不同的Predictor
>
> - CxxConfig 实例化 CxxPredictor 定义在
> - MobileConfig 实例化 LightPredictor

```C++
template <typename ConfigT>
std::shared_ptr<PaddlePredictor> CreatePaddlePredictor(const ConfigT&);
```

##### CxxModelBuffer 

##### CxxConfig->ConfigBase

##### MobileConfig->ConfigBase

### LightPredictor

> 定义实现于[light_api.h](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/api/light_api.h)、[light_api.cc](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/api/light_api.cc)、[light_api.impl](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/api/light_api_impl.cc)

#### `LightPredictor`

**成员变量:**

```C++

std::shared_ptr<Scope> scope_;
std::unique_ptr<RuntimeProgram> program_;
/// program描述
std::shared_ptr<cpp::ProgramDesc> program_desc_;


/// 输入名
std::vector<std::string> input_names_;
/// 输出名
std::vector<std::string> output_names_;


std::vector<PrecisionType> input_precisions_;
bool bool_clear_tensor_ = false;
```



**公有成员函数:**

```c++
LightPredictor(const std::string& lite_model_file,
                 bool model_from_memory = false,
                 bool use_low_precision = false);
void Run();

bool TryShrinkMemory();

bool use_low_precision_ = false;

// Get offset-th col of feed inputs.
Tensor* GetInput(size_t offset);
// get input by name.
Tensor* GetInputByName(const std::string& name);
// get output by name.
const Tensor* GetOutputByName(const std::string& name);
// Get offset-th col of fetch outputs.
const Tensor* GetOutput(size_t offset);

const lite::Tensor* GetTensor(const std::string& name) const;

// get inputnames and get outputnames.
std::vector<std::string> GetInputNames();
std::vector<std::string> GetOutputNames();

// get input tensor precision type
const std::vector<PrecisionType>& GetInputPrecisions() const;
void PrepareFeedFetch();
Scope* scope() { return scope_.get(); }
```

**私有成员函数:**

```c++
// check if the input tensor precision type is correct.
// would be called in Run().
void CheckInputValid();

void Build(const std::string& lite_model_file, bool model_from_memory = false);

void BuildRuntimeProgram(
    const std::shared_ptr<const cpp::ProgramDesc>& program_desc,
    bool use_precision_low);

void DequantizeWeight();

void ClearTensorArray(
    const std::shared_ptr<const cpp::ProgramDesc>& program_desc);
```

**LightPredictor初始化基本流程**

Build->（1）LoadModelNaiveFromMemory 

​	   （2）BuildRuntimeProgram

​	    (3) PrepareFeedFetch



**`BuildRuntimeProgram`函数:**

1. 新建scope的子节点`exe_scope`

```c++
 auto* exe_scope = &scope_->NewScope();
```

2. `scope_`初始化`feed`和`fetch`变量

```c++
  scope_->Var("feed")->GetMutable<std::vector<lite::Tensor>>();
  scope_->Var("fetch")->GetMutable<std::vector<lite::Tensor>>();
```

3. 将programDesc中的所有**中间变量**放入`exe_scope`中，将**持久化变量**放入`scope_`中

```c++
auto block_size = program_desc->BlocksSize();

for (size_t block_idx = 0; block_idx < block_size; ++block_idx) {
    auto block_desc = program_desc->GetBlock<cpp::BlockDesc>(block_idx);
    auto var_size = block_desc->VarsSize();
    
    for (size_t var_idx = 0; var_idx < var_size; ++var_idx) {
      auto var_desc = block_desc->GetVar<cpp::VarDesc>(var_idx);
        
      if (!var_desc->Persistable()) {
        auto* var = exe_scope->Var(var_desc->Name());
        ...
      } else {
        if (var_desc->Name() == "feed" || var_desc->Name() == "fetch") continue;
        scope_->Var(var_desc->Name());
      }
    }
    ...
  }
```

4. 初始化`RuntimeProgram`

```c++
 program_.reset(new RuntimeProgram(
      program_desc, exe_scope, kRootBlockIdx, use_low_precision_));
```





`LightPredictorImpl : PaddlePredictor`

> PaddlePredictor实现层

**成员变量：**

```c++
 private:
  std::unique_ptr<lite::LightPredictor> raw_predictor_;
```

### RuntimeProgram

**成员变量：**

```c++
std::vector<std::vector<Instruction>> instructions_; // Instruction由Oplite和KernelBase构造
Scope* exec_scope_{};
int64_t version_{0};
```



#### 构造函数

```c++
RuntimeProgram::RuntimeProgram(
    const std::shared_ptr<const cpp::ProgramDesc>& program_desc,// 模型静态图
    Scope* exec_scope, // 模型运行的中间变量
    int block_idx, // 运行中的block id
    bool use_precision_low);
```

**将block_desc中的OpDesc转换成对应的OpLite实例和KernelBase实例**

- 根据`block id`取出`progra_desc`中的`block_desc`，遍历`block_desc`中的所有`op_desc`
- 根据`op_desc`的type找到已注册的OpLite**新**实例`op`
- 当`op_desc`的类型是`while`、`conditional_block`和`subgraph`时，做。。。
- `op`绑定`op_desc`和`exec_scope`
- 如果`op_desc`的含有 kKernelTypeAttr 的 attr
  - 根据此信息是解析出kernel信息（kernel type、place、alias）
  - 从`op->CreateKernels({place})`实例化kernelBase
- 如果无 kKernelTypeAttr 的 attr
  - ARM 编译生成 ARM Kernel
  - X86 编译生成 X86 Kernel

**将 `OpLite` 实例和 `KernelBase` 实例放入`instruction_`**

#### Run函数

调用instruction->run()



#### cpp::OpDesc与Kernel关联

##### 注册LiteOP

[lite/core/op_registry.h:255](https://github.com/PaddlePaddle/Paddle-Lite/blob/f8656fd616c9e458c0b9df75804c316bcdfdaf58/lite/core/op_registry.h#L255)

```c++
// Register an op.
#define REGISTER_LITE_OP(op_type__, OpClass)                                   \
  static paddle::lite::OpLiteRegistrar op_type__##__registry(                  \
      #op_type__, []() {                                                       \
        return std::unique_ptr<paddle::lite::OpLite>(new OpClass(#op_type__)); \
      });                                                                      \
  int touch_op_##op_type__() {                                                 \
    op_type__##__registry.touch();                                             \
    OpKernelInfoCollector::Global().AddOp2path(#op_type__, __FILE__);          \
    return 0;                                                                  \
  }
```

如[新增OP](https://paddle-lite.readthedocs.io/zh/develop/develop_guides/add_operation.html)章节中介绍的Argmax的新增与注册

```c++
REGISTER_LITE_OP(arg_max, paddle::lite::operators::ArgmaxOpLite);
```



[创建Op并选择Kernel](https://github.com/PaddlePaddle/Paddle-Lite/blob/f8656fd616c9e458c0b9df75804c316bcdfdaf58/lite/core/program.cc#L300)



```c++
 // Create op
auto op = LiteOpRegistry::Global().Create(op_type);
```



### Instruction

**成员变量:**

```c++
  std::shared_ptr<OpLite> op_; 
  std::unique_ptr<KernelBase> kernel_;
  bool is_feed_fetch_op_{false};
  bool first_epoch_{true};
  bool has_run_{false};
```

#### Run函数

```c++
// 略Profile相关

if (first_epoch_) {
    first_epoch_ = false;
    CHECK(op_->CheckShape());
}
if (op_->run_once() && has_run_) {
    return;
}

op_->InferShape();
kernel_->Launch();
has_run_ = true;
```



# NNAdapter

> 飞桨推理 AI 硬件统一适配框架

## 概要

nnadapter实现推理框架与硬件的解耦

## 框架

### **标准化向上（推理框架）的接口**

### **标准化算子定义和Runtime组件**

中间表示层的算子定义，方便硬件厂商快速完成算子映射/转换

Runtime 作为 API 和 HAL 层的桥梁，其作用不仅是将 API 的调用翻译成模型、操作数、操作符的中间表达以及设备 HAL 层接口的调用，还包括设备 HAL 层库的注册、模型缓存的序列化和反序列化。

#### NNAdapter算子

```c++
typedef enum {
/**
* Performs element-wise binary addition(with Numpy-style broadcasting
* https://numpy.org/doc/stable/user/basics.broadcasting.html).
*
* Inputs:
* * 0: input0, A NNADAPTER_TENSOR_FLOAT32,
* NNADAPTER_TENSOR_QUANT_INT8_SYMM_PER_LAYER tensor.
* * 1: input1, A tensor with the same type as input0.
* * 2: fuse_code, A NNADAPTER_INT32 scalar, specifies the activation to the
* result, must be one of NNAdapterFuseCode values.
*
* Outputs:
* * 0: output, The result with the same type as two inputs.
*
* Available since version 1.
*/
NNADAPTER_ADD = 0,
……
} NNAdapterOperationCode;
```



### **标准化向下（硬件）抽象层（HAL）的接口定义**

实现对硬件设备的抽象和封装，屏蔽了硬件细节，为NNAdaper在不同硬件设备提供统一的访问接口

## 基于NNAdapter的硬件适配实践

### HAL标准接口定义

定义在[Paddle-Lite/lite/backends/nnadapter/nnadapter/include/nnadapter/core/types.h](https://github.com/PaddlePaddle/Paddle-Lite/blob/ede855cb5bf602cbfb3c4e5fb59997f78ec19b81/lite/backends/nnadapter/nnadapter/include/nnadapter/core/types.h)

```c++
typedef struct Operand {
  NNAdapterOperandType type;
  void* buffer;
  uint32_t length;
} Operand;

typedef struct Argument {
  int index;
  void* memory;
  void* (*access)(void* memory, NNAdapterOperandType* type);
} Argument;

typedef struct Operation {
  NNAdapterOperationType type;
  std::vector<Operand*> input_operands;
  std::vector<Operand*> output_operands;
} Operation;

typedef struct Cache {
  const char* token;
  const char* dir;
  std::vector<NNAdapterOperandType> input_types;
  std::vector<NNAdapterOperandType> output_types;
  std::vector<uint8_t> buffer;
} Cache;

typedef struct Model {
  std::list<Operand> operands;
  std::list<Operation> operations;
  std::vector<Operand*> input_operands;
  std::vector<Operand*> output_operands;
} Model;
```

硬件适配开发需要的实现接口,定义在[Paddle-Lite/lite/backends/nnadapter/nnadapter/include/nnadapter/driver/device.h](https://github.com/PaddlePaddle/Paddle-Lite/blob/ede855cb5bf602cbfb3c4e5fb59997f78ec19b81/lite/backends/nnadapter/nnadapter/include/nnadapter/driver/device.h)

```c++
typedef struct Device {
  // Properties
  const char* name;
  const char* vendor;
  NNAdapterDeviceType type;
  int32_t version;
  // Interfaces
  int (*open_device)(void** device);
  void (*close_device)(void* device);
  int (*create_context)(void* device, const char* properties, int (*callback)(int event_id,   void* user_data), void** context);
  void (*destroy_context)(void* context);
  int (*create_program)(void* context, Model* model, Cache* cache, void** program);
  void (*destroy_program)(void* program);
  int (*execute_program)(void* program, uint32_t input_count, Argument* input_arguments, uint32_t output_count, Argument* output_arguments);
} Device;
```

设备接口为 `Runtime` 在不同硬件提供统一的访问接口，需要对硬件的功能进行抽象和封装，涉及设备基本信息和标准功能接口：

```c++
NNADAPTER_EXPORT nnadapter::driver::Device NNADAPTER_AS_SYM2(DEVICE_NAME) = {
    .name = NNADAPTER_AS_STR2(DEVICE_NAME),
    .vendor = "Huawei",
    .type = NNADAPTER_ACCELERATOR,
    .version = 1,
    .open_device = nnadapter::huawei_ascend_npu::OpenDevice,
    .close_device = nnadapter::huawei_ascend_npu::CloseDevice,
    .create_context = nnadapter::huawei_ascend_npu::CreateContext,
    .destroy_context = nnadapter::huawei_ascend_npu::DestroyContext,
    .validate_program = 0,
    .create_program = nnadapter::huawei_ascend_npu::CreateProgram,
    .destroy_program = nnadapter::huawei_ascend_npu::DestroyProgram,
    .execute_program = nnadapter::huawei_ascend_npu::ExecuteProgram,
};
```

### `drive模板代码`

> 需要实现自定义的engine，命名空间和日志输出

```c++
// Copyright (c) 2019 PaddlePaddle Authors. All Rights Reserved.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

#include "driver/device.h"
#include "driver/xxx/engine.h"
#include "utility/logging.h"
#include "utility/micros.h"

namespace nnadapter {
namespace xxx {

int OpenDevice(void** device) {
  auto d = new Device();
  if (!d) {
    *device = nullptr;
    NNADAPTER_LOG(FATAL) << "Failed to open device for xxx.";
    return NNADAPTER_OUT_OF_MEMORY;
  }
  *device = reinterpret_cast<void*>(d);
  return NNADAPTER_NO_ERROR;
}

void CloseDevice(void* device) {
  if (device) {
    auto d = reinterpret_cast<Device*>(device);
    delete d;
  }
}

int CreateContext(void* device,
                  const char* properties,
                  int (*callback)(int event_id, void* user_data),
                  void** context) {
  if (!device || !context) {
    return NNADAPTER_INVALID_PARAMETER;
  }
  auto d = reinterpret_cast<Device*>(device);
  auto c = new Context(d, properties);
  if (!c) {
    *context = nullptr;
    NNADAPTER_LOG(FATAL) << "Failed to create context for xxx.";
    return NNADAPTER_OUT_OF_MEMORY;
  }
  *context = reinterpret_cast<void*>(c);
  return NNADAPTER_NO_ERROR;
}

void DestroyContext(void* context) {
  if (!context) {
    auto c = reinterpret_cast<Context*>(context);
    delete c;
  }
}

int CreateProgram(void* context,
                  core::Model* model,
                  core::Cache* cache,
                  void** program) {
  NNADAPTER_LOG(INFO) << "Create program for xxx.";
  if (!context || !(model || (cache && cache->buffer.size())) || !program) {
    return NNADAPTER_INVALID_PARAMETER;
  }
  *program = nullptr;
  auto c = reinterpret_cast<Context*>(context);
  auto p = new Program(c);
  if (!p) {
    return NNADAPTER_OUT_OF_MEMORY;
  }
  int result = p->Build(model, cache);
  if (result == NNADAPTER_NO_ERROR) {
    *program = reinterpret_cast<void*>(p);
  }
  return result;
}

void DestroyProgram(void* program) {
  if (program) {
    NNADAPTER_LOG(INFO) << "Destroy program for xxx.";
    auto p = reinterpret_cast<Program*>(program);
    delete p;
  }
}

int ExecuteProgram(void* program,
                   uint32_t input_count,
                   core::Argument* input_arguments,
                   uint32_t output_count,
                   core::Argument* output_arguments) {
  if (!program || !output_arguments || !output_count) {
    return NNADAPTER_INVALID_PARAMETER;
  }
  auto p = reinterpret_cast<Program*>(program);
  return p->Execute(
      input_count, input_arguments, output_count, output_arguments);
}

}  // namespace huawei_ascend_npu
}  // namespace nnadapter

NNADAPTER_EXPORT nnadapter::driver::Device NNADAPTER_AS_SYM2(DEVICE_NAME) = {
    .name = NNADAPTER_AS_STR2(DEVICE_NAME),
    .vendor = "xx"
    .type = NNADAPTER_ACCELERATOR,
    .version = 1,
    .open_device = nnadapter::xxx::OpenDevice,
    .close_device = nnadapter::xxx::CloseDevice,
    .create_context = nnadapter::xxx::CreateContext,
    .destroy_context = nnadapter::xxx::DestroyContext,
    .validate_program = 0,
    .create_program = nnadapter::xxx::CreateProgram,
    .destroy_program = nnadapter::xxx::DestroyProgram,
    .execute_program = nnadapter::xxx::ExecuteProgram,
};
```



### `engine模板代码`

定义在[Paddle-Lite/lite/backends/nnadapter/nnadapter/src/driver/XXX/engine.h]()

```c++
namespace nnadapter {
namespace XXX { // 适配硬件的命名空间
    class Device {
        public:
        Device();
        ~Device();
    };
    
    class Context {
     public:
      explicit Context(void* device, const char* properties)
      ~Context();

     private:
      void* device_{nullptr};

    };

    class Program {
     public:
      explicit Program(Context* context) : context_(context) {}
      ~Program();

      int Build(core::Model* model, core::Cache* cache);
      int Execute(uint32_t input_count,
                  core::Argument* input_arguments,
                  uint32_t output_count,
                  core::Argument* output_arguments);

     private:
      void Clear();
      int CheckInputsAndOutputs(uint32_t input_count,
                                core::Argument* input_arguments,
                                uint32_t output_count,
                                core::Argument* output_arguments);

     private:
      Context* context_{nullptr};
      // Map NNAdapter operand to XXX operator
      // std::map<core::Operand*, std::vector<std::shared_ptr<Operator>>> operators_;

      std::vector<NNAdapterOperandType> input_types_;
      std::vector<NNAdapterOperandType> output_types_;
    };

}    
}
```

#### `Device:: Device()`

> 仅实例化设备一次

```c++
Device::Device() { InitializeDevice(); }
```



#### `Context::Context(void* device, const char* properties)`

> 参数解析，根据参数设置设备上下文

device context是“设备运行环境”

```c++
Context::Context(void* device, const char* properties) : device_(device) {
  // Extract the runtime parameters from the context properties
  NNADAPTER_LOG(INFO) << "properties: " << std::string(properties);
  auto key_values = GetKeyValues(properties);
  
  //......
}
```

`GetKeyValues`

```c++
// Parse and get the key value map from a string
std::map<std::string, std::string> GetKeyValues(
    const char* properties,
    const std::string& delimiter = ";",
    const std::string& assignment = "=");
```

[google_xnnpack](https://github.com/PaddlePaddle/Paddle-Lite/tree/develop/lite/backends/nnadapter/nnadapter/src/driver/google_xnnpack)例子

```c++
Context::Context(void* device, const char* properties) : device_(device) {
  // Extract the runtime parameters from the context properties
  NNADAPTER_LOG(INFO) << "properties: " << std::string(properties);
  std::string key_value;
  auto key_values = GetKeyValues(properties);
  // GOOGLE_XNNPACK_NUM_THREADS
  if (key_values.count(GOOGLE_XNNPACK_NUM_THREADS)) {
    num_threads_ = string_parse<int>(key_values[GOOGLE_XNNPACK_NUM_THREADS]);
  } else {
    num_threads_ = GetIntFromEnv(GOOGLE_XNNPACK_NUM_THREADS, 0);
  }
  NNADAPTER_LOG(INFO) << "num_threads: " << num_threads_;
  if (num_threads_ > 1) {
    threadpool_ = pthreadpool_create(num_threads_);
    NNADAPTER_CHECK(threadpool_ != nullptr)
        << "Failed to create a thread pool for XNNPACK library!";
  }
}

Context::~Context() {
  if (threadpool_) {
    pthreadpool_destroy(threadpool_);
    threadpool_ = nullptr;
  }
}
```

#### `Program::Build(core::Model* model, core::Cache* cache)`

> 模型编译，将NNadper中间模型实例编译成设备程序

```c++
int Program::Build(core::Model* model, core::Cache* cache) {
  Clear();
  bool model_from_cache = false;
  if (!cache->buffer.empty()) {
    // Build from cache
    auto input_count = cache->input_types.size();
    NNADAPTER_VLOG(3) << "Model input count: " << input_count;
    input_types_ = cache->input_types;
    auto output_count = cache->output_types.size();
    NNADAPTER_VLOG(3) << "Model output count: " << output_count;
    NNADAPTER_CHECK_GT(output_count, 0);
    output_types_ = cache->output_types;
    NNADAPTER_CHECK(!model);
    if (!DeserializeModel(cache->buffer.data(), cache->buffer.size(), &model)) {
      NNADAPTER_LOG(FATAL)
          << "Failed to deserialize the optimized core::Model from a buffer!";
    } else {
      model_from_cache = true;
      NNADAPTER_VLOG(3)
          << "Deserialize the optimized core::Model from a buffer success.";
    }
    NNADAPTER_VLOG(5) << "Cached model:" << std::endl << Visualize(model);
  } else {
    // Build from model
    // Convert the data layout and the quantization parameters of the NNAdapter Model

  }
  
  return NNADAPTER_NO_ERROR;
}
```

