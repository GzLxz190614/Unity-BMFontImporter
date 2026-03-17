# 中文版

## 概述
BMFontImporter.cs 是一个用于从 AngelCode BMFont 格式的 .fnt 文件中导入自定义位图字体的 Unity 脚本。该脚本特别适用于制作字符数量庞大的字体（例如 CJK 字体），并作为对 Unity Default Font 系统的扩展方案，通过预先生成位图字体数据来克服动态字体在处理大量字符时的性能和内存限制。在 Unity 5.6.4f1 中，脚本解析外部 BMFont 数据，计算每个字符的 UV 坐标、偏移及基线补正，并将结果填入 CustomFont 资产的 Character Rects 字段，从而大幅减少手工配置的工作量。

> **如果您计划替换已发布游戏中的字体（例如进行第三方本地化工作），请先阅读下方“替换已经发布的游戏中的字体”部分。**

## 需要准备的内容
1. 位图纹理文件（通常为 .dds 或 .png）和 AngelCode BMFont 字体索引文件（可通过 [Bitmap Font Generator](https://www.angelcode.com/products/bmfont/) 生成）。
2. 已安装的 Unity Editor。
3. 下载仓库中的 BMFontImporter.cs 文件至本地。

## 获得成品
在 Unity Editor 中可使用的自定义 Unity Default Font 位图字体。

## 使用步骤

### 1. 创建项目并导入资产
1. 创建并打开一个 Unity 项目。
2. 将 BMFontImporter.cs、生成好的 .fnt 文件以及对应的纹理文件导入到 Assets 文件夹中。
3. 在 Assets 文件夹中，右键选择 **Create → Custom Font** 创建一个新的 CustomFont 资产，并命名。
4. 在 Assets 文件夹中，右键选择 **Create → Material** 创建一个新的 Material。
5. 在 Hierarchy 面板中，选择 **Create → Create Empty** 创建一个空 GameObject（推荐命名为 FontManager），然后在 Inspector 面板中通过 **Add Component** 或拖拽的方式将 BMFontImporter.cs 脚本添加到该对象上。
6. 在 Hierarchy 面板中，点击刚刚创建的GameObject，再右键 **Create → 3D Object → 3D Text** 新建一个 Text 对象。

### 2. 调整资产属性
1. **纹理设置**  
   - 在 Assets 文件夹中选中纹理文件，在 Inspector 中确认：  
     - **Read/Write Enable** → 勾选（对于替换字体来说非常重要）  
     - **Wrap Mode** → 设置为 Clamp（或与原始文件相同）  
     - **Filter Mode** → 设置为 Bilinear（或与原始文件相同）  
   - 设置完成后，点击 **Apply**。
2. **材质设置**  
   - 选中创建的 Material，在 Inspector 中设置：  
     - **Shader** → 选择 Unlit/Transparent  
     - 将纹理文件拖拽至 Material 的 None (Texture) 区域进行绑定。
3. **字体设置**  
   - 选中创建的 CustomFont 资产，在 Inspector 中设置：  
     - **Line Spacing** → 设置为与所需字体大小相同（或与原始文件一致）（多少px就写多少）  
     - 将创建的 Material 绑定到字体上。
4. **Text 对象设置**  
   - 在 Hierarchy 中选中创建的 Text 对象，在 Inspector 中：  
     - 在下方的 Add Component 添加 Rect Transform
     - 将 Rect Transform 的 Height 设定为所需字体大小（便于检查文字位置）  
     - 在 Text 组件的 Text 字段中输入测试字符串（建议包含 CJK 字符）  
     - 将 CustomFont 资产分配到 Text 组件的 Font 字段。
5. **BMFontImporter 脚本设置**  
   - 在 Hierarchy 中选中 FontManager GameObject，在 Inspector 中将：  
     - CustomFont 资产分配到 **Custom Font** 字段  
     - 纹理文件分配到 **Font Texture** 字段  
     - .fnt 文件分配到 **Fnt File** 字段。

### 3. 应用脚本
1. 使用 Ctrl+P 快捷键或点击播放按钮运行场景。脚本将解析 .fnt 文件，并将字符数据填入 CustomFont 的 Character Rects 字段。
2. 若 Scene 面板中以新字体显示测试字符串，则说明导入成功。

***如果您计划替换已发布游戏中的字体（如进行第三方本地化工作），请继续阅读下方“替换已经发布的游戏中的字体/测试字体、构建临时游戏并导出字体”部分。***

## 替换已经发布的游戏中的字体

> 本部分面向以下情况：  
> 1. 您确定需要替换的字体为位图格式的 Unity Default Font。  
> 2. 或者您发现替换 TrueType 字体文件后无效，或者根本不存在 TrueType 文件；又或者通过 AssetStudio 预览时发现该字体带有纹理图片，但并不符合 TextMeshPro 字体的特征（如无相应 MonoBehaviour）。此时，本方案可能更适合您。  
> 3. 在这种情况下，我们将采用本项目介绍的方法生成所需字体，并制作相应文件，以便替换已发布游戏中的资源。  
> 4. 整体思路为：先提取原有字体相关信息，再选择功能和属性相似的字体，通过导入到原游戏资源中实现替换。通常需要借助 Unity Editor 制作临时测试项目，对各资源进行预处理。

### 需要准备的内容
1. 您希望导入到游戏中的 TrueType 字体文件。
2. 通过 [AssetStudio](https://github.com/Perfare/AssetStudio/)（或其他更新分支如 [zhangjiequan/AssetStudio](https://github.com/zhangjiequan/AssetStudio/)）查看原字体对应的 Texture2D（纹理图片），并记录以下属性：
   - Format（格式）
   - Filter Mode（过滤模式）
   - Wrap Mode（拉伸模式）
   - Channels（通道）
3. 观察确认原字体纹理的配色类型：
   - 白字＋不透明黑色背景  
   - 黑字＋不透明白色背景  
   - 白字＋透明背景  
   - 黑字＋透明背景
4. 通过 [UABEAvalonia](https://github.com/nesrak1/UABEA) 等工具，以 Export Dump 方式导出原字体的 Font 与 Texture 文本文件（若导出多个字体，建议按字体名称分别存放于不同文件夹中）。
5. 下载位图字体制作工具 [Bitmap Font Generator](https://www.angelcode.com/products/bmfont/) 至本地。
6. 下载仓库中的 CustomFontConfig.bmfc 配置文件至本地。
7. （可选）若您希望生成的字体仅包含 TrueType 字体文件中的部分字符（例如常用汉字），请准备包含这些字符的文本文件（要求以 UTF-8 BOM 或 UTF-16 BOM 格式编码）。

### 步骤
1. **安装字体文件**  
   将目标 TrueType 字体文件安装到您的计算机上。若系统中存在多个字重，请暂时卸载不需要的版本，仅保留目标字重，以确保 Bitmap Font Generator 能正确识别目标字体。
2. **加载配置**  
   打开 Bitmap Font Generator，依次点击 **Options → Load Configuration**，选择 CustomFontConfig.bmfc 导入配置文件。
3. **设置字体参数**  
   点击 **Options → Font Settings**，确认并调整以下设置：
   - **Font** → 更换为目标字体  
   - **Size (px)** → 更换为目标字体大小（单位：像素）  
   - **Match char height** → 建议保持勾选  
   - **Super sampling** → 保持勾选，level 设为 4  
   - 其他选项保持默认或根据需要调整。完成后点击 **OK** 保存。
4. **设置导出选项**  
   点击 **Options → Export Options**，确认以下设置：
   - Padding 已设为 2（若生成后边缘出现杂色，可适当增大 Padding 值）  
   - **Width** 与 **Height** 根据实际需求调整；注意 Unity 5 等旧版本中纹理最大支持 8192×8192 像素  
   - **Bit depth** → 保持为 32，除非有特殊需求  
   - 根据原有文件情况，调整 Presets 中的通道设置：  
     - 白字＋透明背景 → White text with alpha  
     - 黑字＋透明背景 → Black text with alpha  
     - 白字＋不透明黑色背景 → White text on black (no alpha)  
     - 黑字＋不透明白色背景 → Black text on white (no alpha)
   - **File/Font** 选择 Text 格式  
   - 根据记录的原文件 Format 参数，设置 Textures 与 Compression（例如，若为 DXT1，则选择 dds 格式与 DXT1 压缩）。完成后点击 **OK** 保存。
5. **（可选）导入部分字符**  
   若仅生成部分字符的字体，可通过 **Edit → Select Chars From File** 导入包含目标字符的文本文件（要求编码为 UTF-8 BOM 或 UTF-16 BOM）。
6. **预览字体效果**  
   在 Bitmap Font Generator 中，通过左侧的 Unicode 查看面板和右侧字符选择面板，调整或检查字体情况。确认无误后，点击 **Options → Visualize** 预览字体纹理。  
   
   > 预览窗口标题应为 “Preview: 1/1”，表示仅生成了一张纹理。若生成多张纹理，则说明单一纹理不足以容纳所有字符，此时需返回 **Options → Font Settings** 调整纹理尺寸，或减少字符数量。
7. **保存生成的字体**  
   关闭预览窗口后，点击 **Options → Save bitmap font as...**，保存生成的纹理文件和 .fnt 字体索引文件。
8. **基线补正**  
   为确保正确显示效果，请手动校正基线。使用文本编辑器打开生成的 .fnt 文件，将 `base=` 后的值（整数）修改后保存。  
   
   > 我的经验表明，适当的 base 值大约为 lineHeight 的 0.2 倍（或四舍五入后的值）。
   >
   > 调整base是为了让游戏在正确的位置显示文字，不调整的话会出现文字比原来的位置高的情况
9. 完成上述步骤后，您将获得所需的字体纹理文件和 .fnt 字体配置文件，请继续阅读上方的“需要准备的内容”部分，进行后续操作。

## 测试字体、构建临时游戏并导出字体
### 获得成品
通过 Export Dump 方式导出的新字体的Font 和 Texture 2D 文本文件。

### 步骤
#### 1. 检查文字在 Text 中的位置
1. 在 Hierarchy 面板中选中已创建的 Text 对象，检查 Inspector 中 Text 组件显示的文本是否位于灰色边界框（Rect Transform）内，并且垂直居中。
2. 如果发现字符整体位置偏高，则尝试在 .fnt 文件中减小 `base=` 值；如果偏低，则需要增大该值。
3. 为了更精确地调整，您可以放大 Scene 面板的视野，仔细测量并记录所需偏移量，然后进行相应调整。

#### 2. 保存项目并构建临时游戏
1. 首先，通过 Ctrl+S 或菜单栏 **File → Save Project** 保存当前项目，确保所有修改均已存盘。
2. 然后，打开菜单栏中的 **File → Build Settings**，选择目标平台（例如 PC、Mac & Linux Standalone）。
3. 点击 “Add Open Scenes” 将当前场景添加到 Build 列表中。
4. 最后，点击 “Build” 按钮，选择一个保存路径，构建临时游戏。

#### 3. 导出创建的字体
1. 打开构建好的临时游戏的 `GameName_Data` 文件夹。
2. 使用 UABEAvalonia 等工具打开临时游戏中的 `resources.assets` 文件。
3. 以 Export Dump 方式导出新创建的字体的 Font 和 Texture 2D 文本文件（注意不要与原有字体混淆）。
