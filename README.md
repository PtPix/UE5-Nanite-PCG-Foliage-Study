# UE5 Nanite Foliage & PCG Performance Research

这是一个基于 Unreal Engine 5 开发的技术 Demo，旨在探索 **Nanite Foliage** 的渲染特性、**PCG (Procedural Content Generation)** 框架的实际应用，以及大规模植被场景的性能优化方案。

主要目的在于学习 **Nanite** 的原理，核心内容可见Docs.

<img width="1734" height="766" alt="image" src="https://github.com/user-attachments/assets/f481bac2-ccb9-465d-844b-af308069b607" />

<img width="1930" height="830" alt="image" src="https://github.com/user-attachments/assets/2af3a8ff-fb4c-44e9-ba05-b7a6021614ee" />



## 🌟 项目
- **过程化生态构建**: 利用 PCG 框架实现基于 Spline 与 Terrain 的多层次植被分布（树木 -> 灌木 -> 碎石 -> 落叶）。
- **渲染底层分析**: 深入分析了 Nanite 在处理极细碎模型时的 `Preserve Surface Area` 与 `Voxelize` 两种算法的源码实现。
- **性能优化实践**: 通过 Shadow Path 优化、Nanite 精度控制（MaxPixelPerEdge）及 PCG 距离裁剪解决了密集场景的帧率瓶颈。

---

## 🛠 技术实现

### 1. PCG 逻辑
项目实现了一个高度可控的 PCG 生成器：
- **范围控制**: 通过 Actor 中的 Spline 动态限定生成区域。
- **层次化采样**: 采用多级采样逻辑，大树采样位置作为基准，递归生成底层灌木与地表覆盖物。
- **防穿插处理**: 利用 Bounds 节点控制模型密度，避免网格体重叠。

### 2. Nanite 深度分析 (重点)
针对远处植被消失的问题，本项目对比研究了UE给出的两种解决方案：
- **Surface Preservation**: 
  - **原理**: 寻找面积流失严重的材质边界，根据 `DilateDistance = -DilateSurfaceArea / Perimeter` 计算顶点扩张距离。
  - **副作用**: 容易导致严重的 Overdraw。
- **Voxelize (体素化)**: 
  - **原理**: 采用 26-Separating 保守体素化，通过蒙特卡洛积分（Density & NDF）模拟光子穿透。
  - **优势**: 在极远景下提供了更稳定的 VisibilityBuffer 表现，减少了三角形交叉带来的开销。

---

## 🚀 性能优化方案
针对密集植被环境，本项目实施了以下优化：
| 优化项 | 处理手段 | 效果 |
| :--- | :--- | :--- |
| **Shadow** | 关闭细碎物体 Dynamic Shadow，改用 Contact Shadow | Shadow Depth 从 19ms 降至 5ms |
| **Nanite 精度** | 调整 `MaxPixelPerEdge` 从 1 升至 2 | 每帧耗时减少 ~3.6ms |
| **Culling** | PCG Static Mesh Spawner 配合材质透明度淡出 | 显著降低 Nanite 内存页压力，解决 Popping  |

---

## 📂 项目结构
- 'Contents/':
  - `Levels/`: 包含小型测试关卡与大世界测试关卡。
  - `PCG/`: PCG Graph 资产与容器 Blueprint。
- `Docs/`: 详细的技术文档（PDF），包含：
  - `Nanite.pdf` - 渲染管线深度分析
  - `InstanceRendering.pdf` - 实例化渲染研究
  - `OIT.pdf` & `DitheredOpacity.pdf` - 半透明与抖动透明度研究

## 💻 运行环境
- **Engine**: Unreal Engine 5.7
- **Plugins**: PCG Framework, Nanite
