---
title: 1 光与表面的交互
---

计算机图形学（computer graphics）的主要目标之一，就是模拟光与物体（表面或介质内）交互的光学原理，将一个虚拟的数字3D场景，利用计算机生成一张2D的数字图像，供屏幕显示使用或者保存为数字文件格式。

然而，要达到逼真的视觉效果，尤其在实时的环境下，这个渲染的过程却异常复杂。从技术上讲，这里有两个突出的困难需要解决：首先，对于表面上的每个点，我们必须在该点法线方向对应的半空间（hemisphere）上做积分运算以便求出该点的颜色值，因为每个点都可能接受来自场景中所有其他点反射的光照；其次，对于空间中传播的每一条光线（ray），我们必须遍历场景中所有物体表面上的每一个点，来计算出与该光线相交的最近的一个点的位置(即最近点测试)，从而进行光照计算。在当前的硬件条件下，我们几乎不可能实时处理这两个方面的计算。

为了解决这两个问题，计算机图形学提出了大量的方法和理论以简化和加速这两个方面的计算，从而更快（甚至实时）地进行光照计算以渲染场景。

我们可以将所有这些方法和理论分为两大类，其中第一类方法通过寻找更快的数学方法来比较精确地计算渲染积分方程（见第[sec:intro-the-rendering-equation]节），这方面有两大经典方法：即基于有限元分解的辐射度理论，以及基于蒙特卡洛积分的光线追踪技术。

有限元方法（finite element method，FEM）是一种常见的积分计算方法，它将一个沿无限维空间的积分分解为一个有限纬度(称为一个有限元)的积分, 我们只需要求出每个有限元的均值，便可以很容易地计算出渲染积分方程，在辐射度理论中，我们通常通过蒙特卡洛等方法预计算出每个有限元的均值，供渲染时渲染方程的实时计算，关于有限元方法详见本书第[chp:rad]章的内容；而基于蒙特卡洛（Monte Carlo）的方法则是通过随机采样, 使用有限个采样点的函数贡献值，来近似计算积分方程的值，这又称为对积分的估计，由于在渲染中一个这样的样本称为一条光线的路径，因此基于蒙特卡洛的方法又称为光线追踪技术，关于光线追踪技术详见本书第[chp:path-tracing]章的内容。

上述两种方法均能达到非常高的图像品质，然而它们的计算仍然十分耗时, 因此它们主要被用于影视动画等离线渲染领域，例如Pixar的RenderMan RIS就是基于光线追踪技术的渲染器产品。

与辐射度和光线追踪技术通过简化和加速积分方程求解不同，第二类方法的主要思路是根据光学现象或者其他理论拆分渲染方程，使得光照效果最终由多种效果叠加而成，这主要是因为光照是可以线性叠加的，这些拆分的效果包括直接光照（即光从光源出发经过物体表面反射或者折射一次后进入人眼的光照），（软）阴影，漫反射光照，光泽反射光照，间接光照，环境遮挡，次表面散射，环境光照等等。

这些拆分后的效果可能分别使用不同的方法来计算，其中大部分都是本书要重点讨论的内容，这些方法的一些主要思路包括但不限于：

- 使用GPU的光栅化特性，例如用来简化物体间的遮挡关系和阴影计算，这可以通过使用光栅化从光源的角度生成光照贴图来实现。
- 为了减少计算量，另一些技术从屏幕空间（screen space）进行计算，例如环境遮挡可以在渲染的后处理阶段，基于屏幕（而不是物体）空间计算像素的环境遮挡值，还有一些技术在屏幕空间进行光线追踪计算，这都大大减少了着色计算的数量。
- 对于一些静态的数据，例如静态光源和场景，我们可以在预处理阶段将某些光照的中间结果转换为纹理的形式，例如环境贴图和光照贴图等，而在运行时只需要使用简单的贴图即可；有时候一个复杂的积分方程的一部分静态数据也可以进行类似的预处理。 
- 有些算法则使用一些特定的数据格式来加速其中的一些计算，如最近点测试和遮挡范围计算；例如Unreal Engine 4在预处理阶段对整个场景构建一个距离场的数据结构，通过它来加速遮挡关系的计算，从而实现如软阴影和环境遮挡等效果。
- 对于被积函数中的某些低频部分（low-frequency），我们可以通过使用比如低阶的球谐函数等来近似这一部分，它们仅被存储为少量一些系数，从而能够快速地进行积分计算。
- 光会在场景中进行多次反射或折射后才会进入摄像机，模拟这种多次反弹（bounce）的效果计算量非常大，但是一些近似方法通过仅计算两次反弹（即光从光源出发，经过两次与表面反射或折射，然后进入摄像机）也能获得非常好的效果。
- ......

从这些众多看似复杂而零散的计算机图形学知识碎片中，我们可以看到其实它们是具有一定的“逻辑”的。从一个个完整的全局光照系统的角度，以及通过多个不同全局光照模型之间层层递进的关系，通过本书读者应该能够很轻易地去理解这些逻辑，从而站在一个更系统的角度去看待计算机图形学，最终能够更得心应手地解决工作和学习中的问题。