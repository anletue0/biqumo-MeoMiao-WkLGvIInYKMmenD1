> [【从UnityURP开始探索游戏渲染】](https://github.com)**专栏-直达**

**双向反射分布函数** Bidirectional Reflectance Distribution Function 解释当光线从某个方向照射到一个表面时，有多少光线被反射、反射方向有哪些。BRDF大多使用一个数学公式表示，并提供一些参数来调整材质属性。

BRDF（双向反射分布函数）是计算机图形学和光学中描述物体表面反射特性的核心数学模型，其定义和特性如下：

---

# **基本定义**

BRDF（Bidirectional Reflectance Distribution Function）表示‌**入射光方向（ωi）与出射光方向（ωr）的反射辐射率（radiance）与入射辐照度（irradiance）的比值**‌，数学表达式为：

$f\_r(ω\_i,ω\_r)=\frac{dL\_r(ω\_r)}{dE\_i(ω\_i)}$

其中，$L\_r*L\_r*$为反射辐射率，$E\_i$为入射辐照度‌。

---

# **核心特性**

* ‌**双向性**‌同时依赖入射和出射方向，能精确描述光线在表面的空间分布‌。
* ‌**能量守恒**‌反射率总和≤1，避免非物理的光能溢出‌。
* ‌**微观结构关联**‌通过微表面理论（Microfacet Theory）建模表面粗糙度对反射的影响‌。

---

# **物理意义**

* ‌**反射行为分解**‌
  + ‌**漫反射**‌：光线均匀散射（如Lambert模型）
  + ‌**镜面反射**‌：光线集中反射（如GGX模型）‌。
* ‌**材质区分**‌金属与非金属的BRDF差异显著（如菲涅尔效应在金属中更明显）‌。

---

# **‌BRDF的光照分解与实现原理‌**

BRDF将表面反射分为‌**漫反射Diffuse**‌ 和‌**镜面反射Specular**‌ 两部分（环境光通过IBL技术整合），其数学表达式为：

$f\_r(ω\_i,ω\_o)=f\_{diffuse}+f\_{specular}$

## **‌漫反射（Diffuse）‌**

* ‌**作用**‌：模拟光线在表面微结构中多次散射的均匀反射（如布料、粗糙墙面）。
* ‌**物理模型**‌：
  + ‌**Lambertian模型**‌：基础形式 $f\_{\text{diff}} = \frac{\text{albedo}}{\pi}$
  + ‌**改进模型**‌：Oren-Nayar（考虑表面粗糙度）或 Disney BRDF（艺术可控）
* ‌**能量守恒约束**‌：漫反射部分需满足：$∫\_Ωf\_{diff}(ω\_i⋅n)dω\_i≤1−F\_0$其中 $F\_0$ 是菲涅尔基础反射率。

## **‌镜面反射（Specular）‌**

基于‌**微表面理论**‌（Microfacet Theory），分解为三个物理项：

$f\_{spec}=\frac{F(θ\_h)⋅D(α,θ\_h)⋅G(α,θ\_i,θ\_o)}{4⋅(n⋅ω\_i)⋅(n⋅ω\_o)}$

* ‌**法线分布函数 NDF**‌
  + ‌**作用**‌：描述微表面法线朝向的统计分布（决定高光形状）。
  + ‌**常用模型**‌：
    - ‌**GGX/Trowbridge-Reitz**‌：$D(h) = \frac{\alpha\_g^2}{\pi [(n \cdot h)^2 (\alpha\_g^2 - 1) + 1]^2}$（`α`=粗糙度，`h`=半角向量）
    - ‌**Beckmann**‌：较早的物理模型，拖尾效果不如GGX真实
* ‌**几何遮蔽函数 G**‌
  + ‌**作用**‌：模拟微表面间阴影和遮挡（如粗糙表面的光能损失）。
  + ‌**Smith模型**‌：$G = G\_1(\omega\_i) \cdot G\_1(\omega\_o)G\_1(\omega) = \frac{n \cdot \omega}{(n \cdot \omega) (1 - k) + k}$（`k`=粗糙度重映射参数）
* ‌**菲涅尔项 F**‌
  + ‌**作用**‌：计算不同视角下的反射率变化（如掠射角反射增强）。
  + ‌**Schlick近似**‌：$F(\theta) = F\_0 + (1 - F\_0)(1 - \cos\theta)^5$（$F\_0$=基础反射率，金属≈0.5-1.0, 非金属≈0.02-0.05）

---

## **‌环境光的处理（间接光照）‌**

传统“环境光”在BRDF中被升级为 ‌**IBL Image-Based Lighting**‌：

* ‌**漫反射环境光**‌：通过‌**辐照度贴图Irradiance Map**‌ 预计算半球积分$L\_{diff}=albedo⋅\frac1π∫\_ΩL\_i(ω\_i)(n⋅ω\_i)dω\_i$
* ‌**镜面反射环境光**‌：
  + 预过滤环境贴图（Prefiltered Environment Map）
  + 重要性采样 + BRDF LUT（查找表）

---

## **‌与传统光照模型的对比‌**

| 光照成分 | 标准光照模型 | BRDF实现方式 |
| --- | --- | --- |
| ‌**漫反射**‌ | $Lambert = k\_d \* (n·l)$ | 能量守恒约束的`Oren-Nayar/Disney`模型 |
| ‌**高光反射**‌ | $Phong = k\_s \* (v·r)^n$ | 微表面模型（D+F+G项物理计算） |
| ‌**环境光**‌ | 恒定或环境贴图采样 | IBL技术（辐照度图+预过滤镜面贴图） |
| ‌**能量守恒**‌ | 不保证（可能过曝） | 强制满足`diffuse + specular ≤ 1` |

---

## **‌BRDF在渲染管线中的实现流程（以GGX为例）‌**

```
|  |  |
| --- | --- |
|  | hlsl |
|  | // Unity URP 中的核心代码片段（简化版） |
|  | float3 BRDF_PBS(float3 albedo, float metallic, float roughness, |
|  | float3 N, float3 V, float3 L) { |
|  |  |
|  | // 计算基础参数 |
|  | float3 H = normalize(V + L); |
|  | float NdotV = saturate(dot(N, V)); |
|  | float NdotL = saturate(dot(N, L)); |
|  |  |
|  | // 1. 菲涅尔项 (F) |
|  | float3 F0 = lerp(0.04, albedo, metallic); // 基础反射率 |
|  | float3 F = FresnelSchlick(saturate(dot(H, V)), F0); |
|  |  |
|  | // 2. 法线分布 (D) |
|  | float D = NDF_GGX(roughness, N, H); |
|  |  |
|  | // 3. 几何遮蔽 (G) |
|  | float G = GeometrySmith(roughness, NdotV, NdotL); |
|  |  |
|  | // 4. 组合镜面反射 |
|  | float3 nominator = D * G * F; |
|  | float denominator = 4 * NdotV * NdotL; |
|  | float3 specular = nominator / max(denominator, 0.001); |
|  |  |
|  | // 5. 漫反射 (能量守恒) |
|  | float3 kD = (1 - F) * (1 - metallic); // 金属无漫反射 |
|  | float3 diffuse = kD * albedo / PI; |
|  |  |
|  | return (diffuse + specular) * NdotL; |
|  | } |
```

---

# **‌关键突破‌**

* ‌**物理正确性**‌：通过微表面理论和能量守恒避免人工调参的不真实感。
* ‌**材质统一性**‌：参数（金属度/粗糙度）在所有光照环境下保持一致性。
* ‌**环境响应**‌：IBL使物体自然融入环境光照（如金属反射周围景物）。

> ‌示例对比‌：传统Phong模型在粗糙金属表面会产生圆形高光，而GGX BRDF会生成拖尾式高光（符合真实相机拍摄效果）。

---

> [【从UnityURP开始探索游戏渲染】](https://github.com):[闪电加速器](https://shandianjiasu.com)**专栏-直达**

（欢迎*点赞留言*探讨，更多人加入进来能更加完善这个探索的过程，🙏）
