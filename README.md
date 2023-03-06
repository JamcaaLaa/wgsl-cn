你好，这是一个介绍 WGSL 的仓库，蓝本源自 [WebGPU Shading Language](https://www.w3.org/TR/WGSL)。

---

下面是一个用点光源给具有纹理的几何体着色的片段着色器，作为本仓库的起手式：

```rust
struct PointLight {
  position: vec3f,
  color: vec3f,
}
struct LightStorage {
  pointCount: u32,
  point: array<PointLight>,
}
@group(0) @binding(0) var<storage> lights: LightStorage;

// 纹理和采样器
@group(1) @binding(0) var baseColorSampler: sampler;
@group(1) @binding(1) var baseColorTexture: texture_2d<f32>;

// 片段着色器的入口函数，参数将从顶点着色器中传递过来
@fragment
fn fragmentMain(
  @location(0) worldPos: vec3f,
  @location(1) normal: vec3f,
  @location(2) uv: vec2f
) -> @location(0) vec4f {
  // 取基本颜色
  let baseColor = textureSample(baseColorTexture, baseColorSampler, uv);

  let N = normalize(normal);
  var surfaceColor = vec3f(0);

  // 迭代所有点光源
  for (var i = 0u; i < lights.pointCount; i++) {
    let worldToLight = lights.point[i].position - worldPos;
    let dist = length(worldToLight);
    let dir = normalize(worldToLight);

    // 计算出这个点光源的贡献量
    let radiance = lights.point[i].color * (1 / pow(dist, 2));
    let nDotL = max(dot(N, dir), 0);

    // 累计光量到表面颜色上
    surfaceColor += baseColor.rgb * radiance * nDotL;
  }

  // 返回颜色值
  return vec4(surfaceColor, baseColor.a);
}
```

WebGPU 会递送两种 GPU 指令到 GPU，一种是来自渲染管线的 **绘制指令**，另一种是来自计算管线的 **计算指令**。这两种管线的着色器都要用到 WGSL 来编写着色代码。

> 严格来说，WGSL 程序的入口点函数不是必须的。但是想让着色器正确地运作，那必须遵守 GPUProgrammableStage 接口的规定，那就是必须有一个入口函数。

WGSL 是一种静态类型的命令式语言，它没有类型的隐式转换或提升。



# 官方文档中的符号约定

- 斜体字 - 表示该语法规则

- 加粗字和右侧单引号 `'` - 表示关键字和标记

- 英文冒号 `:` - 表示语法规则注册

- 竖线 `|` - 表示其它候选项

- 英文问号 `?` - 表示左边被修饰的关键字、语法出现 0 或 1 次（即可选）

- 星号 `*` - 表示左边被修饰的关键字、语法出现 0 或多次

- 加号 `+` - 表示左边被修饰的关键字、语法出现 1 或多次

- 成对的小括号 `()` - 表示一组元素



# 目录

- [01 代码文本结构与声明语法]

- [02 类型 - Types]

- [03 变量与值 - Variable & Value]

- [04 表达式 - Expressions]

- [05 语句 - Statements]

- [06 函数 - Functions]

- [07 入口点 - EntryPoints]

- [08 最大限制]

- [09 内存说明]

- [10 关键字、保留字和语法标记]

- [11 内置值和内置函数]


