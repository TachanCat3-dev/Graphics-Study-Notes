# UnityShaderlab中的一些经验
这篇随手收录了一些个人写Shader时的疑惑点和常用的数学知识 省略了很多简单内容 无序

# 纹理TEXTURE使用方法和API示例

## Shader中贴图的基础声明和使用流程
Unity默认的Shader模板的Properties中有BaseMap贴图声明

形式如下:

    `[MainTexture]_BaseMap ("Base Map", 2D) = "white" {}`

`[MainTexture]`：Property Attribute (属性标签 / 特性)。 可选参数，用于控制材质面板的 UI 行为或指定特殊逻辑。

`_BaseMap`：Property Name (属性名 / 内部变量名)。

`"Base Map"`：Display Name (显示名)。 可以自己定义在Unity材质面板中显示的名称。

`2D`：Property Type (属性类型)。 标准的二维纹理。常用的还有`Cube`(由六张图像组成 常用于天空盒与环境反射探针) `3D`(用于体积雾流体数据计算等)。

`white`：Default Value (默认值)。 在Unity侧无指定纹理时生效。

接下来会在HLSLPROGRAM中使用

    TEXTURE2D(_BaseMap);
    SAMPLER(sampler_BaseMap);

我们需要在CBUFFER中声明偏移项 因为Unity使用宏处理缩放偏移项，所以用的是名称对齐，用原有的贴图名+_ST。

    CBUFFER_START(UnityPerMaterial)

    float4 _BaseMap_ST;
    float4 _DetailTex_ST;

    CBUFFER_END

同时在VertexShader中对其进行变换。

    OUT.uv.xy = TRANSFORM_TEX(IN.uv.xy, _BaseMap);
## 法线贴图的使用方法
声明

    [NoScaleOffset] _NormalMap("Normal Map", 2D) = "bump" {}

正常进行贴图的准备

    TEXTURE2D(_NormalMap);

    SAMPLER(sampler_NormalMap);

采样时略有不同 这里我们要专门为法线贴图使用Unity给我们准给好的法线解包工具:

    float3 normalTS = UnpackNormalScale(normalMap, _normalScale);

这里的UnpackNormal干了什么?

简而言之 Unity在导入NormalMap的时候会使用 ***DXT5nm*** 协议进行压缩，Unpack顾名思义就是把它解压缩回RGB空间。

注意到这个normalTS的后缀为 TS切线空间(Tangent Space)，我们需要用TBN矩阵将其转换到 WS世界空间(WorldSpace)。

## 混合法线
我们无法像之前那样用两个颜色相乘的方式混合法线 

我们可以通过在归一化之前对他们进行平均的方式混合 注意：这种混合方式会导致法线变平

```shaderlab
void InitializeFragmentNormal(inout Interpolators i)
{
	float3 mainNormal = UnpackScaleNormal(tex2D(_NormalMap, i.uv.xy), _BumpScale);
	float3 detailNormal = UnpackScaleNormal(tex2D(_DetailNormalMap, i.uv.zw), _DetailBumpScale);
	i.normal = (mainNormal + detailNormal) * 0.5;
	i.normal = i.normal.xzy;
	i.normal = normalize(i.normal);
}
```




