# 预备知识

## 计算图

计算图是一种用来**描述运算的有向无环图（DAG）。结点表示数据或运算，边表示数据流向。**

### 动态图

计算图的构建与运算是同时发生的（define by run）。在系统中，不需要先描述整个计算图的结构，再提供计算数据，而是一开始就计算数据进行相关操作。

**优点：**

-   模型的构建与调试方便
    

**缺点：**

-   缺少对模型整体结构的认知，无法做一些优化
    

### 静态图

计算图的构建与运算是分开发生的（define and run）。先构建好运算流程（计算图），在下次运算时就不需要再次构建图。

**优点：**

-   对图进行等价变化，做优化处理
    

**缺点：**

-   优化后的图可能导致中间结果不复存在，不便于调试
    

## 中间表达

中间表示（IR）是从源到目标的转换过程中一个中间形式

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2JkMzBkZGM0NWM4YTIzNDg2NDZhMjA2M2RhN2NiNjFfT1JzeFpEN1pVQkFreFBTNnJWTVY3OXVzb3ZncVJDYWVfVG9rZW46Ym94Y25xVkRqZWk2cTBwMzVjd1MxTUx6cEIyXzE2NzgwOTYzMTY6MTY3ODA5OTkxNl9WNA)![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=NzEyNmRlOGM1YTUxNTVkMDgwYzUyYzJlOTAwYmQwYzRfN2hhSVNUbncxVVdSckJHdGt4VkR0bHpQSExQR2tISTdfVG9rZW46Ym94Y24xWWkzMWdXQUlvbWxBTGJBcU1QNTNmXzE2NzgwOTYzMTY6MTY3ODA5OTkxNl9WNA)

设计一个跨多平台多门语言集合的编译器的复杂度是$$$$N\times M $$$$，而有了中间表示的复杂度是$$$$N+M $$$$

  

### 特点

1.  降低前端语言与后端对接的复杂度
    

  

**抽象层次**

中间表达一般采用三层抽象

### 中间表达类型

1.  **图IR**
    
2.  线性IR
    
3.  混合IR
    

  

  

# 基础实践

paddle lite是一组工具，以实现在多端设备（移动端设备、物联网设备等）上运行机器学习模型

## docker环境准备

```Bash
#从 Dockerhub 直接拉取 Docker 镜像
docker pull paddlepaddle/        paddle-lite:2.0.0_beta

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

#./lite/tools/build_linux.sh --arch=x86 --with_extra=ON --with_log=ON --with_exception=ON --with_nnadapter=ON --nnadapter_with_vastai=ON --nnadapter_vastai_sdk_root=./vastai_sdk
```

## 基础编译参数

Paddle Lite 仓库中`./lite/tools/build_linux.sh`脚本文件用于构建 linux 版本的编译包，通过修改`build_linux.sh`脚本文件中的参数，可满足不同场景编译包的构建需求，常用的基础编译参数如下表所示： 有特殊硬件需求的编译参数见后文。

## 开发流程

### 生成paddle lite模型

使用paddle lite opt工具将paddle模型转换成paddle lite模型

```Bash
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

```C++
#include <paddle_api.h>
class CxxConfig;
```

`CxxConfig` 用来配置构建 Full feature predictor 的配置信息，如 protobuf 格式的模型地址、能耗模式、工作线程数、place 信息等等

设置/获取模型文件夹，**加载非combined模型**

```C++
void set_model_dir(const std::string& x);  

const std::string& model_dir();
```

设置/获取模型文件，**加载combined模型**

```C++
void set_model_file(const std::string& path);

std::string model_file();
```

##### `MobileConfig`

```C++
#include <paddle_api.h>
class MobileConfig;
```

`MobileConfig` 用来配置构建light weight predictor 的配置信息，如 NaiveBuffer 格式的模型地址、模型的内存地址（从内存加载模型时使用）、能耗模式、工作线程数等等。

> 注意：输入的模型需要使用 [Model Optimize Tool](https://paddle-lite.readthedocs.io/zh/develop/user_guides/model_optimize_tool.html)转化为 NaiveBuffer 格式的优化模型。

设置模型文件

```C++
void set_model_from_file(const std::string& x);
```

#### 2.创建Predictor

```C++
#include <paddle_api.h>

template <typename ConfigT>
std::shared_ptr<PaddlePredictor> CreatePaddlePredictor(const ConfigT&);
```

`CreatePaddlePredictor` 用来根据 `Config` 构建预测器`PaddlePredictor`对象。

#### 3.执行Predictor

```C++
#include <paddle_api.h>

class PaddlePredictor;
```

`PaddlePredictor` 是 Paddle Lite 的预测器，由 `CreatePaddlePredictor` 根据 `Config` 进行创建。用户可以根据 PaddPredictor 提供的接口设置**输入数据、执行模型预测、获取输出**以及获得当前使用 lib 的版本信息等。

**1）.通过PaddlePredictor的GetInput接口获取输入的Tensor**

```C++
std::unique_ptr<Tensor> input_tensor(std::move(predictor->GetIntput(x)));
```

**2）.调整Tensor的维度以适应输入数据的维度**

```C++
void Tensor::Resize(const shape_t &shape);
input_tensor->Resize({1, 2, 3, 4});
```

**3）.向Tensor中填充输入数据**

获取Tensor底层数据的常量指针（n维的shape，底层使用一维数组），根据传入的不同模型类型获取相应数据。

```C++
template <typename T> const T* data() const;
```

设置Tensor底层数据

```C++
template <typename T> T* mutable_data() const;

auto* data = input_tensor->mutable_data<float>();
// 设置 Tensor 数据
for (int i = 0; i < ShapeProduction(input_tensor->shape()); ++i) {
  data[i] = 1;
}
```

**4）.运行predictor**

```C++
predictor->Run();
```

**5）.获取predictor的输出结果**

通过PaddlePredicator的GetOutput接口获取输出Tensor

```C++
std::unique_ptr<Tensor> out_tensor(std::move(predictor->GetOutput(x)));
```

获取输出Tensor底层数据的常量指针

```C++
auto* output = output_tensor->data<float>();
```

# 架构设计

重点在于**多硬件**、**高性能**的支持，主要设计思想如下：

-   引入**类型系统**，强化多硬件、量化方法、data layout 的混合调度能力
    
-   硬件细节隔离，通过不同编译开关，对支持的任何硬件可以自由插拔
    
-   引入 MIR(Machine IR) 的概念，强化带执行环境下的优化支持
    
-   优化期和执行期严格隔离，保证预测时轻量和高效率
    

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2Q0Y2MzNGI2NTllN2I4OTBlOWI3ODdjODdiMTE0MDBfaG9xakpIeTM5UnpKbzA0NmlvOEFSUnRCQzhrMFdLdWVfVG9rZW46Ym94Y25veTBOQWZSWGViOEFlYlhHWFVuSFpkXzE2NzgwOTYzMTY6MTY3ODA5OTkxNl9WNA)

芯片种类繁多（ARM、X86、GPU、FPGA等），AI框架的主要技术特点是：

-   **多硬件支持**
    
-   **高性能**
    

  

## 类型系统

> 强化多硬件、量化方法、data layout 的混合调度能力

Lite框架使用`Type`类定义框架下所有类型——算子或Kernel的输入和输出。

简单来说，Lite框架的类型是一个五元组$$$$(DataType, Target, Precision, DataLayout, Device) $$$$

-   `DataType`：算子或Kernel输入输出的数据载体，可以是`void`表示为止类型、`Tensor`表示Tensor类型、`Unsupported`表示不支持的类型等
    
-   `Target`
    
-   `Precisioin`：表示数据的精准度，如float, double等
    
-   `DataLaylout`：表示数据布局，如NCHW等
    
-   `Device`
    

不同类型之间可以通过一些特殊的算子转换，比如_DataLayoutTransformOp_能够将_`TensorFp32NCHWTy`_ _to a_ _`TensorFp32NHWCTy`__。_

不同的类型之间也可是是兼容的。比如`VoidTy`代表任意类型，所以任何类型都与`VoidTy`兼容。

###   Lite框架的Type类

1.  ####     DataTye类
    

    `Void`数据类型代表任意类型

    `Unsupported`：没有被注册的类型都会标记为`Unsupported`，它不会被系统处理，以避免未定义的行为或bug

```C++
class DataType {
 public:
  enum class ID : int {
    Void = 0,     // unknown type that can be cast to any data type.
    Unsupported,  // Unsupported data type that will not be analyzed.
    // Tensor_Any represents a Tensor with any place, data, layout. It is used
    // in some IO kernels those doesn't care the data.
    Tensor,
    // A tensor list, but all the elements should have the same type.
    TensorList,
    // A vector of local scope, which size equals the step number of While Op.
    // The i'th scope storages temporary variables generated in the i'th step.
    StepScope,
    // ---------
    NumTypes,  // Must remains as last defined ID.
  };

  ID id() const { return id_; }
  // type check.
  bool IsVoid() const { return id_ == ID::Void; }
  bool IsUnsupported() const { return id_ == ID::Unsupported; }
  bool IsTensor() const { return id_ == ID::Tensor; }
  bool IsTensorList() const { return id_ == ID::TensorList; }
  bool IsStepScope() const { return id_ == ID::StepScope; }
  // Get number of types.
  int num_types() const { return static_cast<int>(ID::NumTypes); }
 protected:
  // Can only extended by subclass.
  explicit DataType(ID id) : id_(id) {}
  ID id_{ID::Unsupported};
};
```

2.  ####     Type类
    

    `GetTensorTy`、`GetTensorListTy`、`GetStepScopeTy`等功能是：根据所给的Place生成特定数据类型的Type

```C++
class Type : public DataType {
 public:
  // Can cast to another type. This is heavily used in MIR, by determine whether
  // is possible to add a statement to transform a type to another.
  virtual bool TypeCastable(const Type& type) const { return id_ == type.id(); }

  /// Get a Tensor type.
  static const Type* GetTensorTy(TargetType target,
                                 PrecisionType precision = PRECISION(kFloat),
                                 DataLayoutType layout = DATALAYOUT(kNCHW),
                                 int device = 0);
  /// Get a TensorList type.
  static const Type* GetTensorListTy(
      TargetType target,
      PrecisionType precision = PRECISION(kFloat),
      DataLayoutType layout = DATALAYOUT(kNCHW),
      int device = 0);
  /// Get a StepScope type.
  static const Type* GetStepScopeTy();
  /// Get an Unsupported type.
  static const Type* GetUnsupportedTy();
  /// Get an Void type.
  static const Type* GetVoidTy();

  static const Type* Get(DataType::ID type_id,
                         TargetType target = TARGET(kUnk),
                         PrecisionType precision = PRECISION(kUnk),
                         DataLayoutType layout = DATALAYOUT(kUnk),
                         int device = 0);

  TargetType target() const { return place_.target; }
  PrecisionType precision() const { return place_.precision; }
  DataLayoutType layout() const { return place_.layout; }
  int16_t device() const { return place().device; }
  const Place& place() const { return place_; }
  const std::string& name() const { return name_; }

  bool operator==(const Type& other) {
    return id_ == other.id() && place_ == other.place();
  }
  friend STL::ostream& operator<<(STL::ostream& os, const Type& other);

  virtual ~Type() = default;

 protected:
  /// One should avoid using this construct.
  Type(ID id,
       const std::string& name,
       TargetType target = TargetType::kHost,
       PrecisionType precision = PrecisionType::kFloat,
       DataLayoutType layout = DataLayoutType::kNCHW,
       int16_t device = 0)
      : DataType(id), place_{target, precision, layout, device}, name_(name) {}

  Place place_;
  const std::string name_;
};
```

3.  ####     兼容性检查
    

    检查五元组之间的兼容性，跟领域相关略

```C++
static bool TargetCompatibleTo(const Type& a, const Type& b) {
  auto is_host = [](TargetType x) -> bool {
    return x == TARGET(kHost) || x == TARGET(kX86) || x == TARGET(kARM) ||
           x == TARGET(kAny);
  };
  if (a.IsTensor() || b.IsTensor() || a.IsTensorList() || b.IsTensorList()) {
    return is_host(a.target()) ? is_host(b.target()) : a.target() == b.target();
  }
  return true;
}

static bool DataLayoutCompatibleTo(const Type& a, const Type& b) {
  return a.IsVoid() ||                 //
         (a.layout() == b.layout() ||  //
          ((b.layout() == DATALAYOUT(kAny)) &&
           (a.layout() != DATALAYOUT(kImageDefault) &&
            a.layout() != DATALAYOUT(kImageFolder))));
}
static bool DataLayoutCompatible(const Type& a, const Type& b) {
  return a.IsVoid() || b.IsVoid() ||   //
         (a.layout() == b.layout() ||  //
          ((b.layout() == DATALAYOUT(kAny)) &&
           (a.layout() != DATALAYOUT(kImageDefault) &&
            a.layout() != DATALAYOUT(kImageFolder))) ||
          ((a.layout() == DATALAYOUT(kAny)) &&
           (b.layout() != DATALAYOUT(kImageDefault) &&
            b.layout() != DATALAYOUT(kImageFolder))));
}

static bool PrecisionCompatibleTo(const Type& a, const Type& b) {
  return a.IsVoid() ||  //
         (((a.IsTensor() && b.IsTensor()) ||
           (a.IsTensorList() && b.IsTensor()) ||
           (a.IsTensor() && b.IsTensorList()) ||
           (a.IsTensorList() && b.IsTensorList())) &&
          (a.precision() == b.precision() ||  //
           b.precision() == PRECISION(kAny) ||
           a.precision() == PRECISION(kAny)));
}
static bool PrecisionCompatible(const Type& a, const Type& b) {
  return a.IsVoid() || b.IsVoid() ||  //
         (((a.IsTensor() && b.IsTensor()) ||
           (a.IsTensorList() && b.IsTensorList())) &&
          (a.precision() == b.precision() ||  //
           b.precision() == PRECISION(kAny) ||
           a.precision() == PRECISION(kAny)));
}

static bool DeviceCompatibleTo(const Type& a, const Type& b) {
  return a.IsVoid() ||  //
         (((a.IsTensor() && b.IsTensor()) ||
           (a.IsTensorList() && b.IsTensorList())) &&  //
          (a.device() == b.device()));
}

// Can type 'a' be passed to 'b' directly.
static bool TypeCompatibleTo(const Type& a, const Type& b) {
  return TargetCompatibleTo(a, b) && DataLayoutCompatibleTo(a, b) &&
         PrecisionCompatibleTo(a, b) && DeviceCompatibleTo(a, b);
}
static bool TypeCompatible(const Type& a, const Type& b) {
  return TargetCompatibleTo(a, b) && DataLayoutCompatible(a, b) &&
         PrecisionCompatible(a, b) && DeviceCompatibleTo(a, b);
}
```

4.  ####     Kernel参数类型
    

  

## 技术特点

### 多硬件支持

Paddle Lite是移动端框架，移动端设备多元化

多种硬件的 Kernel 在代码层和执行层均互不干扰，用户可以自由插拔任何硬件的支持。

### 高性能

### 量化支持

Lite 支持Paddle Slim 强大的量化训练完毕的模型，因此完整保留了量化计算的高性能以及量化训练的高精度。

### 图分析与优化

Paddle Lite架构包含**IR**及**Pass集合**以实现计算图优化，包括算子融合、计算剪枝、存储优化、量化计算等

### 轻量级部署

Lite框架支持图分析及优化阶段和执行阶段分离，实现移动端轻量级部署

### 可支持任意硬件的混合调度

Lite架构通过从底层支持 `Type system` 的方式通用建模各类混合执行的行为，从而能够相对完备地支持混调。

## 架构设计

### 分析阶段与执行阶段隔离

保证预测时（推理）轻量和高效率

#### 分析阶段

**载入模型并构建计算图：**

-   载入Paddle模型（**第一层IR**）
    
-   按照如下规则将每个Block生成计算图（**第二层IR**）：_每个算子或变量都对应计算图的一个节点，节点间的有向边由算子的输入、输出决定（依赖关系确定边的方向），算子节点与变量节点相邻。_
    

**计算图分析与优化：**

将一系列 pass （优化器，用于描述一个计算图变换得到另一个计算图的处理过程）按照一定的顺序依次应用到每个块对应的计算图的过程，包括**量化信息处理**、算子融合、 **Kernel 选择**、类型转化、**上下文创建**、**内存复用优化**和**子图检测**等，实现不同设备的适配、高效的计算和更少的内存占用。

#### 执行阶段

  

# 源码分析

## 数据生命周期

### 分析阶段

> 从NB文件中反序列化出模型结构program_desc并在scope中初始化变量，包括权重

**数据表示形式：**

-   program_desc
    
-   scope
    

program_desc存储模型结构、变量和参数配置等描述内容

scope中记录变量名到实际变量的映射

### 执行阶段

> 从main_block开始，根据opdesc的type找到注册的**OpLite**并找到最佳的**Kernel**

**上层的接口调用顺序：**

-   仅第一个epoch调用`OpLite::CheckShape()`
    
-   调用`OpLite::InferShape`推理输出的Tensor shape
    
-   调用`kernel::Launch`——为输出分配内存及计算输出结果
    

**数据接收方式：**

-   `OpLite::Attach`接口绑定op_desc和scope
    

**数据表示形式：**

-   `Op Param`
    

Param中是从op_desc和scope解析出的信息等

#### SubgraphOp

**数据表示的形式：** `SubgraphParam`

```C++
struct SubgraphParam : ParamBase {
  std::vector<std::string> input_names{};
  std::vector<std::string> output_names{};
  std::vector<std::string> input_data_names{};
  std::vector<std::string> output_data_names{};
  std::vector<float> input_data_scales{};
  std::vector<float> output_data_scales{};
  int block_idx{-1};
  std::shared_ptr<const cpp::ProgramDesc> program_desc{nullptr};
  Scope* exec_scope{nullptr};
};
```

#### SubgraphCompute

**功能：**

-   将Paddle子图转化成NNAdapter的中间表达
    
-   编译设备程序
    
-   执行设备程序
    

**数据表现形式**

```C++
typedef struct {
  std::string name;
  Tensor* value{nullptr};
  std::vector<std::vector<int64_t>> dynamic_dimensions{};
  float quant_scale{-1.0f};
  int32_t quant_zero_point{0};
} Variable;

// Engine
std::vector<Variable> input_vars_;
std::vector<Variable> output_vars_;
```

通过SubgraphCompute中的Program在执行前会调用`SetInputsAndOutputs(std::vector<Variable>* input_vars, std::vector<Variable>* output_vars)`方法，内部的核心代码实现如下：

```C++
bool Program::SetInputsAndOutputs(std::vector<Variable>* input_vars,
                                  std::vector<Variable>* output_vars) {
       ...
     
      for (uint32_t i = 0; i < input_count; i++) {
        NNAdapterExecution_setInput_invoke(
            execution_, 
            i, // index
            reinterpret_cast<void*>(input_vars->at(i).value),// memory
            AccessModelInput); // access
      }
      for (uint32_t i = 0; i < output_count; i++) {
        NNAdapterExecution_setOutput_invoke(
            execution_,
            i,
            reinterpret_cast<void*>(output_vars->at(i).value),
            AccessModelOutput);
      }
      return true;
}
```

**属性支持的类型与变量支持的类型:**

> 属性不支持Tensor

算子**属性值**存放在`program_desc`中，用Any类型存储。支持的类型只有数组和几个基本类型

```C++
enum class OpAttrType {
  INT = 0,
  FLOAT = 1,
  STRING = 2,
  INTS = 3,
  FLOATS = 4,
  STRINGS = 5,
  BOOLEAN = 6,
  BOOLEANS = 7,
  BLOCK = 8,
  LONG = 9,
  BLOCKS = 10,
  LONGS = 11,
  UNK,
};
```

**变量值**存放`Scope`，**变量描述**存放在`program_desc`。在支持的类型，同时支持变量的shape

```C++
enum class VarDataType {
  // Pod Types
  BOOL = 0,
  INT16,
  INT32,
  INT64,
  FP16,
  FP32,
  FP64,
  // Tensor<size_t> is used in C++.
  SIZE_T,
  UINT8,
  INT8,

  // Other types that may need additional descriptions
  LOD_TENSOR,
  SELECTED_ROWS,
  FEED_MINIBATCH,
  FETCH_LIST,
  STEP_SCOPES,
  LOD_RANK_TABLE,
  LOD_TENSOR_ARRAY,
  PLACE_LIST,
  READER,
  // Any runtime decided variable type is raw
  // raw variables should manage their own allocations
  // in operators like nccl_op
  RAW,
  TUPLE
};
```

### 补充

**持久化参数保存NB:**

-   预测层调用`SaveModelNaive`
    
-   取出**main_block中满足****`IsParamVarDesc`****的变量**，其实现如下：
    
    ```C++
    inline bool IsParamVarDesc(const VarDescReadAPI& var) {
      return var.GetType() == VarDataType::LOD_TENSOR && var.Persistable();
    }
    ```
    

## 基本数据类型

### Scope

> Scope是一个树形结构，树中每个节点都有一个map记录`变量名->Variable`的映射。

**API：**

```C++
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

-   `scope_`是根节点的**Scope**，用于存放持久化变量，包括`feed`变量和`fetch`变量。以及模型的权重参数等
    
-   `exec_scope`是`scope_`的子节点，存放模型计算的中间变量
    

### Variable

> 底层数据是`Any`类型，`Variable`能以任意类型的方式读取变量和修改变量

> [Any](https://dmlc-core.readthedocs.io/en/latest/doxygen/any_8h_source.html)是dml-core中的实现，dml-core是构建机器学习的一组工具包

> `Any`是一个**容器可以存放任意的数据类型**

**API:**

```C++
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

### Tensor

> 底层是一个`buffer`（一维数组）

> `dims_`记录Tensor的维度，`Resize`接口只调整`dims_`的值

> `Tensor`可以在任意设备端存在，比如host、ARM、X86、CUDA等。因此`Tensor`用`TargetType`记录其工作的设备端，根据`TargetType`找到内存分配的函数模板实例

**提供三种访问数据的方式：**

-   `data`接口: 返回**指定类型**的**常量指针**
    
    ```C++
    // 根据指定类型将buffer中存储的void类型data强转返回
    template <typename T, typename R = T>
    const R *data() const{
       return reinterpret_cast<const R *>(static_cast<char *>(buffer_->data()) +
                                          offset_);
    }
    ```
    
      没有`PrecisionType`的校验
    
-   `mutable_data`接口：**重定buffer**，返回**指定类型**指针。**引起内存分配**
    

```C++
// 内存分配大小取决于dims_.production() * sizeof(T)
template <typename T, typename R = T>
R *mutable_data() {
    // type_trait得到T的类型
    precision_ = lite_api::PrecisionTypeTrait<T>::Type(); 
    memory_size_ = dims_.production() * sizeof(T);
    // 重定buffer的值
    buffer_->ResetLazy(target_, memory_size_);
    return reinterpret_cast<R *>(static_cast<char *>(buffer_->data()) +
                                 offset_);
}

template <typename T, typename R = T>
R *mutable_data(TargetType target)

// 内存分配大小取决于memory_size
template <typename T, typename R = T>
R *mutable_data(TargetType target, size_t memory_size)

void *mutable_data(size_t memory_size);
void *mutable_data(TargetType target, size_t memory_size);
```

-   `raw_data`接口：直接返回buffer中的data数据
    

```C++
void *raw_data() {
    return static_cast<char *>(
        (static_cast<char *>(buffer_->data()) + offset_));
}
const void *raw_data() const
```

### Buffer

> 不同设备端的内存管理，包括但不限于主机端、X86、ARM、CUDA

为了统一不同端分配内存和释放内存的接口，Paddle-Lite提供了`TargetMalloc/TargetFree`接口

```C++
LITE_API void* TargetMalloc(TargetType target, size_t size);

void LITE_API TargetFree(TargetType target,
                         void* data,
                         std::string free_flag = "");
```

针对不同的`target`实现不同的`malloc/free`，很容易想到**模板特化**。**Paddle Lite**也是如此

```C++
template <TargetType Target, typename StreamTy = int, typename EventTy = int>
class TargetWrapper{ 略 }


// 针对host端的实现
template <>
class TargetWrapper<TARGET(kHost)> {
  // 没看懂
  static void* Malloc(size_t size);
  static void Free(void* ptr);
}
```

**重定Buffer:**

当target发生变化或内存不足，则释放原有数据，重新申请

```C++
virtual void ResetLazy(TargetType target, size_t size) {
    if (target != target_ || space_ < size) {
      CHECK_EQ(own_data_, true) << "Can not reset unowned buffer.";
      Free();
      data_ = TargetMalloc(target, size);
      target_ = target;
      space_ = size;
#ifdef LITE_WITH_OPENCL
      cl_use_image2d_ = false;
#endif
#ifdef LITE_WITH_METAL
      metal_use_image2d_ = false;
#endif
    }
  }
```

## 技术关键

  Lite框架里的 Op 与 Kernel 的设计采用了“**桥接模式**”。抽象部分是 OpLite 、具体部分是 KernelLite 。

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmM3NGE0Yzg5YjAwNzg4YWYyZmUxM2Y4NGMyMDBhNzZfeXQya0ZnTlJDTWU4QzA3ajlhRWJERkRPOVJDbnFacG5fVG9rZW46Ym94Y25yNU90dlpFckVmQ0VrOG8xUXEzb2xnXzE2NzgwOTYzMTY6MTY3ODA5OTkxNl9WNA)

1.  ###   Op
    

  设计理念

-   持有算子参数和一些计算资源：`op_info`和`scope`
    
-   应该类似于函数调用，不应该拥有太多的：
    

2.  ###   Kernel
    

-   统一不同平台的Kernel实现
    

## 网络拓扑描述（IR）

Lite框架到目前为止支持2种模型格式ProtoBuffer和NaiveBuffer。第一层IR将这种模型格式映射成统一的ProgramDesc

1.  ###   ProgramDesc
    

一个program由若干个block组成

2.  ###   BlockDesc
    

  block由若干个op和var组成

  默认main block（也叫boot block）id是0

3.  ###   OpDesc
    
        Op由输入、输出、类型、属性、属性类型组成。
    
        Op的输入和输出都是key-value格式的map，Op不同的实现会绑定到不同的key，如`feed_op`输入key是`X`，输出key是`Out`
    
    ```C++
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
    
    ####     OpDesc属性类型的Type Traits
    
        定义在_/lite/core/model/base/traits.h_
    
        **算子属性的类型**
    
    ```C++
    enum class OpAttrType {
      INT = 0,
      FLOAT = 1,
      STRING = 2,
      INTS = 3,
      FLOATS = 4,
      STRINGS = 5,
      BOOLEAN = 6,
      BOOLEANS = 7,
      BLOCK = 8,
      LONG = 9,
      BLOCKS = 10,
      LONGS = 11,
      UNK,
    };
    ```
    
        **Type Trait模板**
    
    >     模板参数U的意义？？？？？
    
    ```C++
    struct Standard {};
    template <typename T, typename U = Standard>
    struct OpDataTypeTrait;
    ```
    
        **Type Trait的模板“偏”特化**
    
        _INT、STRING类型的手写模板特化_
    
    ```C++
    template<typename U>
    struct OpDataTypeTrait<int32_t, U> {                  
        typedef int32_t ET;                                 
        typedef int32_t RT;                                 
        static constexpr OpAttrType AT{OpAttrType::INT};     
        static constexpr const char* ATN{"INT"};              
     }; 
    
    template<typename U>
    struct OpDataTypeTrait<std::string, U> {                  
        typedef std::string ET;                                 
        typedef std::string RT;                                 
        static constexpr OpAttrType AT{OpAttrType::STRING};     
        static constexpr const char* ATN{"STRING"};              
     }; 
    ```
    
        **宏定义简化模板偏特化**
    
    ```C++
    template <typename T, typename U = Standard>
    struct OpDataTypeTrait;
    
    #define ATTR_TYPE_TRAIT_IMPL(T, type__)                \
      template <typename U>                                \
      struct OpDataTypeTrait<type__, U> {                  \
        typedef type__ ET;                                 \
        typedef type__ RT;                                 \
        static constexpr OpAttrType AT{OpAttrType::T};     \
        static constexpr const char* ATN{#T};              \
      };                                                   \
      template <typename U>                                \
      constexpr OpAttrType OpDataTypeTrait<type__, U>::AT; \
      template <typename U>                                \
      constexpr const char* OpDataTypeTrait<type__, U>::ATN;
    ```
    

4.  ###   VarDesc
    

  var由名称、类型、是否可持久化以及shape组成

  存储block中所有变量的描述，包括中间变量

## SSA Graph（MIR）

  

## PaddlePredictor

#### Lite API

> [paddle::lite_api::paddle_api.h](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/api/paddle_api.h)定义了PaddlePredictor的Lite接口，支持多种硬件包括ARM、X86、OpenCL、CUDA等

##### Tensor

##### PaddlePredictor

`CreatePaddlePredictor` 根据 `Config` 构建预测器。

> CreatePaddlePredictor有两个特化版本，根据不同的Config实例化不同的Predictor

-   CxxConfig 实例化 CxxPredictor 定义在
    
-   MobileConfig 实例化 LightPredictor
    

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

```C++
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

```C++
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

（2）BuildRuntimeProgram

(3) PrepareFeedFetch

**`BuildRuntimeProgram`****函数:**

1.  新建scope的子节点`exe_scope`
    

```C++
 auto* exe_scope = &scope_->NewScope();
```

1.  `scope_`初始化`feed`和`fetch`变量
    

```C++
  scope_->Var("feed")->GetMutable<std::vector<lite::Tensor>>();
  scope_->Var("fetch")->GetMutable<std::vector<lite::Tensor>>();
```

1.  将programDesc中的所有**中间变量**放入`exe_scope`中，将**持久化变量**放入`scope_`中
    

```C++
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

1.  初始化`RuntimeProgram`
    

```C++
 program_.reset(new RuntimeProgram(
      program_desc, exe_scope, kRootBlockIdx, use_low_precision_));
```

`LightPredictorImpl : PaddlePredictor`

> PaddlePredictor实现层

**成员变量：**

```C++
 private:
  std::unique_ptr<lite::LightPredictor> raw_predictor_;
```

### RuntimeProgram

**成员变量：**

```C++
std::vector<std::vector<Instruction>> instructions_; // Instruction由Oplite和KernelBase构造
Scope* exec_scope_{};
int64_t version_{0};
```

#### 构造函数

```C++
RuntimeProgram::RuntimeProgram(
    const std::shared_ptr<const cpp::ProgramDesc>& program_desc,// 模型静态图
    Scope* exec_scope, // 模型运行的中间变量
    int block_idx, // 运行中的block id
    bool use_precision_low);
```

**将block_desc中的OpDesc转换成对应的OpLite实例和KernelBase实例**

-   根据`block id`取出`progra_desc`中的`block_desc`，遍历`block_desc`中的所有`op_desc`
    
-   根据`op_desc`的type找到已注册的OpLite**新**实例`op`
    
-   当`op_desc`的类型是`while`、`conditional_block`和`subgraph`时，做。。。
    
-   `op`绑定`op_desc`和`exec_scope`
    
-   如果`op_desc`的含有 kKernelTypeAttr 的 attr
    
    -   根据此信息是解析出kernel信息（kernel type、place、alias）
        
    -   从`op->CreateKernels({place})`实例化kernelBase
        
-   如果无 kKernelTypeAttr 的 attr
    
    -   ARM 编译生成 ARM Kernel
        
    -   X86 编译生成 X86 Kernel
        

**将** **`OpLite`** **实例和** **`KernelBase`** **实例放入****`instruction_`**

#### Run函数

调用instruction->run()

#### cpp::OpDesc与Kernel关联

##### 注册LiteOP

[lite/core/op_registry.h:255](https://github.com/PaddlePaddle/Paddle-Lite/blob/f8656fd616c9e458c0b9df75804c316bcdfdaf58/lite/core/op_registry.h#L255)

```C++
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

```C++
REGISTER_LITE_OP(arg_max, paddle::lite::operators::ArgmaxOpLite);
```

[创建Op并选择Kernel](https://github.com/PaddlePaddle/Paddle-Lite/blob/f8656fd616c9e458c0b9df75804c316bcdfdaf58/lite/core/program.cc#L300)

```C++
 // Create op
auto op = LiteOpRegistry::Global().Create(op_type);
```

### Instruction

**成员变量:**

```C++
  std::shared_ptr<OpLite> op_; 
  std::unique_ptr<KernelBase> kernel_;
  bool is_feed_fetch_op_{false};
  bool first_epoch_{true};
  bool has_run_{false};
```

#### Run函数

```C++
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

### OpLite与KernelLite

#### Subgraph

```C++
REGISTER_LITE_KERNEL(subgraph,  // Kernel
                     kNNAdapter, // Target
                     kAny,// Precision
                     kNCHW, // Layout
                     paddle::lite::kernels::nnadapter::SubgraphCompute, // Kernel Class
                     def) // alias
    .BindInput("Inputs",
               {LiteType::GetTensorTy(TARGET(kAny),
                                      PRECISION(kAny),
                                      DATALAYOUT(kNCHW))})
    .BindOutput("Outputs",
                {LiteType::GetTensorTy(TARGET(kAny),
                                       PRECISION(kAny),
                                       DATALAYOUT(kNCHW))})
    .Finalize();
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

```C++
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

```C++
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

```C++
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

```C++
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

```C++
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

定义在Paddle-Lite/lite/backends/nnadapter/nnadapter/src/driver/XXX/engine.h

```C++
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

```C++
Device::Device() { InitializeDevice(); }
```

#### `Context::Context(void* device, const char* properties)`

> 参数解析，根据参数设置设备上下文

device context是“设备运行环境”

```C++
Context::Context(void* device, const char* properties) : device_(device) {
  // Extract the runtime parameters from the context properties
  NNADAPTER_LOG(INFO) << "properties: " << std::string(properties);
  auto key_values = GetKeyValues(properties);
  
  //......
}
```

`GetKeyValues`

```C++
// Parse and get the key value map from a string
std::map<std::string, std::string> GetKeyValues(
    const char* properties,
    const std::string& delimiter = ";",
    const std::string& assignment = "=");
```

[google_xnnpack](https://github.com/PaddlePaddle/Paddle-Lite/tree/develop/lite/backends/nnadapter/nnadapter/src/driver/google_xnnpack)例子

```C++
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

```C++
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