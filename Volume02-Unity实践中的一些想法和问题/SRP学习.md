# Unity的SRP相关内容学习
SRP(SriptableRenderPipeline)可以说是目前Unity最大的特色了，可以通过SRP非常方便的更改渲染管线，
常用的URP和HDRP也都是Unity官方在SRP的基础上写出来的，所以理解SRP非常重要。

SRP 的架构设计非常精妙，理解了“数据注入 -> 指令打包 -> 统一提交”的流向，写管线时就会清晰很多，理解了SRP之后写URP项目也会如鱼得水。

# RenderPipeline和RenderPipelineAsset
Unity 引擎读取你配置好的 RenderPipelineAsset。

引擎在后台调用 Asset 里的重写方法 CreatePipeline()。

Asset 在这个方法里，把自己的配置数据作为参数，new 出来一个 RenderPipeline 实例并交还给引擎。

引擎随后每帧调用这个 RenderPipeline 实例的 Render() 方法。

>为什么要分成两个类？ 因为 Unity 需要区分“数据”和“逻辑”。


# 渲染指令
## ScriptableRenderContext (context) 桥梁与大管家
`context` 是 C# 脚本与 Unity 底层 C++ 图形引擎通信的唯一桥梁。

它主要负责宏观层面的调度：执行 CommandBuffer、画天空盒 (context.DrawSkybox)、画剔除后的网格 (context.DrawRenderers)。

在一切都组装完成后调用Submit()方法传递给C++引擎和GPU去执行。
```csharp
context.Submit();
```

## CommandBuffer 底层图形指令的打包器
`CommandBuffer` 里装的是原生的、直接给 GPU 下达的渲染状态指令。

它的核心意义在于“打包预组装”。我们为了减少 C# 和 C++ 之间的通信开销，会把几十条指令写进 CommandBuffer，然后一次性塞给 context。

例如设置渲染目标（Render Target）、清除屏幕（Clear）、设置全局材质参数（SetGlobalColor）、调用计算着色器（DispatchCompute）。

常用的执行方法是将执行和清空绑定在一起：
```csharp
void ExecuteBuffer () 
{
    context.ExecuteCommandBuffer(buffer); // 执行CommandBuffer也得传递给context，光靠Buffer无法运行。
    buffer.Clear();
}
```
