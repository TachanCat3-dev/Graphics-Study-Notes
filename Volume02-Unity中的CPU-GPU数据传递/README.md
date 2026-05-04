

# Unity中的CPU与GPU数据传递API使用方法
    因为本人仍在学习探索过程中 暂不能给出绝对正确且通俗易懂的解释

## CPU和GPU内的数据都是什么样的?

## 核心数据流:
    CPU 准备数据 -> 提交到 Command Buffer -> GPU 读取渲染


## C#(CPU)端用于传递给GPU的几大数据类型

### 基础单体传参（Material API）

`Material.SetFloat(), SetInt(), SetColor(), SetVector(), SetTexture(), SetMatrix()`

> 避坑： Renderer.material（会实例化新材质，破坏合批，可能引起内存泄漏）与 Renderer.sharedMaterial（修改全局共用材质）的区别。

### 全局数据传递（Global API）

`Shader.SetGlobalFloat(), SetGlobalTexture()` 等。

应用场景：全局天气系统、时间变量（如风吹草动的时间驱动）、全局环境光纹理。

高性能/多实例传参（MaterialPropertyBlock）

讲清楚为什么需要它：在使用 GPU Instancing 时，如何给成百上千个同材质但不同颜色/属性的物体传参，且不产生材质实例化（不额外增加Draw Call）。

使用方法：`Renderer.SetPropertyBlock()`。

大规模结构化数据传递（ComputeBuffer）

应用场景：Compute Shader计算、大规模粒子系统、传递自定义结构体数组。

核心API：`ComputeBuffer.SetData()` 和 `Material.SetBuffer()`

## Shader(GPU)端的数据接收类型

### ShaderLab 的 Properties 块

说明这只是为了让 Unity Inspector 面板能显示和调节，它不是真正在 GPU 代码里接收数据的变量。

HLSL 常规变量接收

C# 端的数据类型与 HLSL 类型的映射关系对照表（例如：Vector -> float4, Color -> float4, Texture -> Texture2D / sampler2D, Matrix -> float4x4）。

强调变量名必须与 C# 端 Set API 中的字符串（或 `Shader.PropertyToID`）完全一致。

常量缓冲区（Constant Buffer）与 SRP Batcher（现代渲染管线必修）

说明在 URP/HDRP 中，为了支持 SRP Batcher 提升性能，材质属性必须包裹在 CBUFFER_START(UnityPerMaterial) 和 CBUFFER_END 中。这也是区别于传统老旧教程的重要知识点。

StructuredBuffer 接收大规模数据

对应 CPU 端的 `ComputeBuffer`，HLSL 中如何使用 `StructuredBuffer<T>` 和 `RWStructuredBuffer<T>` 来读取和写入。


## 日常书写建议

改单个物体且不需要合批 ->`Material.SetX`

改全局环境 -> `Shader.SetGlobalX`

大量同材质物体各自状态不同 -> `MaterialPropertyBlock`

传递海量计算数据 -> `ComputeBuffer`