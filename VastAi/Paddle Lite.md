# Paddle Lite

## 概要

paddle lite是一组工具，以实现在多端设备（移动端设备、物联网设备等）上运行机器学习模型

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

