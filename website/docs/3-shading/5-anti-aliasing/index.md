---
title: 3.5 延迟着色中的反走样技术
---

我们在第1章介绍了对数字信号采样的基础知识，以及走样的概念，并在第[ref sec:intro-msaa]节简要介绍了图形学渲染领域的两种基本全屏反走样技术：超采样反走样和多重采样反走样，本节我们将深入介绍全屏反走样的一些流行技术。

首先我们回顾一下走样（aliasing）及反走样（anti-aliasing，AA）的基本概念。一个连续信号在被采样为离散信号时，只有在其对应的频率域（frequency domain）上，使用能够覆盖2倍及以上的频率带宽（frequency bandwidth）的采样率（sample rate）时，离散信号才能被完美重建（reconstruction）为原始连续信号，否则重建后的原始信号将会发生走样，由于一些高频信号被“削平”了，许多原本不同的信号看起来像相同的信号（一些信号像另一些信号的别名）。由于在场景中，几何物体的边缘周围的颜色频率域变化非常大，基本上一般的屏幕分辨率根本无法捕捉这些频率域的变化范围，所以走样几乎不可避免。另一些图形学中的走样发生于着色器中对高频（如高光）函数进行采样，更多的信息详见第（1）章第（1.4）节的内容。

:::tip[aliasing]
	关于aliasing，国内有多种翻译：锯齿，混淆和走样。alias在英语中表示“别名”的意思，可以是正确（例如小名）或错误（例如间谍冒充别人）的别名，aliasing是采样不足导致的多个信号之间无法区分（其中一个像另一个的别名）的现象，只有在图形学中以单个值表示一个像素块的值时才表现为锯齿，锯齿对应的词通常为jaggies。aliasing是可以减弱的，而jaggies是不可消除的，anti-aliasing之后的图像放大之后仍然有锯齿，但是这些信号之间的过度更平滑，所以笔者认为“走样”的翻译更精确一些，而混淆太语义化了，不利于理解概念。
:::

在图形学中主要有两大类走样类型: 其一是由于对空间域的信号采样不足导致的走样，这些称为空间走样（spatial aliasing），它包括几何走样（geometry aliasing），着色走样（shader aliasing）等，空间走样对应的反走样技术称为空间反走样（spatial anti-aliasing），相应使用的过滤器称为空间过滤器（spatial filter）；另一种走样是由于帧率的限制，导致对相邻时间内信号的采样不足导致的走样，称为时间走样（temporal aliasing），对应的反走样及过滤器称为时间反走样（temporal anti-aliasing）和时间过滤器（temporal filter）。

在图形学中，空间走样的黄金标准是全屏超采样（supersampling）技术，它直接使用更高的分辨率对场景进行渲染，然后对这种高分辨率的渲染结果进行过滤以“缩放到”屏幕分辨率，使用超采样进行反走样的计算称为超采样反走样（supersample anti-aliasing，SSAA）或全屏反走样（full-screen anti-aliasing，FSAA）。

由于渲染中使用的着色计算可能非常复杂，而考虑到屏幕中最主要的走样类型为几何走样，因此多重采样（multisampling）技术将几何可见性和着色计算分开，它对几何可见性函数的采样使用类似超采样一样更高的分辨率进行采样，但是每个像素点内只做一次着色计算，并使用可见性的覆盖率作为比率来混合当前的颜色值，这样的反走样技术称为多重采样反走样（multisample anti-aliasing，MSAA）。MSAA将每个像素点的着色计算结果分别复制到每个子采样点，所以它使用和SSAA一样大小的存储空间，但是计算量大大降低，不过缺点是不能处理着色走样。

然而MSAA并不适用于延迟着色，因为它会自动将片元着色器输出的结果拷贝多次，这使得本身就非常大的G-buffer更要使用多几倍的存储空间，这又大大增加了延迟着色计算中的带宽占用，因为每个像素要读取多几倍的数据。

另一方面，即使G-buffer的存储和带宽都不是问题，由于MSAA着重于解决几何走样，它仍然无法解决着色器走样。而现代游戏引擎大都使用基于物理的渲染管线，着色器中对函数的采样不足导致的高光部分的走样特别明显，这使得我们必须寻找新的更有效的反走样计算。

本节我们将讨论三种被广泛用在延迟着色中的反走样技术，这三种技术几乎也是目前工业中最流行的反走样技术，它们分别使用几乎完全不同的思路和原理进行反走样处理。







