# (第七部分。)Unity 中的 Limbo 风格菜单

> 原文：<https://itnext.io/limbo-style-menu-in-unity-4fe3fdee3420?source=collection_archive---------2----------------------->

在本教程中，我将复制 2010 年著名游戏《地狱边缘》的菜单风格:

![](img/fb84987a4b02c8d48e488c37db402ad0.png)

(上一部分教程[这里](/killing-enemies-with-telekinetic-bricks-in-unity-gamedev-tutorial-part-6-4d43874eae29))。

下载大气图像，例如从 unsplash 下载:

![](img/a5ba37d06f05236a59ee9213a6ce4ef1.png)

在 Unity 中创建新场景:

![](img/eb15cceb25c2f7bb513762404840aee0.png)

将图像用户界面添加到场景中

![](img/f1a04bb7f8138e6c75e941fbfcd5de1a.png)

导入图像并将纹理类型更改为 UI:

![](img/3d3bd3bda70c4a80035d175a739882dd.png)

将图像拖到源图像上:

![](img/33a70322142d435219a755bf1108c555.png)

更改锚点以扩展(按住 alt 并选择右下角的选项)

![](img/96bba43a4efe8a1ba17a87626f1a1194.png)

现在看起来是这样的(我增加了图像的对比度，转换成黑白):

![](img/307fb2b9b5a394d44b800a4a0aca6b9d.png)

点击画布，将用户界面缩放模式改为“随屏幕尺寸缩放”。改变你的目标分辨率。

![](img/95974efc8b17bcf1f7f69a0d09707093.png)

向画布添加一个新按钮(TextMeshPro)。

![](img/105732674457314b8cd505d9b4f25117.png)

在弹出窗口中导入 TMP:

![](img/a850e23eb44c8c13a7123c150e018716.png)

调整按钮的大小，并根据您的喜好更改值。

![](img/efbda3cb6bcd3ff1c520ac8a59ab8ba5.png)

在按钮中，删除图像组件:

![](img/be778db7c982b0a60907c692303eaa7a.png)

更改按钮的文本:

![](img/ace336810c6de98c43c9bdfb8edf24dd.png)

在按钮设置中，将动画/淡入淡出的目标图形更改为按钮的文本:

![](img/8cfb4b0b04438c375f9a6862b78c6bab.png)

使用高亮/聚焦/鼠标悬停选项。我将*的正常颜色*设置为浅灰色，将*的高亮颜色*设置为白色。

![](img/770a5f0ee76490b3f79c580b26ebf388.png)

将按钮重复几次，并添加更多菜单选项:

![](img/36a282325ae58e0952bd32d50ca358ab.png)

将标题文本添加到画布:

![](img/c96b673dac375e768e67b2dedc059934.png)

创建一个新脚本，它将保存菜单的逻辑:

![](img/80c1f9e1779fa7bc5a0f93f065b1c3f2.png)

右键单击画布对象，创建一个空对象，调用它的主菜单，并分配按钮给它。

![](img/9e872dca78165d7c8f2cf991727a442c.png)

将脚本附加到主菜单对象。

![](img/072e284178e8197fdbb3154934befff8.png)

我想让新的游戏按钮开始一个新的场景。为此我需要知道去现场的路。

转到文件->构建设置和场景你有“场景在构建”。我的*新游戏场景*是*样本场景。*

![](img/c51e25612c86f9178fa6f2b2f69e00f2.png)

在脚本中，使用 SceneManager 将场景加载到一个新函数中，稍后我们会将该函数与按钮挂钩。

![](img/b46e6482a2c4d958227efcdf932c03fa.png)

转到按钮检查器并添加一个新的 OnClick 事件。将 MainMenu 对象分配给它，以便它可以访问其功能:

![](img/9a39719158cf8511521100f61753e0eb.png)

选择 StartNewGame 功能(如果您希望能够在 Unity 编辑器中运行，也可以切换到*编辑器和运行时*):

![](img/e26d3a04d439d835f3922d2ab0fbeb09.png)

类似地，将功能分配给其他按钮(退出等。):

![](img/537531d4033bd3600fb547c196372c8a.png)![](img/52b0ce9d5cb63c4d6497d1056b87f2e5.png)

“设置”按钮需要另一个子菜单。复制主菜单对象，并重命名它及其子项目:

![](img/cfa9654055a3314e5a8e4d62fcd19fa9.png)

停用它以将其隐藏在场景中:

![](img/d4e3f3ca35b734fb410c0b2e7898eb44.png)

向设置按钮添加新的 *OnClick* 功能(OnClick 隐藏主菜单并显示设置菜单):

![](img/b6c97ccd929b46ff6a43c5947545839f.png)

在设置菜单中的后退按钮上，执行相反的操作:

![](img/f99ae50c62aca3c3cfa388c94944632c.png)

点击 play 后，你可以用鼠标、箭头键或游戏手柄控制菜单。

![](img/e7f58c908f15c44d8af5d7a68fe3ddad.png)

如果你刚刚开始学习如何开发游戏，你可以关注我的 udemy 初学者 Unity 开发课程:

[](https://www.udemy.com/course/make-a-3d-game-in-unity-2020-from-scratch-with-free-assets/?referralCode=8B96F6C67527AEEA39D9) [## 完整指南:Unity 2020 中的动作恐怖 3D 游戏

### 大家好，我叫 Jan Jileč ek，是一名拥有计算机科学硕士学位的专业游戏开发人员，我…

www.udemy.com](https://www.udemy.com/course/make-a-3d-game-in-unity-2020-from-scratch-with-free-assets/?referralCode=8B96F6C67527AEEA39D9)