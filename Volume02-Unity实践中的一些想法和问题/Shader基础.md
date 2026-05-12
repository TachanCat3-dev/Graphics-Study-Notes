# UnityShaderlab中的一些经验
这篇随手收录了一些个人写Shader时的疑惑点和常用的数学知识 无序

# 纹理TEXTURE使用方法和API示例

## Shader中贴图的声明和使用流程
Unity默认的Shader模板的Properties中有BaseMap贴图声明

形式如下:

    `[MainTexture]_BaseMap ("Base Map", 2D) = "white" {}`

`[MainTexture]` Property Attribute (属性标签 / 特性)。 可选参数，用于控制材质面板的 UI 行为或指定特殊逻辑。

`_BaseMap`: Property Name (属性名 / 内部变量名)。

`"Base Map"`: Display Name (显示名)。 可以自己定义在Unity材质面板中显示的名称。

`2D` : Property Type (属性类型)。 标准的二维纹理。常用的还有`Cube`(由六张图像组成 常用于天空盒与环境反射探针) `3D`(用于体积雾流体数据计算等)。

`white`: Default Value (默认值)。

接下来会在HLSLPROGRAM中使用

    `TEXTURE2D(_BaseMap);`
    `SAMPLER(sampler_BaseMap);`

## 法线贴图的使用方法
声明:

`_NormalMap("Normal Map", 2D) = "bump" {}`

正常进行贴图的准备

`TEXTURE2D(_NormalMap);`

`SAMPLER(sampler_NormalMap);`

采样时略有不同 这里我们要专门为法线贴图使用Unity给我们准给好的工具:

`float3 normalTS = UnpackNormalScale(normalMap, _normalScale);`

这里的UnpackNormal干了什么?


