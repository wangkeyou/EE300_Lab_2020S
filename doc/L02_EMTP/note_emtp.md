## 电磁暂态仿真算法介绍

### 电磁暂态仿真算法的基本类型

电力系统的电磁暂态仿真算法主要可以分为两个大类：状态变量法和节点分析法。

状态变量法，其数学本质是建立描述电力系统电磁暂态特性的代数-微分方程组，利用数值积分方法直接对其进行求解。该方法具有普适性，理论上可以用于一切动态系统的求解。该方法的优点主要包括：1、容易处理非线性模型；2、没有数值振荡问题； 3、容易调整仿真步长。同时，该方法的缺点主要包括：1、单个仿真步内有迭代过程， 求解时间较长；2、形成独立微分方程模型时增加了编程的复杂度；3、不易于求解含分布参数的输电线模型。基于Simulink 环境的 SimPowerSystems 电力系统仿真工具箱[11]和瑞士 Plexim 公司开发的 PLECS 电力电子仿真软件都采用了这种算法。

节点分析法，于 20 世纪 60 年代，由邦尼韦尔电管局（BPA）的 H. W. Dommel 博士开发，最早用于输电线的电磁暂态仿真。最早采用这一方法的电磁暂态仿真程序以EMTP（Electro-Magnetic Transient Programs）的名字著称。不同于基于微分方程模型的状态变量法，节点分析法是基于差分电路模型。每个元件先经过数值积分代换后形成差分电路模型，然后建立网络的节点电压方程进行求解，故而称为节点分析法。此外，该方法还被称为 Dommel 方法、EMTP 法、数值积分代换法等等。其优缺点正好和状态变量法相反。
EMTP优点包括：
1、单个仿真步内无迭代过程，求解时间较短；
2、形成网络方程简单，易于编程实现；
3、易于求解分布参数的输电线模型。

EMTP缺点包括：
1、不易于处理非线性模型；
2、可能出现数值振荡问题；
3、不易于调整仿真步长。

尽管如此，节点分析法可能仍是目前使用最广的电磁暂态仿真算法。从 Dommel 博士的 EMTP 程序又衍生出了众多版本，包括美国电科院和 EMTP 发展协调组织开发推广的 EPRI/DCG 版本（即EMTP-RV）、由 BPA 牵头开发的 ATP 版本以及不列颠哥伦比亚大学的UBC 版本（即 Microtran）等。受 Dommel 博士工作的启发，曼尼托巴水电局（现在更名为曼尼托巴水电国际）开发了 PSCAD/EMTDC[14]，用于交直流输电系统的相关研究，是目前使用范围最广的电磁暂态仿真软件之一。在电力系统的实时仿真方面，主流的仿真平台有 RTDS 公司开发的 RTDS、魁北克水电局开发的 HYPERSIM（现由 OPAL-RT 公司维护）、法国电力公司（EDF）开发的 ANENE、中国电科院开发的 ADPSS 等，也都采用了这种算法。

电磁暂态仿真算法在本质上没有明显的优劣之分，通过采用高效的建模方法、合理的算法优化技术以及高性能的计算平台，都有实现实时仿真的潜力。但相较状态变量法而言，节点分析法在形成网络方程时，无需考虑相连支路间的独立性，编程实现更为简单有效，为实时仿真平台广泛采用。

### 节点分析法的算法流程

### 元件的差分化
节点分析法中，对电路网络进行求解之前，需要对动态电路元件进行差分化处理。
1）电阻元件
如图所示，节点i 和节点j间的电阻，其特性方程可以表示为：

$$i_{ij}(t)=\frac{1}{R}\left(v_i(t)-v_j(t) \right)$$
<center><img src="img/note_emtp-2020-01-30-22-10-19.png" alt="" width="30%"></center>


电阻的特性方程中不含微分项，因而不需要进行差分化处理。
2）电感元件
如图所示，节点i 和节点 j 之间的电感，其特性方程可以表示为：

$$u_L(t)=v_i(t)-v_j(t)=L\frac{di_L(t)}{dt}$$
<center><img src="img/note_emtp-2020-01-30-22-15-00.png" alt="" width="30%"></center>

以梯形法为例，用进行数值积分求解，经整理后可以将电感支路表示为等效导纳与历史电流源的并联的形式：
$$i_L(t)=i_L(t-\Delta t)+\frac{\Delta t}{2 L} \left[u_L(t)+u_L(t-\Delta t)\right] $$
整理
$$
  \begin{matrix}
   i_L(t) & = & \frac{\Delta t}{2 L} u_L(t)+i_L(t-\Delta t)+\frac{\Delta t}{2L}u_L(t-\Delta t) \\
    & = & Y_L u_L(t)-i_{history}(t-\Delta t)
  \end{matrix} 
$$
其中， $Y_L$为电感的等效导纳， $i_{history}(t-\Delta t)$为历史电流源：
$$
\begin{cases}
Y_L= \frac{\Delta t}{2L} \\
i_{history}(t-\Delta t)= -i_L(t-\Delta t)- \frac{\Delta t}{2L} u_L(t-\Delta t)
\end{cases} 
$$

利用上述模型的整理形式，可以得到如图的 Norton 等效电路，即电感的差分电路模型。
<center><img src="img/note_emtp-2020-01-30-23-07-16.png" alt="" width="30%"></center>


3）电容元件
如图所示，电容元件的特性方程可以表示为：
$$i_C(t)=i_{ij}(t)=C\frac{d(v_i-v_j)}{dt}=\frac{du_C}{dt}$$
<center><img src="img/note_emtp-2020-01-30-23-11-57.png" alt="" width="30%"></center>

与电感类似，可以用梯形法积对电容的特性方程进行数值积分求解得：
$$ u_C(t)= u_C(t-\Delta t)+\frac{\Delta t}{2C} \left[i_C(t) + i_C(t-\Delta t) \right]$$

为了将所有元件整理成统一的 Norton 形式，该方程经过整理得：
$$
  \begin{matrix}
   i_C(t) & = & \frac{2C}{\Delta t}u_C(t)-\frac{2C}{\Delta t} u_C(t-\Delta t)- i_C(t-\Delta t)\\
    & = & Y_C u_C(t)-i_{history}(t-\Delta t)
  \end{matrix} 
$$
其中， $Y_C$ 为电容等效导纳， $i_{history}(t-\Delta t)$为历史电流源：
$$
\begin{cases}
Y_C=\frac{2C}{\Delta t} \\
i_{history}(t-\Delta t)=\frac{2C}{\Delta t} u_C(t-\Delta t)+ i_C(t-\Delta t)
\end{cases}
$$
由此，可以得到形如图的 Norton 等效电路。

<center><img src="img/note_emtp-2020-01-30-23-07-16.png" alt="" width="30%"></center>


4）交流电压源
交流电压源为电力系统中常见的电源元件。交流电压源的建模方式为一个理想电压源与一个小电阻串联，如图所示。交流电压源同样需要转化成统一的 Norton 等效电路形式。由于电压源特性方程不含微分项，因此也不需要进行差分化。

<center><img src="img/note_emtp-2020-01-30-23-20-07.png" alt="" width="30%"></center>
设交流电压源的瞬时值为 

$$u_s(t)=A sin(\omega t + \phi)$$

其中 $A$ 为电压源幅值，$\omega$ 、$\phi$ 为别为角速度与初始相位，转化成 Norton 等效电路可以表示为：
$$
\begin{cases}
Y_s=\frac{1}{R_s} \\
i_{s}(t)=\frac{A}{R_s} sin(\omega t + \phi)
\end{cases}
$$
值得说明的是，节点分析法中，交流电压源的内阻不可忽略，必须设置，且理想电压源与内阻作为一条支路进行计算。
交流电流源由于本身就是 Norton 等效电路形式，因而不需要进行等效电路转换。另外，直流电源可以利用零频率的交流电源进行表示。


### 网络方程的建立与求解
利用数值积分法将所有动态元件进行差分化，并将所有元件都用统一的 Norton 等效电路形式表示。下一步就是将电路网络进行编号，由此即可列写网络的节点电压方程， 并进行求解。
为了更直观地说明网络方程的建立和求解过程，下面以如图所示的简单 LCL 滤波电路为例，进行说明。           
<center><img src="img/note_emtp-2020-01-30-23-28-31.png" alt="" width="40%"></center>

根据元件差分化方法，将上图中的动态网络，表示成如图所示的 Norton 等效电路组成的网络。      
<center><img src="img/note_emtp-2020-01-30-23-30-13.png" alt="" width="40%"></center>

然后，以节点 2 为例，说明节点电压方程的形成。与节点 2 相连的有两个电感支路和一个电容支路，根据元件差分电路模型，可以列写各个支路的电流表达式如下：

$$
\begin{cases}
i_{21}(t)=\frac{\Delta t}{2 L_1}\left( v_2(t)-v_1(t)\right)+i_{hL1} \\
i_{20}(t)=\frac{2C}{\Delta t}\left( v_2(t)-0\right)-i_{hC} \\
i_{23}(t)=\frac{\Delta t}{2 L_2}\left( v_2(t)-v_3(t)\right)-i_{hL2} \\
\end{cases}
$$
根据基尔霍夫电流定律，有节点 2 处注入电流之和为 0，据此就可以将上式整理成节点电压方程的形式，
$$ -\frac{\Delta t}{2 L_1} v_1(t）+\left[ \frac{\Delta t}{2L_1}+\frac{2C}{\Delta t}+\frac{\Delta t}{2 L_2}\right]v_2(t)-\frac{\Delta t}{2 L_2}v_3(t)=-i_{hL1}+i_{hC}+i_{hL2}$$

利用相同的方法，将节点 1 与节点 3 进行节点电压方程列写，而后将方程组联立，整理成如下矩阵形式的节点电压方程：
$$ \bm{Yv} (t)=\bm{i}(t)$$
具体展开后有网络方程如下：
$$
\left[ \begin{matrix}
\frac{1}{R_s}+\frac{\Delta t}{2L_1}& -\frac{\Delta t}{2L_1} & 0\\
-\frac{\Delta t}{2L_1} & \frac{\Delta t}{2L_1}+\frac{2C}{\Delta t}+\frac{\Delta t}{2L_2} & -\frac{\Delta t}{2L_2}\\
0 & -\frac{\Delta t}{2L_2} & \frac{\Delta t}{2L_2}+\frac{1}{R}
\end{matrix} \right]
\left[ \begin{matrix}
v_1 \\ v_2\\ v_3
\end{matrix} \right]=
\left[ \begin{matrix}
\frac{u_s}{R_s}+i_{hL1}\\
-i_{hL1}+i_{hC}+i_{hL2}\\
-i_{hL2}
\end{matrix} \right]
$$
对于被仿真网络，无论规模大小，都可按照上述相同方法进行建模，以列写得到节点电压方程。
对该网络方程进行求解，是节点分析法的关键步骤。方程右侧的$\bm{i}(t)$ 为节点注入电流向量，由各动态元件支路的历史电流以及独立电源支路的等效电流计算得到。在 EMTP 程序计算开始时，根据不同的初始化方法设置历史电流源，通常采用历史电流源初始值为$0$ 的方式。

在每一次仿真循环的计算过程中，先由上一仿真步的支路电压和支路电流计算得到当前仿真步的历史电流，与当前仿真步的独立电压源的等效电流一起，计算得出节点注入电流向量$\bm{i}(t)$。然后根据网络方程求解节点电压向量 $\bm{v}(t)$，并根据节点电压计算得到支路电压向量、支路电流向量。如此循环往复并记录每一仿真步的仿真结果，直到仿真过程终止。在仿真过程终止后，即可打印仿真结果，包括支路电流、支路电压、节点电压等。

基于上述过程，节点分析法的算法流程如图所示。
<center><img src="img/note_emtp-2020-01-30-23-53-36.png" alt="" width="35%"></center>