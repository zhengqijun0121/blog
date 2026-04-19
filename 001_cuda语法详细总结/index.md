# CUDA 语法详细总结


# CUDA语法详细总结
本文基于CUDA 11+ 现代标准，覆盖CUDA核心编程模型、语法规范、运行时API、内置函数、进阶特性与使用约束，兼容NVIDIA主流GPU架构，是CUDA开发的完整语法参考。

## 一、CUDA核心基础概念
CUDA是NVIDIA推出的GPU并行计算编程模型，核心是将并行任务映射到GPU海量线程执行，先明确核心基础概念，是理解语法的前提：
1.  **主机(Host)**：CPU及其内存（主机内存），负责逻辑控制、资源管理
2.  **设备(Device)**：GPU及其内存（设备内存），负责大规模并行计算
3.  **核函数(Kernel)**：GPU上并行执行的函数，是CUDA并行计算的核心单元
4.  **线程层次结构（从大到小）**
    - Grid：一个核函数调用对应一个Grid，由多个Block组成，全局可见
    - Block：Grid内的线程块，由多个Thread组成，Block内线程可共享内存、同步
    - Thread：最小执行单元，单一线程，对应一个独立的计算任务
    - Warp：GPU硬件调度的最小单元，固定32个线程为一个Warp，遵循SIMT（单指令多线程）执行模型
5.  **内存层次**：按访问速度从快到慢排序：寄存器 > 共享内存 > L1/L2缓存 > 常量内存 > 全局/本地/纹理内存

---

## 二、核函数定义与调用语法
核函数是CUDA程序的核心，是GPU并行执行的入口，有严格的语法规范。

### 2.1 核心函数类型限定符
CUDA通过限定符明确函数的执行位置与调用规则，是最基础的语法核心：

| 限定符 | 标准语法 | 执行位置 | 可调用位置 | 核心规则与约束 |
|--------|----------|----------|------------|----------------|
| `__global__` | `__global__ void func(...)` | GPU设备端 | 主机端/设备端（动态并行） | 1. 返回值必须为`void`；2. 必须通过`<<<>>>`执行配置调用；3. 异步执行，主机端需显式同步；4. 不支持递归；5. 不能是类的非静态成员函数 |
| `__device__` | `__device__ T func(...)` | GPU设备端 | 设备端（`__global__`/`__device__`函数内） | 1. 可带任意返回值；2. 仅能在设备端调用；3. 支持内联控制；4. 可作为类的成员函数 |
| `__host__` | `__host__ T func(...)` | CPU主机端 | 主机端 | 等价于普通C/C++函数，可省略；可与`__device__`联用，同时编译出主机/设备双版本函数 |
| `__noinline__`/`__forceinline__` | 配合`__device__`使用 | 设备端 | - | 强制控制设备函数是否内联，优化性能 |

#### 基础核函数定义示例
```cpp
// 向量加法核函数：两个N维向量逐元素相加
__global__ void vectorAdd(const float* A, const float* B, float* C, int N) {
    // 计算当前线程的全局ID
    int tid = blockIdx.x * blockDim.x + threadIdx.x;
    // 边界判断：避免越界访问
    if (tid < N) {
        C[tid] = A[tid] + B[tid];
    }
}
```

### 2.2 核函数调用语法：执行配置`<<<>>>`
CUDA通过`<<<>>>`执行配置指定核函数的并行规模，是CUDA独有的核心语法，格式如下：
```cpp
kernel_name<<<grid_dim, block_dim, shared_mem_size, stream>>>(参数列表);
```

#### 执行配置4个参数详解
| 参数 | 类型 | 作用 | 约束与规范 |
|------|------|------|------------|
| `grid_dim` | `dim3` | Grid的维度，指定Grid中Block的数量 | 支持1D/2D/3D，默认值`dim3(1,1,1)`；主流架构x维度最大支持`2^31-1`，y/z维度最大65535 |
| `block_dim` | `dim3` | Block的维度，指定单个Block内的线程数量 | 支持1D/2D/3D，默认值`dim3(1,1,1)`；**单Block内线程总数最大为1024（全架构通用）**，必须是32的倍数（Warp对齐，避免性能损耗） |
| `shared_mem_size` | `size_t` | 动态共享内存大小，单位字节 | 可选参数，默认0；用于核函数内`extern __shared__`声明的动态共享内存 |
| `stream` | `cudaStream_t` | 核函数执行的CUDA流 | 可选参数，默认0（默认流）；用于异步执行、多流并行 |

#### 核函数调用完整示例
```cpp
#include <cuda_runtime.h>
#include <cstdio>
#include <cstdlib>

// 错误检查宏（调试必备，下文详细说明）
#define CHECK_CUDA_ERROR(call) \
    do { \
        cudaError_t err = call; \
        if (err != cudaSuccess) { \
            fprintf(stderr, "CUDA Error at %s:%d: %s\n", __FILE__, __LINE__, cudaGetErrorString(err)); \
            exit(EXIT_FAILURE); \
        } \
    } while(0)

int main() {
    const int N = 1024 * 1024;
    const size_t size = N * sizeof(float);

    // 1. 主机内存分配与初始化
    float* h_A = (float*)malloc(size);
    float* h_B = (float*)malloc(size);
    float* h_C = (float*)malloc(size);
    for (int i = 0; i < N; i++) {
        h_A[i] = 1.0f;
        h_B[i] = 2.0f;
    }

    // 2. 设备内存分配
    float *d_A, *d_B, *d_C;
    CHECK_CUDA_ERROR(cudaMalloc(&d_A, size));
    CHECK_CUDA_ERROR(cudaMalloc(&d_B, size));
    CHECK_CUDA_ERROR(cudaMalloc(&d_C, size));

    // 3. 数据拷贝：主机内存 -> 设备内存
    CHECK_CUDA_ERROR(cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice));
    CHECK_CUDA_ERROR(cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice));

    // 4. 核函数执行配置与调用
    const int blockSize = 256; // 单Block线程数，32的倍数
    const int gridSize = (N + blockSize - 1) / blockSize; // 向上取整，保证覆盖所有元素
    vectorAdd<<<gridSize, blockSize>>>(d_A, d_B, d_C, N);

    // 检查核函数启动错误
    CHECK_CUDA_ERROR(cudaGetLastError());
    // 同步：等待核函数执行完成（核函数异步执行，必须显式同步）
    CHECK_CUDA_ERROR(cudaDeviceSynchronize());

    // 5. 数据拷贝：设备内存 -> 主机内存
    CHECK_CUDA_ERROR(cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost));

    // 6. 资源释放
    CHECK_CUDA_ERROR(cudaFree(d_A));
    CHECK_CUDA_ERROR(cudaFree(d_B));
    CHECK_CUDA_ERROR(cudaFree(d_C));
    free(h_A);
    free(h_B);
    free(h_C);

    return 0;
}
```

### 2.3 核函数核心约束
1.  核函数调用是**异步执行**的，主机端调用后立即返回，不会等待核函数执行完成，必须通过同步API等待执行结束
2.  核函数参数默认存储在常量内存中，最大参数大小限制为4KB，大参数需通过设备内存指针传递
3.  核函数不支持递归、可变参数列表、函数指针调用，不能定义虚函数

---

## 三、线程层次内置变量与全局ID计算
核函数内通过**只读内置变量**获取线程/块的坐标与维度，是并行计算的核心，所有变量仅在设备端可用。

### 3.1 核心内置变量详解
| 内置变量 | 类型 | 含义 | 取值范围 |
|----------|------|------|----------|
| `threadIdx` | `dim3` | 线程在Block内的相对坐标 | `threadIdx.x ∈ [0, blockDim.x-1]`，y/z维度同理 |
| `blockIdx` | `dim3` | Block在Grid内的相对坐标 | `blockIdx.x ∈ [0, gridDim.x-1]`，y/z维度同理 |
| `blockDim` | `dim3` | 单个Block的线程维度（核函数调用时的`block_dim`） | 核函数执行期间固定不变 |
| `gridDim` | `dim3` | Grid的Block维度（核函数调用时的`grid_dim`） | 核函数执行期间固定不变 |
| `warpSize` | `int` | 硬件Warp的线程数 | 固定为32（全NVIDIA架构通用） |

### 3.2 全局线程ID计算（高频使用）
全局线程ID用于定位当前线程处理的数据索引，是核函数内的必备代码，分维度场景如下：

#### 1. 1D Grid + 1D Block（向量/一维数组处理，最常用）
```cpp
__global__ void kernel1D(const float* arr, int N) {
    // 核心公式：全局ID = 块ID * 块内线程数 + 块内线程ID
    int global_tid = blockIdx.x * blockDim.x + threadIdx.x;
    // 必须做边界判断，避免越界
    if (global_tid < N) {
        // 业务逻辑
    }
}
```

#### 2. 2D Grid + 2D Block（图像/矩阵处理）
```cpp
__global__ void kernel2D(float* matrix, int width, int height) {
    // 计算行列坐标
    int col = blockIdx.x * blockDim.x + threadIdx.x;
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    // 边界判断
    if (col < width && row < height) {
        // 行优先的全局线性ID
        int global_tid = row * width + col;
        matrix[global_tid] = 0.0f;
    }
}
```

#### 3. 3D Grid + 3D Block（3D体数据/三维数组处理）
```cpp
__global__ void kernel3D(float* volume, int x_dim, int y_dim, int z_dim) {
    int x = blockIdx.x * blockDim.x + threadIdx.x;
    int y = blockIdx.y * blockDim.y + threadIdx.y;
    int z = blockIdx.z * blockDim.z + threadIdx.z;
    if (x < x_dim && y < y_dim && z < z_dim) {
        int global_tid = z * x_dim * y_dim + y * x_dim + x;
        volume[global_tid] = 0.0f;
    }
}
```

---

## 四、内存模型与对应语法
CUDA的内存模型是性能优化的核心，不同内存类型有严格的声明语法、作用域与使用规则，按访问特性分类详解如下：

### 4.1 寄存器（Register）
- **特性**：线程私有，访问速度最快，核函数内非数组局部变量默认分配到寄存器
- **语法**：无显式限定符，核函数内局部变量自动分配
- **示例**：
  ```cpp
  __global__ void kernel() {
      int a = 10;       // 存储在寄存器
      float b = 3.14f;  // 存储在寄存器
  }
  ```
- **约束**：单线程寄存器数量有限，寄存器过多会导致GPU占用率下降，可通过`__launch_bounds__`限制寄存器使用

### 4.2 本地内存（Local Memory）
- **特性**：线程私有，实际位于设备全局内存，访问延迟极高，仅在寄存器溢出时自动使用
- **语法**：无显式限定符，核函数内大数组、动态索引数组、无法编译期确定大小的数组会自动分配到本地内存
- **示例**：
  ```cpp
  __global__ void kernel() {
      int arr[1024]; // 大数组，寄存器无法容纳，自动分配到本地内存
  }
  ```
- **最佳实践**：尽量避免使用本地内存，会严重降低核函数性能

### 4.3 共享内存（Shared Memory）
- **特性**：Block内所有线程共享，速度仅次于寄存器，生命周期与Block一致，用于Block内线程通信、数据复用，是性能优化的核心
- **核心语法**：`__shared__`限定符，分为静态共享内存和动态共享内存两种

#### 1. 静态共享内存（编译期确定大小）
```cpp
__global__ void staticSharedKernel() {
    // 静态共享内存声明：Block内所有线程共享该数组
    __shared__ int s_arr[256];
    int tid = threadIdx.x;

    // 线程写入共享内存
    s_arr[tid] = tid;
    // 块内同步：确保所有线程写入完成，再进行读取
    __syncthreads();

    // 线程读取共享内存
    int val = s_arr[255 - tid];
}
```

#### 2. 动态共享内存（运行时确定大小）
```cpp
// 核函数定义：extern 声明动态共享内存
__global__ void dynamicSharedKernel() {
    // 同一个核函数内只能有一个动态共享内存声明
    extern __shared__ float s_data[];
    // 多变量复用：手动地址偏移
    float* s_arr1 = s_data;                // 前256个float
    float* s_arr2 = &s_data[256];          // 后256个float
}

// 核函数调用：第三个参数指定动态共享内存总大小（字节）
int main() {
    int blockSize = 256;
    int gridSize = 16;
    // 动态共享内存大小：256*2 * sizeof(float) = 2048字节
    dynamicSharedKernel<<<gridSize, blockSize, 2048>>>();
    return 0;
}
```

#### 共享内存核心约束
1.  读写共享内存必须配合`__syncthreads()`同步，避免先读后写导致的数据竞争
2.  `__syncthreads()`必须在Block内所有线程都能到达的代码路径，否则会导致死锁
3.  需避免bank冲突：共享内存分为32个bank，同一Warp内多个线程访问同一bank的不同地址会导致性能下降

### 4.4 全局内存（Global Memory）
- **特性**：设备端全局可见，所有Grid/Block/线程均可访问，生命周期与分配一致，容量最大，访问延迟最高，是GPU的主内存
- **核心语法**：无显式限定符，通过CUDA运行时API分配/释放/拷贝，主机端通过指针管理，设备端通过指针访问
- **核心API**：`cudaMalloc`（分配）、`cudaFree`（释放）、`cudaMemcpy`（拷贝），详见下文运行时API章节
- **核心约束**：
  1.  主机端**绝对不能直接解引用设备全局内存指针**，必须通过`cudaMemcpy`拷贝数据后访问
  2.  全局内存访问需满足**合并访问**：同一Warp内的线程访问连续、对齐的内存地址，可最大化带宽，否则性能会大幅下降

### 4.5 常量内存（Constant Memory）
- **特性**：只读内存，全局可见，有专用常量缓存，生命周期与程序一致，适合存储所有线程共享的只读参数，单设备最大64KB
- **核心语法**：`__constant__`限定符，必须在全局作用域声明，主机端通过`cudaMemcpyToSymbol`初始化
- **完整示例**：
```cpp
// 全局作用域声明常量内存
__constant__ float d_const[256];

__global__ void constKernel(float* out, int N) {
    int tid = blockIdx.x * blockDim.x + threadIdx.x;
    if (tid < N) {
        // 设备端只读访问常量内存
        out[tid] = d_const[tid] * 2.0f;
    }
}

int main() {
    // 主机端初始化常量内存
    float h_const[256];
    for (int i = 0; i < 256; i++) h_const[i] = i;
    // 数据拷贝：主机内存 -> 常量内存
    CHECK_CUDA_ERROR(cudaMemcpyToSymbol(d_const, h_const, 256 * sizeof(float)));

    // 核函数调用
    constKernel<<<1, 256>>>(d_out, 256);
    return 0;
}
```
- **核心约束**：常量内存设备端仅支持只读，写入会导致未定义行为；仅当所有线程访问同一地址时，常量缓存效率最高

### 4.6 纹理/表面内存（Texture/Surface Memory）
- **特性**：基于GPU纹理单元的内存，有专用纹理缓存，适合空间局部性强的2D/3D数据（如图像、体数据），支持硬件滤波、地址归一化、边界处理
- **语法分类**：分为纹理引用（老API）和纹理对象（CUDA 5.0+ 新API，推荐使用）
- **核心API**：`cudaCreateTextureObject`（创建纹理对象）、`tex2D`（2D纹理采样）、`cudaDestroyTextureObject`（销毁）

---

## 五、CUDA运行时API核心语法
CUDA运行时API封装了设备管理、内存管理、同步、流与事件等核心能力，是主机端控制GPU的核心接口，所有API均返回`cudaError_t`错误码，成功返回`cudaSuccess`。

### 5.1 设备管理API
| API函数 | 函数原型 | 核心功能 | 常用示例 |
|---------|----------|----------|----------|
| `cudaGetDeviceCount` | `cudaError_t cudaGetDeviceCount(int* count)` | 获取当前系统支持的CUDA设备数量 | `int deviceCount; cudaGetDeviceCount(&deviceCount);` |
| `cudaSetDevice` | `cudaError_t cudaSetDevice(int device)` | 设置当前使用的CUDA设备（多GPU场景） | `cudaSetDevice(0); // 使用0号GPU` |
| `cudaGetDeviceProperties` | `cudaError_t cudaGetDeviceProperties(cudaDeviceProp* prop, int device)` | 获取指定GPU的硬件属性（型号、内存大小、核心数、最大线程数等） | `cudaDeviceProp prop; cudaGetDeviceProperties(&prop, 0); printf("GPU: %s\n", prop.name);` |
| `cudaDeviceReset` | `cudaError_t cudaDeviceReset()` | 重置当前设备，释放所有分配的资源 | `cudaDeviceReset();` |

### 5.2 内存管理API（核心）
| API函数 | 函数原型 | 核心功能 | 关键参数说明 |
|---------|----------|----------|--------------|
| `cudaMalloc` | `cudaError_t cudaMalloc(void** devPtr, size_t size)` | 分配设备全局内存 | `devPtr`：设备指针的地址；`size`：分配大小（字节） |
| `cudaFree` | `cudaError_t cudaFree(void* devPtr)` | 释放设备全局内存 | `devPtr`：设备内存指针 |
| `cudaMemcpy` | `cudaError_t cudaMemcpy(void* dst, const void* src, size_t count, cudaMemcpyKind kind)` | 主机与设备间阻塞式内存拷贝 | `kind`：拷贝方向，核心枚举值：<br>`cudaMemcpyHostToDevice`：主机→设备<br>`cudaMemcpyDeviceToHost`：设备→主机<br>`cudaMemcpyDeviceToDevice`：设备→设备 |
| `cudaMemcpyAsync` | `cudaError_t cudaMemcpyAsync(void* dst, const void* src, size_t count, cudaMemcpyKind kind, cudaStream_t stream = 0)` | 异步内存拷贝，非阻塞 | 新增`stream`参数，指定执行的CUDA流 |
| `cudaMemset` | `cudaError_t cudaMemset(void* devPtr, int value, size_t count)` | 设备内存按字节初始化 | `value`：要设置的字节值；`count`：初始化字节数 |
| `cudaMallocManaged` | `cudaError_t cudaMallocManaged(void** devPtr, size_t size, unsigned int flags = cudaMemAttachGlobal)` | 分配统一内存，主机和设备可通过同一指针访问，自动处理数据迁移 | `devPtr`：统一内存指针地址；`size`：分配大小 |
| `cudaMemcpyToSymbol` | `cudaError_t cudaMemcpyToSymbol(const void* symbol, const void* src, size_t count)` | 主机数据拷贝到常量内存/全局符号 | `symbol`：`__constant__`声明的符号名 |

### 5.3 同步API
| API函数 | 核心功能 | 适用场景 |
|---------|----------|----------|
| `cudaDeviceSynchronize()` | 阻塞主机端，等待当前设备上所有任务（核函数、异步拷贝）执行完成 | 单流场景、核函数调试、错误定位 |
| `cudaStreamSynchronize(cudaStream_t stream)` | 阻塞主机端，等待指定流上的所有任务执行完成 | 多流场景，仅同步指定流，不阻塞其他流 |
| `cudaEventSynchronize(cudaEvent_t event)` | 阻塞主机端，等待指定事件记录完成 | 核函数计时、流间依赖同步 |

### 5.4 CUDA流与事件API
#### 1. CUDA流
流是GPU上的异步执行队列，不同流之间的任务可并行执行，实现计算与数据拷贝的重叠，最大化GPU利用率。
```cpp
// 流创建
cudaStream_t stream;
CHECK_CUDA_ERROR(cudaStreamCreate(&stream));

// 核函数、异步拷贝在指定流执行
vectorAdd<<<gridSize, blockSize, 0, stream>>>(d_A, d_B, d_C, N);
cudaMemcpyAsync(d_A, h_A, size, cudaMemcpyHostToDevice, stream);

// 同步流
CHECK_CUDA_ERROR(cudaStreamSynchronize(stream));

// 销毁流
CHECK_CUDA_ERROR(cudaStreamDestroy(stream));
```

#### 2. CUDA事件
事件用于标记流中的执行点，实现核函数精准计时、流间同步，是性能测试的核心工具。
```cpp
// 事件创建
cudaEvent_t start, stop;
CHECK_CUDA_ERROR(cudaEventCreate(&start));
CHECK_CUDA_ERROR(cudaEventCreate(&stop));

// 记录事件与核函数执行
CHECK_CUDA_ERROR(cudaEventRecord(start, stream));
vectorAdd<<<gridSize, blockSize, 0, stream>>>(d_A, d_B, d_C, N);
CHECK_CUDA_ERROR(cudaEventRecord(stop, stream));

// 同步事件并计算耗时（毫秒）
CHECK_CUDA_ERROR(cudaEventSynchronize(stop));
float ms;
CHECK_CUDA_ERROR(cudaEventElapsedTime(&ms, start, stop));
printf("Kernel execution time: %.2f ms\n", ms);

// 销毁事件
CHECK_CUDA_ERROR(cudaEventDestroy(start));
CHECK_CUDA_ERROR(cudaEventDestroy(stop));
```

### 5.5 错误处理API（调试必备）
CUDA所有API的错误都必须检查，否则无法定位核函数启动、执行、内存操作的问题，核心API如下：
1.  `cudaGetLastError()`：获取并清除之前异步操作的最后一个错误码，用于检查核函数启动错误
2.  `cudaPeekAtLastError()`：获取但不清除最后一个错误码
3.  `cudaGetErrorString(cudaError_t error)`：将错误码转换为人类可读的错误字符串

通用错误检查宏（可直接复用）：
```cpp
#define CHECK_CUDA_ERROR(call) \
    do { \
        cudaError_t err = call; \
        if (err != cudaSuccess) { \
            fprintf(stderr, "CUDA Error at %s:%d: %s\n", __FILE__, __LINE__, cudaGetErrorString(err)); \
            exit(EXIT_FAILURE); \
        } \
    } while(0)
```

---

## 六、设备端内置函数
设备端内置函数是核函数内的核心工具，分为同步函数、原子操作、数学函数、Warp操作四大类，仅在设备端可用。

### 6.1 同步函数
#### 1. `__syncthreads()`
- **语法**：`__device__ void __syncthreads();`
- **功能**：Block内线程屏障同步，等待Block内所有线程执行到该语句后，再继续执行后续代码
- **核心约束**：必须在Block内所有线程都能到达的代码路径中调用，不能在仅部分线程执行的分支内调用，否则会导致死锁
- **正确示例**：见上文共享内存示例
- **错误示例**：
  ```cpp
  __global__ void badSyncKernel() {
      if (threadIdx.x < 128) {
          __syncthreads(); // 仅一半线程执行，死锁，严重错误
      }
  }
  ```

#### 2. `__syncwarp()`
- **语法**：`__device__ void __syncwarp(unsigned int mask = 0xffffffff);`
- **功能**：Warp内线程同步，仅等待`mask`指定的Warp内线程到达，CUDA 7.0+支持
- **优势**：相比`__syncthreads()`，仅同步Warp内线程，性能开销更小，适合Warp内线程通信

### 6.2 原子操作函数
原子操作是不可分割的读-改-写操作，避免多线程同时修改同一内存地址导致的数据竞争，所有原子操作均返回操作前的旧值。

高频使用的原子操作函数：
| 函数 | 标准语法 | 核心功能 | 支持数据类型 |
|------|----------|----------|--------------|
| `atomicAdd` | `T atomicAdd(T* address, T val)` | 原子加法：`*address += val` | int、unsigned int、long long、float、double（计算能力6.0+） |
| `atomicCAS` | `T atomicCAS(T* address, T compare, T val)` | 原子比较并交换：`if(*address == compare) *address = val` | int、unsigned int、long long、float |
| `atomicExch` | `T atomicExch(T* address, T val)` | 原子赋值：`*address = val` | int、unsigned int、float、long long |
| `atomicMin/Max` | `T atomicMin(T* address, T val)` | 原子取最小/最大值：`*address = min(*address, val)` | int、unsigned int、long long、float |
| `atomicSub` | `T atomicSub(T* address, T val)` | 原子减法：`*address -= val` | int、unsigned int |

#### 原子操作示例：多线程累加求和
```cpp
__global__ void sumKernel(const float* arr, float* sum, int N) {
    int tid = blockIdx.x * blockDim.x + threadIdx.x;
    if (tid < N) {
        // 多线程原子累加，避免数据竞争
        atomicAdd(sum, arr[tid]);
    }
}
```

### 6.3 数学函数
CUDA设备端提供了完整的数学函数，兼容C/C++标准，分为单精度（后缀`f`）和双精度版本，禁止在设备端使用主机端`cmath`库的函数。

高频使用的数学函数：
- 三角函数：`sinf`/`cosf`/`tanf`（单精度）、`sin`/`cos`/`tan`（双精度）
- 指数对数：`expf`/`logf`/`powf`/`sqrtf`（单精度）、`exp`/`log`/`pow`/`sqrt`（双精度）
- 数值运算：`fabsf`/`fminf`/`fmaxf`/`floorf`/`ceilf`（单精度）、`fabs`/`fmin`/`fmax`/`floor`/`ceil`（双精度）

### 6.4 Warp Shuffle函数
CUDA 3.0+支持，无需共享内存，直接在Warp内的线程之间交换数据，延迟更低、性能更高，核心函数如下：
| 函数 | 核心功能 |
|------|----------|
| `__shfl_sync(mask, var, srcLane)` | 从Warp内`srcLane`号线程获取`var`的值 |
| `__shfl_down_sync(mask, var, delta)` | 从当前线程ID + `delta`的线程获取`var`的值 |
| `__shfl_up_sync(mask, var, delta)` | 从当前线程ID - `delta`的线程获取`var`的值 |
| `__shfl_xor_sync(mask, var, laneMask)` | 从当前线程ID ^ `laneMask`的线程获取`var`的值 |

#### 示例：Warp内归约求和
```cpp
__device__ float warpReduceSum(float val) {
    // Warp内并行归约，32线程→16→8→4→2→1
    for (int delta = 16; delta > 0; delta /= 2) {
        val += __shfl_down_sync(0xffffffff, val, delta);
    }
    return val; // 仅Warp内0号线程返回总和
}
```

---

## 七、CUDA C++特性与语法限制
CUDA基于C++标准开发，支持大部分C++11/14/17特性，同时有明确的语法限制。

### 7.1 支持的C++核心特性
1.  **模板**：核函数、`__device__`函数均支持模板，实现泛型编程
    ```cpp
    template <typename T>
    __global__ void templateAdd(const T* A, const T* B, T* C, int N) {
        int tid = blockIdx.x * blockDim.x + threadIdx.x;
        if (tid < N) C[tid] = A[tid] + B[tid];
    }
    // 调用：templateAdd<float><<<gridSize, blockSize>>>(d_A, d_B, d_C, N);
    ```
2.  **类与结构体**：支持自定义结构体、类，`__device__`函数可作为类的成员函数，核函数可作为类的静态成员函数
3.  **Lambda表达式**：C++11+支持`__device__` lambda，可在核函数内使用
4.  **函数重载、命名空间、静态变量**：完全支持
5.  **运算符重载**：支持自定义运算符重载，适配自定义数据类型

### 7.2 核心语法限制
1.  **核函数限制**：返回值必须为`void`、不支持递归、不支持可变参数、不能是类的非静态成员函数
2.  **设备端C++限制**：
    - 不支持运行时多态：不能定义虚函数，不能使用`dynamic_cast`
    - 不支持异常处理：不能使用`try/catch/throw`
    - 不支持`new/delete`：设备端不能动态分配内存（动态并行除外）
    - 不支持STL容器：设备端不能直接使用`std::vector`/`std::map`等，可使用Thrust库替代
3.  **全局变量限制**：设备端全局变量必须用`__device__`/`__constant__`限定，不能直接在主机端访问

### 7.3 核函数优化限定符
#### 1. `__launch_bounds__`
用于限制核函数的寄存器使用，提升GPU占用率，语法：
```cpp
__global__ void __launch_bounds__(maxThreadsPerBlock, minBlocksPerMultiprocessor)
kernel(...) {
    // 核函数代码
}
```
- `maxThreadsPerBlock`：该核函数单Block最大线程数
- `minBlocksPerMultiprocessor`：每个SM最少驻留的Block数（可选）

#### 2. 循环展开指令`#pragma unroll`
用于编译器循环展开，减少循环分支开销，提升性能：
```cpp
__global__ void unrollKernel() {
    // 手动指定展开4次
    #pragma unroll 4
    for (int i = 0; i < 4; i++) {
        // 循环逻辑
    }
    // 完全展开（编译期自动展开）
    #pragma unroll
    for (int i = 0; i < 8; i++) {
        // 循环逻辑
    }
}
```

---

## 八、进阶语法特性
### 8.1 动态并行（Dynamic Parallelism）
CUDA 5.0+支持，计算能力3.5+，核函数内可调用其他核函数，设备端可启动核函数、分配内存、同步，实现嵌套并行。
```cpp
// 子核函数
__global__ void childKernel() {
    // 子任务逻辑
}

// 父核函数
__global__ void parentKernel() {
    if (threadIdx.x == 0) {
        // 设备端调用核函数
        childKernel<<<1, 32>>>();
        // 设备端同步，等待子核函数执行完成
        cudaDeviceSynchronize();
    }
}
```

### 8.2 协同组（Cooperative Groups）
CUDA 9.0+支持，更灵活的线程同步模型，替代`__syncthreads()`，支持Block内、Warp内、Grid级别的线程同步，语法更安全、扩展性更强。
```cpp
#include <cooperative_groups.h>
namespace cg = cooperative_groups;

__global__ void cooperativeKernel() {
    // 获取当前Block的线程组
    cg::thread_block block = cg::this_thread_block();
    __shared__ int s_arr[256];

    s_arr[block.thread_rank()] = block.thread_rank();
    // 块内同步，等价于__syncthreads()，更安全
    block.sync();
}
```

### 8.3 统一内存（Unified Memory）
CUDA 6.0+支持，通过`cudaMallocManaged`分配，主机和设备使用同一指针访问，驱动自动处理数据在主机和设备间的迁移，大幅简化编程。
```cpp
int main() {
    const int N = 1024;
    float* data;
    // 分配统一内存
    CHECK_CUDA_ERROR(cudaMallocManaged(&data, N * sizeof(float)));

    // 主机端直接初始化
    for (int i = 0; i < N; i++) data[i] = i;

    // 设备端直接使用，无需手动拷贝
    vectorAdd<<<4, 256>>>(data, data, data, N);
    CHECK_CUDA_ERROR(cudaDeviceSynchronize());

    // 主机端直接访问结果
    printf("data[0] = %.2f\n", data[0]);

    cudaFree(data);
    return 0;
}
```

---

## 九、高频语法坑与最佳实践
1.  **必须检查所有CUDA API的返回值**，核函数启动后必须先检查`cudaGetLastError()`，再同步检查执行错误
2.  核函数是异步执行的，主机端必须显式同步，否则会导致数据未就绪就访问，出现未定义行为
3.  `__syncthreads()`必须在Block内所有线程都能到达的路径，禁止在分支内仅让部分线程执行
4.  单Block内线程数不能超过1024，必须是32的倍数（Warp对齐），避免Warp分化导致性能下降
5.  全局内存访问必须保证合并访问，共享内存访问要避免bank冲突，这是CUDA性能优化的核心
6.  主机端绝对不能直接解引用设备内存指针，必须通过`cudaMemcpy`拷贝数据后访问
7.  常量内存设备端仅支持只读，写入会导致未定义行为
8.  多流场景下，避免使用默认流（0），默认流会阻塞其他流的执行
9.  核函数参数不能超过4KB，大参数需通过设备内存指针传递
10. 尽量减少核函数内的寄存器使用，避免寄存器溢出到本地内存，提升GPU占用率

---

## 十、编译相关规范
1.  CUDA源文件后缀为`.cu`，使用NVIDIA的`nvcc`编译器编译
2.  基础编译命令：`nvcc -o vectorAdd vectorAdd.cu`
3.  指定GPU架构编译：`nvcc -arch=sm_75 -o vectorAdd vectorAdd.cu`（`sm_75`对应Turing架构，需根据GPU型号调整）
4.  启用C++标准：`nvcc -std=c++17 -o vectorAdd vectorAdd.cu`
5.  必须包含头文件：`#include <cuda_runtime.h>`，使用内置数学函数需包含`#include <math.h>`，使用半精度需包含`#include <cuda_fp16.h>`


