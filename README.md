# Mm ItemWheel

Unity 程序化物品/武器轮盘 Demo。扇环由代码绘制，不依赖扇形美术资源；支持编辑器一键生成预制体、鼠标悬停判定与 DOTween 高亮动画。

**环境：** Unity 6000.4.10f1  
**依赖：** DOTween、Odin Inspector、Input System（新输入系统）

---

## 设计结构

```
Item Wheel（根节点）
├── ItemWheelController      # 轮盘总控：指针解析、扇区切换
├── Sector_0
│   ├── RingDraw             # 程序化绘制扇环 mesh
│   ├── RingController       # 单扇区交互：OnEnter/OnExit 动画
│   └── Icon                 # Image，挂物品图标
├── Sector_1 …
└── CenterCircle
    └── CenterCircleDraw     # 中心圆，遮住内缘拼接棱角
```

### 分层职责

| 层级 | 目录 | 职责 |
|------|------|------|
| **Data** | `Assets/Scripts/Item Wheel/Data/` | 数据结构（`RingLayoutData` 等） |
| **View** | `Runtime/View/` | 纯绘制：`RingDraw`、`CenterCircleDraw` |
| **Logic** | `Runtime/Logic/` | 运行时逻辑与交互 |
| **Editor** | `Editor/` | 预制体生成、判定区 Scene 可视化 |

### 核心脚本

| 脚本 | 作用 |
|------|------|
| `ItemWheelEditorWindow` | 菜单 **Tools → Mm ItemWheel**，调参并生成 `Item Wheel.prefab` |
| `ItemWheelController` | 每帧解析鼠标所在扇区，派发 `IRingBehaviour` |
| `ItemWheelPointerResolver` | 屏幕坐标 → 极坐标 → 扇区 index |
| `RingController` | 实现 `IRingBehaviour`，DOTween 外扩缩放 + 高亮 |
| `RingDraw` | `MaskableGraphic`，`OnPopulateMesh` 画楔形扇环 |

### 设计要点

- **视觉与判定分离**：扇环长什么样由 `RingDraw` 管；鼠标落在哪由 `ItemWheelPointerResolver` 用极坐标算，可用 `expandedInnerRadius` / `expandedOuterRadius` 扩大响应区（不必精确划过灰色扇环）。
- **一扇区一 GameObject**：每个 `Sector_i` 的 Rect 是整圆外接方框，真正形状由 mesh 决定。
- **角度逆时针为正**：0° 在正右，与 `RingDraw` 绘制一致。

---

## 如何使用

### 1. 生成预制体

1. 菜单打开 **Tools → Mm ItemWheel**
2. 调整同心圆、扇环个数、颜色、描边、图标尺寸/朝向等
3. 底部预览可实时查看效果
4. 点击 **生成 Item Wheel 预制体**
5. 输出路径默认：`Assets/Scripts/Item Wheel/Prefab/Item Wheel.prefab`

生成物已自动挂载 `ItemWheelController`、各扇区 `RingController` 与子物体 `Icon`。

### 2. 放入场景

1. 将 `Item Wheel.prefab` 拖入 **Canvas**（建议 Screen Space 场景）
2. 确保场景有 **EventSystem**（若仅用 Controller 判定可不依赖 UI 射线）
3. 打开示例场景：`Assets/Scripts/Item Wheel/WheelSample.unity` 可参考布局

### 3. 配置物品图标

- 在预制体中展开 `Sector_i/Icon`，给 **Image → Source Image** 赋 Sprite  
- 或运行时通过代码设置（需在 `RingController` 上扩展 `SetItemIcon` 并由总控调用）

### 4. 调整响应范围

选中根节点 **Item Wheel**，在 `ItemWheelController` 上：

| 参数 | 含义 |
|------|------|
| **扩展响应区域内距离** | 从视觉内径再往里扩；设为内径数值 ≈ 从圆心楔形都可响应 |
| **扩展响应区域外距离** | 从视觉外径再往外扩 |

Scene 视图选中该物体时，**青色/橙色圈**为判定内外径（见 `ItemWheelControllerEditor`）。

### 5. 自定义扇区行为

实现 `IRingBehaviour` 接口（`OnEnter` / `OnExit`），挂在扇区上；默认 `RingController` 已提供缩放与高亮。

---

## 目录速览

```
Assets/Scripts/Item Wheel/
├── Data/           RingLayoutData.cs, ItemWheelData.cs
├── Editor/         ItemWheelEditorWindow.cs, ItemWheelControllerEditor.cs
├── Prefab/         Item Wheel.prefab
├── Runtime/
│   ├── Logic/      ItemWheelController, ItemWheelPointerResolver, RingController
│   └── View/       RingDraw, CenterCircleDraw
└── WheelSample.unity
```

---

## 插件说明

本仓库包含 **DOTween**、**Odin Inspector** 插件文件。若公开分发，请遵守各自授权；协作者也可自行安装插件后从版本中排除 `Assets/Plugins/`。

---

## 仓库

https://github.com/Haki-sheep/MmItemWheel
