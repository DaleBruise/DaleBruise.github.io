# 相机标定流程和相关原理
相机标定总的来讲，就是将由于相机本身的特性——“鱼眼效应”造成的畸变现象通过一定的方法修正过来。并且，我们还可以补偿相机外界旋转造成的偏移。下面将会介绍相机标定的原理和我们的解决方法, 本文章是介绍针对类似小孔成像面阵相机的标定进行介绍。 
## 1、相机畸变
为什么我们要相机标定？难道相机看到的东西，不就是我们想要的东西了吗？其实我们在一般的图像处理中是不需要进行标定流程的，只有我们需要将图像中的位置与现实中的位置对应起来以后，才需要进行一系列标定流程，相机标定只是我们标定流程中的一部分。根据项目的不同，我们需要走的标定流程不同，但是可以确认的是，相机标定是大概率被需要的，是在全套标定流程中简单但至关重要的一环，也就是相机的畸变修正。  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/pinHole.png"/>  
</div>  

对于小孔成像模型（如上图所示），一般的镜头畸变是由于透镜存在球面像差（由于透镜或者镜子的球形形状引起的像差，使得中心光线和边缘光线有不同的焦点）导致的。比如，我们投影一个直线时，越靠近边缘的部分，直线的弯曲程度就会越大。类似地，根据镜头不同的光学特性，就会有不同的畸变现象。      
### 1.1、径向畸变
我们经常遇到的畸变类型为径向畸变，径向畸变的特性就是从图像中心点出发，与靠近边缘的位置现象越明显，其大致可以分为桶型畸变和枕型畸变。  
 - **桶型畸变**中，图像的放大倍率随距主光轴（不偏移）的距离而减小，类似将一个图像投影到一个桶型或者球形的物体上。鱼眼镜头采用半球形视角，利用这种畸变可以扩大我们的视野范围，牺牲的则是物体在视野中的形变。  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/barrel_distortion.png" width="192" height="193">  
</div>  

 - **枕型畸变**中，与桶型畸变相反，图像的放大倍数随着距主光轴（不偏移）的距离而变大，没有穿过图像中心的线条会向内弯曲。凸球面透镜往往就会产生枕型畸变。  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/pincushion_distortion.png" width="192" height="193">  
</div>  

如果我们想要使用数学语言去表达此类畸变的程度，那么一般来讲，我们可以使用二次多项式来表达，他们随着主光轴的距离的平方次增加。后面就会讲到我们如何使用数学方法来补偿畸变带来的误差。
### 1.2、切向畸变
切向畸变是由于感光元件和镜头不平行导致的，我们在程序中进行畸变矫正的过程中，一般不考虑切向畸变。
## 2、畸变矫正方法
相机畸变的矫正不需要我们去移动平台或者其他平面，根据张正友提出的方法，对于大多数的情况，我们只需要一张棋盘格图像即可矫正，如下图所示。  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/chessboard.png" width="210" height="143">  
</div>  

棋盘格所在平面要和镜头镜头尽量平行，也就是棋盘格所在平面的Z轴要尽量与镜头所在平面垂直，从而我们的精度可以更高。搭建好平台后，我们就可开始对相机进行建模，相机可以视为小孔成像模型，在此模型下，我们使用一个变换矩阵来将实际的坐标点投影到屏幕上：  

$$ sm' = A[R|t]M' $$  

或者   

$$s\begin{bmatrix}
u \\ 
v \\ 
1 \end{bmatrix}
= \begin{bmatrix} f_x & 0 & c_x \\ 
0 & f_y & c_y \\
0 & 0 & 1 
\end{bmatrix}
\begin{bmatrix} r_{11} & r_{12} & r_{13} & t_1 \\
r_{21} & r_{22} & r_{23} & t_2 \\ 
r_{31} & r_{32} & r_{33} & t_3 \end{bmatrix}
\begin{bmatrix} X \\ 
Y \\ 
Z \\
1 
\end{bmatrix}
$$  

其中：
  - $(X,Y,Z)$ 是实际坐标点
  - $(u,v)$ 是投影到图像坐标系下的坐标点
  - 矩阵 $A$ 中保存的信息是相机的内参
  - $(c_x, c_y)$ 是图像中心坐标
  - $f_X,f_y$ 是以像素为单位的焦距
如果我们想要将模型应用在变焦镜头中，由于 $f_x,f_y$ 改变了，所以对于不同焦距的镜头，我们需要进行不同的建模。反之，如果仅仅只是对图像的等比例放大和缩小，那么这个模型就可以反复使用不需要额外改变。
根据我们上述描述的相机模型，我们可以使用以下流程来对相机畸变的修正：

 1. 世界坐标系(WCS)->相机坐标系(CCS)
 2. 相机坐标系点->图像坐标系点(IPCS)
 3. 图像坐标系点->应用相机径向畸变后的图像坐标系点
 4. *应用相机径向畸变后的图像坐标系点->应用相机切向畸变后的图像坐标系点（选择性）
 5. 应用相机畸变后的点->真实图像坐标系下的点

### 2.1、世界坐标系点->相机坐标系点
旋转矩阵 $[R|t]$ 是相机的外参矩阵，该矩阵是用来描述相机外部的旋转情况。也就是说，该矩阵将点 $(X,Y,Z)$ 转换为相机固定的坐标系。其中，在***OpenCV***方法***cameraCalibration***中我们会得到的外部参数为：  

$$  
R=\begin{bmatrix} \alpha & \beta & \gamma \end{bmatrix},
t = \begin{bmatrix} t_x & t_y & t_z \end{bmatrix}
$$  

若我们在使用时，额外输入了相机的外部参数的话，就会使用如下方式使用相机的外部参数：  

$$
\begin{bmatrix} x \\
y \\
z
\end{bmatrix} = R \begin{bmatrix} X \\
Y \\
Z
\end{bmatrix} + t
$$

至此，我们就可以得到了在相机坐标系下的点。
### 2.2、相机坐标系点->图像坐标系点
下一步即是将相机坐标系下的点投影到图像坐标系下，这是因为 $2D$ 平面下的点集更方便我们去处理。但其实实际处理过程中，由于我们的平台是和相机镜头所在平面尽可能平行，所以我们所有角点的 $z$ 方向上的值可以设为1。从而可以将投影步骤复杂度降低。并且，对于上述处理相机坐标系下的点时，我们也会将 $Z$ 值设为0，来假设我们的标定面垂直于Z轴（相机中轴线）。
对于小孔成像的相机模型，我们的变换公式为：  

$$
\binom{u}{v} = \frac{f}{z}\binom{x}{y}
$$

然而对于远心镜头，那么我们的焦距要添加一个负号：  

$$
\binom{u}{v} = \frac{-f}{z}\binom{x}{y}
$$

### 2.3、图像坐标系点->应用相机径向畸变后的图像坐标系点
相机畸变现象导致的点集偏移误差情况如下图所示。***OpenCV***为我们提供了多项式模型来进行矫正，在***HALCON***中同时提供了多项式模型和除法模型，这两个模型针对不同的情况，无论是线扫相机还是面阵相机，都可以使用。
对于我们的相机，一般是多项式模型的效果会比较好。其中特别的是，***OpenCV***和***HALCON***的多项式模型不太相同，前者使用的是**Distortion Coefficient**(畸变参数)，后者使用的是**Radial-Tangential**(径向切向)。两者的计算方法不同，***OpenCV***的计算方法为：  

$$
u' = u(1 + k_1r^2 + k_2r^4 + k_3r^6) + 2p_1uv + p_2(r^2 + 2x'^2)
$$

$$
v' = v(1 + k_1r^2 + k_2k^4 + k_3r^6) + p_1(r^2 + 2v'^2) + 2p_2uv  
$$

其中， $r^2 = u^2 + v^2$ 。
***HALCON***的计算方法为：  

$$
u' = u + (2p_1uv + p_2(r^2 + 2u^2)) + K_1r^2 + K_2r^4
$$

$$
v' = v + (2p_2uv + p_1(r^2 + 2v^2)) + K_3r^2 + K_4r^4
$$

其中， $r^2 = u^2 + v^2$ 。  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/distortion.png"/>  
</div>  

***HALCON***也在其技术文档中提供了畸变参数的计算方式，均可以根据项目的情况使用，我们在此仅考虑畸变参数这一方式。对于五个参数： $k_1,k_2,k_3,p_1,p_2$ ，***HALCON***文档中也列出了，当他们的符号改变时对应的大致的畸变情况，来供我们参考我们算出的参数是否正确，如下图所示：  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/effect_of_distortion.png"/>  
</div>  

### 2.4、应用相机径向畸变后的图像坐标系点->应用相机切向畸变后的图像坐标系点
相机在一些情况下，可能会出现本身的旋转偏移，这就是前面所提及的切向畸变。在极致平行的外界条件下，是可以不用考虑该步骤的，如果不考虑可以跳过本章节。相机切向畸变的情况如下图所示：  
<div align=center>
<img src="https://raw.githubusercontent.com/DaleBruise/DaleBruise.github.io/main/docs/images/2D_Calibration/tiltModel.png"/>   
</div>  
 
在不同相机模型的旋转情况下，会使用到不同的模型，***OpenCV***中使用到了**Louhichi_H**团队研究出来的模型，其计算方式如下所示：  

$$
\begin{bmatrix} u'' \\  
v'' \\
1
\end{bmatrix} 
= \begin{bmatrix} r_{33}(\tau_x,\tau_y) & 0 & -r_{13}(\tau_x,\tau_y) \\
0 & r_{33}(\tau_x,\tau_y) & -r_{23}(\tau_x,\tau_y) \\
0 & 0 & 1  
\end{bmatrix}
R(\tau_x,\tau_y)
\begin{bmatrix} u'\\ 
v' \\ 
1 
\end{bmatrix}
$$

其中， $R(\tau_x,\tau_y)$ 的定义如下：  

$$
R(\tau_x,\tau_y) 
= \begin{bmatrix} cos(\tau_y) & 0 & -sin(\tau_y) \\
0 & 1 & 0 \\
sin(\tau_y) & 0 & cos(\tau_y)  
\end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 \\
0 & cos(\tau_x) & sin(\tau_x) \\
0 & -sin(\tau_x) & cos(\tau_x)  
\end{bmatrix}
= \begin{bmatrix} cos(\tau_y) & sin(\tau_y)sin(\tau_x) & -sin(\tau_y)cos(\tau_x) \\
0 & cos(\tau_x) & sin(\tau_x) \\
sin(\tau_y) & -cos(\tau_y)sin(\tau_x) & cos(\tau_x)cos(\tau_y)
\end{bmatrix}
$$

至此旋转部分的转换就结束了，关于具体的公式和计算原理推导，请参考作者的文献，我们在此不过多讲述。  
### 2.5、应用相机畸变后的点->真实图像坐标系下的点
最终，当我们已经完成对畸变的所有处理后，就可以将点转换到图像坐标系下的点集。我们可以从先前建立的小孔成像的模型得到，在相机坐标系下中心点为坐标系的零点，但是在投影后的图像坐标系下，中心点的位置并不是零点而是图像中心点，我们最终需要将这个补偿加上。而且，画面还会有一定的缩放，加入缩放因子后，我们的表达式如下：  

$$
\binom{c}{r} = \binom{\frac{u''}{s_x} + c_x}{\frac{v''}{s_y} + c_y}
$$

其中， $s_x,s_y$ 即为缩放因子， $c_x,c_y$ 为图像中心坐标。我在前文提及到，该方法进行一次标定后，对于相同焦距的相机我们可以重复使用。因为由于画面被缩放了，因此我们需要最终改变点的大小，来应变画面的缩放。对于针孔相机，缩放因子就是CCD芯片上传感器原件的水平和垂直距离；对于远心相机，缩放因子则是世界坐标系中表示像素的实际大小比例。  
到目前为止，畸变模型计算的流程就结束了。但是我们可能会有疑问，首先是我们如何计算这五个参数（ $k_1,k_2,k_3,p_1,p_2$ ）；以及外部参数在我们仅仅知道相机的内部参数的情况下，如何计算……等等，这类问题我会留在后面的附录中，不定时更新，有任何问题或者错误都可以填补！🙇‍

## 附录1
### 径向畸变参数计算
我们需要进行计算的径向畸变参数有三个： $k_1,k_2,k_3$ 。我们设 $(u,v)$ 为没有畸变情况的点， $(\hat{u}, \hat{v})$ 为实际图像点， $(u_0,v_0)$ 为图像中心点（我们默认无误差，中心没有畸变现象），由上述内容，我们可以得到如下公式：  

$$
\hat{u} = u + (u-u_0)[k_1r^2 + k_2r^4 + k_3r^6]
$$  

$$
\hat{v} = v + (v-v_0)[k_1r^2 + k_2r^4 + k_3r^6]
$$  

其中， $r^2 = u^2 + v^2$ ，把上述公式转换成矩阵的形式，我们可以得到：  

$$
\begin{bmatrix} (u-u_0)r^2 & (u-u_0)r^4 & (u-u_0)r^6 \\
(v-v_0)r^2 & (v-v_0)r^4 & (v-v_0)r^6
\end{bmatrix}
\begin{bmatrix} k_1 \\
k_2 \\
k_3
\end{bmatrix} 
= \begin{bmatrix} \hat{u} - u \\
\hat{v} - v
\end{bmatrix}
$$

根据张正友老师的理论，我们可以使用线性最小二乘来解该线性方程组。首先我们有三个线性无关的向量 $(k_1, k_2, k_3)$ ，这三个线性无关的向量张成了 $\mathbb{R}^m$ 上的3维子空间 $V$ （或者更高维，根据我们需要计算的参数而定）。我们可以设估计向量 $\mathbf{p}$ 为：  

$$
\mathbf{p} = \mathbf{D}\begin{bmatrix} k_1 \\  
k_2 \\  
k_3 
\end{bmatrix} = \mathbf{D}\hat{\mathbf{k}}
$$  

其中，估计量向量 $\mathbf{p}$ 是实际量向量 $\mathbf{d}$ 在子空间上的投影。然后我们可以设误差向量 $\mathbf{e}$ 为实际量 $\mathbf{d}$ 向量和估计量向量之间的差：  

$$
\mathbf{e} = \mathbf{d} - \mathbf{p}  
$$

由于估计量向量 $\mathbf{p}$ 是子空间 $V$ 的基向量线性组合而成的，因此 $\mathbf{e} \bot \mathbf{v}$ 并且, $\forall\mathbf{v}\in V$ 。
也就是 $\mathbf{D^\top}\mathbf{e} = 0$ ，进一步，有：  

$$
\mathbf{D^\top}(\mathbf{d} - \mathbf{p}) = 0 
\implies \mathbf{D^\top}\mathbf{d} - \mathbf{D^\top}\mathbf{D}\mathbf{\hat{k}} = 0 \\  
\implies \mathbf{\hat{k}} = (\mathbf{D^\top}\mathbf{D})^{-1}\mathbf{D^\top}\mathbf{d}
$$  

也就是张正友老师在论文提出的最小二乘模型：  

$$
\mathbf{k} = (\mathbf{D^\top} \mathbf{D})^{-1}\mathbf{D^\top} \mathbf{d}
$$

我们在一张棋盘格图像中，可以得到 $m$ 个角点，因此我们也就会有 $m$ 个该方程式。我们求解如上式子，就可以得到 $k_1,k_2,k_3$ 的初值。供接下来的非线性最小二乘法来进行求解迭代后的精确参数结果，其模型为：  

$$
\sum_{i = 1}^{m} \begin{Vmatrix} \mathbf{m_i} - \mathbf{\hat{m}}(\mathbf{A}, k_1, k_2, k_3, \mathbf{R},\mathrm{t},\mathrm{M})\end{Vmatrix}^2
$$

其中， $\mathbf{\hat{m}}(\mathbf{A}, k_1, k_2, k_3, \mathbf{R},\mathrm{t},\mathrm{M})$ 是点 $\mathrm{M}$ 在当前图像下的投影。那么我们令 $\mathbf{f(k)} = \mathbf{m_i} - \mathbf{\hat{m}}(\mathbf{A}, k_1, k_2, k_3, \mathbf{R},\mathrm{t},\mathrm{M})$ ，其中 $\mathbf{f}$ 是一个含有三个未知量的标准差函数向量。我们想要找到最小的标准差对应的三个未知量的值，那么就是：  

$$
x^* = argmax_x{F(x)}，
$$

其中， $F(x)$ 的定义为：  

$$
F(x) = \frac{1}{2}\sum_{i = 1}^{m}(f_i(x))^2 = \frac{1}{2} {\begin{Vmatrix} \mathbf{f(x)} \end{Vmatrix}} ^ 2 = \frac{1}{2}\mathbf{f(x)}^{\top}\mathbf{f(x)}.
$$

默认 $\mathbf{f}$ 是连续二阶可导的函数的情况下，我们就可以对其进行泰勒一阶展开，并且利用雅可比矩阵表示，可以得到：  

$$
\mathbf{f(x + h)} = \mathbf{f(x)} + \mathbf{J(x)h} + o({\begin{Vmatrix} \mathbf{h} \end{Vmatrix}} ^ 2)
$$

然后我们假设有一个模型 $L$ 来表示 $F$ 在当前迭代 $x$ 的领域中的行为，那么：  
  
$$
\mathbf{f(x+h)} \simeq l\mathrm{h} \equiv \mathbf{f(x)} + \mathbf{J(x)h}
$$
  
由此，我们可以得到：  

$$
\begin{aligned} F(\mathbf{x+h}) \simeq L(\mathbf{h}) & \equiv \frac{1}{2}l(h)^{\top}l(\mathbf{h}) \\  
& = \frac{1}{2}\mathbf{f^{\top}f} + \mathbf{h^{\top}J^{\top}f} + \frac{1}{2}\mathbf{h^{\top}J^{\top}Jh} \\  
& = F(x) + \mathbf{h^{\top}J^{\top}f}+ \frac{1}{2}\mathbf{h^{\top}J^{\top}Jh} \end{aligned}  
$$

通过高斯牛顿优化方法原理，我们需要得到 $L(h)$ 的*jacobin*矩阵和*Hessian*矩阵：  

$$
\mathbf{L'(h)} = \mathbf{J^{\top}f} + \mathbf{J^{\top}Jh},\  \mathbf{L''(h)} = \mathbf{J^{\top}J}.
$$

我们通过输入高斯扭断的上一次迭代参数 $\mathbf{}h_{gn}$ 来进行下一次迭代，因此我们需要求出下面这个式子：  

$$
\mathbf{(J^{\top}J)h_{gn}} = -\mathbf{J^{\top}f}
$$

然而，我们使用的是经过 **LM** 算法优化的最小二乘迭代，所以上述式子会稍许不太一样，如下所示：  

$$
(\mathbf{J^{\top}J} + \mu\mathbf{I})h_{lm} = -\mathbf{J^{\top}f} \, and \, \mu \geq 0
$$

其中 $\mu$ 是一个限制因子，从而根据得分来控制我们迭代的速度。这个得分 $\rho$ 的计算方式如下：  

$$
\rho = \frac{F(\mathbf{x}) - F(\mathbf{x + h_lm})}{L(0) - L(\mathbf{h_lm})}
$$

根据上式中 $L$ 模型的特性，我们可以知道这个得分是一个大概率大于0的值，算法会根据这个得分来更改 $\mu$ 的值来控制迭代的速度。从而能够更加准确地估计出我们想要的参数。具体的推导流程和算法流程，可以参考 *K.Madsen* 的文献，名称会贴到附录2中。

## 附录2
文章中的参考链接或文献如下：  

1. 畸变 https://en.wikipedia.org/wiki/Distortion_(optics)
2.  ***HALCON*** 《Solution Guide Ⅲ-C 3D Vision》
3.  ***OpenCV*** https://docs.opencv.org/3.4.1/d9/d0c/group__calib3d.html#ga3207604e4b1a1758aa66acb6ed5aa65d
4. ***OpenCv***源码解析 https://blog.csdn.net/weixin_43956164/article/details/126771627?spm=1001.2014.3001.5501
5. Zhengyou Z. Flexible Camera Calibration By Viewing a Plane From Unknown Orientations [J] Seventh IEEE International Conference on Computer Vision.IEEE, 1999.DOI:10.1109/ICCV.1999.791289
6. 矩阵求导 https://zhuanlan.zhihu.com/p/273729929
7. LM最小二乘优化 Note L. Methods for Non-Linear Least Squares Problems[J] Lecture Note [2023-11-21]

