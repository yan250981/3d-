# 雨夜便利店街角 · 三维微缩景观

三渲二（卡通渲染）风格的日式雨夜街角小场景：一块 4×4 的方形底座，便利店、街道、街角道具全部搭建其上，可当作收藏级微缩模型自由旋转观看。

## 交付物

| 文件 | 说明 |
|---|---|
| `rainy_konbini_hero.png` | 主视角成图（边长 1500px） |
| `rainy_konbini_storefront.png` | 店头特写：玻璃窗与店内陈列 |
| `rainy_konbini_alley.png` | 背侧视角：小巷、空调外机、霓虹竖招 |
| `rainy_konbini.glb` | 可交互三维模型（3.8 MB，1516 对象 / 14.6k 面 / 368 段动画） |
| `viewer_offline.html` | **单文件离线查看器（6.6 MB）：双击即开，不需要本地服务** |
| `viewer.html` | 网页查看器（依赖同目录 vendor/），需本地服务打开 |
| `../rainy_konbini.blend` | Blender 源文件（含三台相机、灯光、全部动画关键帧） |

查看器有两种开法：**`viewer_offline.html` 直接双击**（three.js 与模型全部内嵌，无需服务）；或 `viewer.html` + 同目录 `vendor/`（three.js r160），后者必须通过本地服务打开（如 `python -m http.server`），双击 file:// 会被浏览器的跨域策略拦住。

> 离线版原理：把 three.js / OrbitControls / GLTFLoader / BufferGeometryUtils 与 GLB 全部内嵌，
> 运行时用 `Blob` 造出 `blob:` URL 动态 `import()`，并把 addon 里 `from 'three'` 的裸标识符
> 改写成 three 的 blob URL，从而绕开 ES module 在 file:// 下的加载限制。
> 由 `scene/make_offline.py` 生成，改动 `viewer.html` 后重跑即可。

## 场景构成

**底座**：4×4 方形基座，厚 0.18，顶面下沿一圈黄铜饰边，正面嵌金属铭牌（RAINY CORNER — KONBINI）。路面为 L 形沥青，人行道 L 形抬高 0.05，路缘石、黄色盲道、排水篦子、井盖齐备。

**便利店**：1.6×1.6，净高 1.12。两面临街大玻璃（西、南），南侧为双扇自动门 + 绿色雨棚 + 门垫；檐口为奶白色灯箱，蓝/绿品牌色条 + "24H MART" 字牌；东南角挂粉白霓虹竖招；屋顶有女儿墙、栏杆、空调外机、通风管、检修口、爬梯与落水管。

**店内**（透过玻璃清晰可见）：6 组天花灯箱、3 排双面货架（每层密排彩色商品）、北墙冷藏饮料柜（含冷光背板与瓶装饮料）、东侧收银台（收银机、咖啡机、关东煮）、西窗饭团/便当展示柜、南窗杂志架、热食保温柜、吊挂促销牌、墙面海报、后场门与储物柜、地砖缝导视。

**街角**：电线杆（横担、瓷瓶、变压器、三根带垂度电线）、路灯、路牌、人行信号灯、白色护栏、自动贩卖机、自行车、伞架与四把伞、三分类垃圾桶、A 字立牌、公告栏、盆栽、排水沟、斑马线、停车位与一辆轻型车、小巷（铁门、空调外机、周转箱、垃圾袋、壁灯、锁链）、远侧信号灯。

## 风格与技术

- **渲染**：EEVEE，1500×1125，48 samples；Standard 色彩空间；曝光 +0.12；画面无 UI。
- **三渲二**：漫反射 → Shader to RGB → 三档硬阶梯 → 乘固有色 → 自发光输出；室内单独一套更亮、更暖的阶梯，室外调色板整体压向冷蓝，形成"暖室内 / 冷夜色"的对比。
- **描边**：反向外壳（复制网格 → 沿法线外扩 3–15mm → 翻转面朝向 → 单材质 + 背面剔除）。注意 EEVEE 不认 Solidify 的 `material_offset`，外壳会沿用原材质导致描边完全不可见，因此改为生成独立描边对象，随模型一同导出。
- **泛光**：合成器 Glare(Bloom)，阈值 1.05，只让灯箱、霓虹、窗内暖光溢出，阈值卡高避免整图起雾。
- **湿地面**：低粗糙度沥青 + 屏幕空间反射，店头暖光在积水与路面上拉出光带。
- **阴影**：默认 512MB 阴影池在千级对象下会溢出（表现为阴影缺失），已调到 1024MB。

## 动画（第 1–180 帧循环，24fps）

持续降雨（330 条雨丝各自循环下落）· 屋檐滴水与落地涟漪 · 积水波纹扩散 · 玻璃雨水痕 · 自动门偶尔开合 · 招牌灯箱轻微闪烁 · 人行信号灯红/绿变化。

## 复现

```bash
# 构建场景（生成 .blend）
blender --background --python scene/build.py
# 出图
blender --background scene/rainy_konbini.blend --python scene/render.py -- out.png 1500 48 cam_main
# 导出 GLB
blender --background scene/rainy_konbini.blend --python scene/export_glb.py -- out.glb
```

`scene/common.py` 为材质与几何工具库，`scene/build.py` 为场景搭建脚本，`scene/cams.py` 可试机位。
