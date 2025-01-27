## Stable Diffusion

#### DDPM原理

从$X_0$（输入的自然图片）到$X_T$（加了很多噪声的纯噪音）的加噪过程我们成为前向过程（forward process），

从$X_T$到$X_0$的去噪过程我们成为反向过程（reverse process/inversion）

通过前向过程我们想要得到一个$X_T$所有可能的**分布**概率—$X_T \sim N(0,1)$，而不是一个确切的$X_T$，然后从中取出一个$X_{T}^{'}\rightarrow X_0^{'}$，$X_0{'}$就是我们的目标—生成出来的与原图不同的具有创新性的新图片（generated by DDPM and different from the original picture ,a new image）。

对于反向过程中中的某一step$X_t$，我们的目标是得到$X_{t-1}$的分布$P(X_{t-1}|X_t)$而不是一个确定的$X_{t-1}$​，因为这样我们可以从得到的分布中随机抽取一个样本，从而具有随机性。随机性对于生成模型是非常重要的，因为我们希望生成的图片与原图是不一样的。

>  所谓分布，可以把它理解成对于each pixel，其取值的可能性集合。如14x14的图片我们就可以得到一个196维的分布，那我们sample的时候就会根据每个pixel落在不同区间的概率，当做权重去给每个pixel赋值。

所以我们反向过程一步步denoise的目标就是求$P(X_{t-1}|X_t),t∈[0,steps]$，需要求1000个分布。
$$
P(X_{t-1}|X_t)=\frac{P(X_t,X_{t-1})}{P(X_t)}=\frac{P(X_t|X_{t-1})P(X_{t-1})}{P(X_t)}
$$
那么这个$P(X_t|X_{t-1})$实际上就是forward process，添加噪声的过程，那我们具体怎么添加噪声呢？

利用如下公式添加：
$$
X_t=\sqrt{a_t}X_{t-1}+\sqrt{\beta_t} \epsilon,\epsilon\sim N(0,1)
$$
$\beta_t=1-a_t$，那么$\sqrt{\beta_t}\epsilon\sim N(0,\beta_t)$，我们规定$\beta_t$一定非常小，趋于0
$$
\therefore X_t\sim N(\sqrt{a_t}X_{t-1},\beta_t)
$$
所以我们就拿到了$X_t$的概率分布$P(X_t|X_{t-1})$————$P(X_t|X_{t-1})\sim N(\sqrt{a_t}X_{t-1},\beta_t)$

那**（1）**中的$P(x-1)$怎么办？我们可以根据👆的递推式一步一步推出来，相当于$P(X_{x-1}|X_0)$

即：
$$
P(X_{t-1})\rightarrow P(X_{t-1}|X_{0})\\
P(X_t)\rightarrow P(X_{t}|X_0)
$$
那让我们重写一下**（1）**式吧！！！！
$$
P(X_{t-1}|X_t,X_0)=\frac{P(X_t,X_{t-1},X_0)}{P(X_t)}=\frac{P(X_t|X_{t-1},X_0)P(X_{t-1}|X_0)}{P(X_t|X_0)}
$$
但是DDPM有一个大前提！或者说假设：整个过程符合**马尔科夫过程**！就是当前状态只跟上一个状态有关，跟再之前的状态没有任何关系！！！

所以**（5）**中的$P(X_{t}|X_{t-1}X_0)$就可以去掉$X_0$！------>$P(X_t|X_{t-1})$

那$P(X_{t-1}|X_0)$呢？因为这里没有$X_{t-2}$，所以这个$X_0$不能去。但是我们思考一下从$X_0$到$X_{t-1}$是怎么来的？
$$
X_{t-1}=\sqrt{a_{t-1}}X_{t-2}+\sqrt{\beta_{t-1}}\epsilon_{t-1}\\
\because X_{t-2}=\sqrt{a_{t-2}}X_{t-3}+\sqrt{\beta_{t-2}}\epsilon_{t-2}\\
我们带进去：\\
X_{t-1}=\sqrt{a_{t-1}}(\sqrt{a_{t-2}}X_{t-3}+\sqrt{\beta_{t-2}}\epsilon_{t-2})+\sqrt{\beta_{t-1}}\epsilon_{t-1}\\
X_{t-1}=\sqrt{a_{t-1}a_{t-2}}X_{t-3}+(\sqrt{a_{t-1}\beta_{t-2}}\epsilon_{t-2}+\sqrt{\beta_{t-1}}\epsilon_{t-1})\\
n\ steps\ later\ ......\\
X_{t-1}=\sqrt{a_{t-1}a_{t-2}...a_{1}}X_0+C\epsilon^{'},\epsilon'是各个step的\epsilon的线性组合，\epsilon^{'}\sim N(0,1)\\
\therefore X_{t-1}\sim N(\sqrt{a_{t-1}a_{t-2}...a_1}X_0,1-\sqrt{a_{t-1}a_{t-2}...a_1})\\
令\bar{a}_{t-1}=a_{t-1}a_{t-2}...a_1\\
\therefore P(X_{t-1}|X_0)\sim N(\sqrt{\bar{a}_{t-1}}X_0,1-\sqrt{\bar{a}_{t-1}})\\同理，P(X_{t}|X_0)\sim N(\sqrt{\bar{a}_{t}}X_0,1-\sqrt{\bar{a}_{t}})
$$
那么**（5）**中的这三项我们就都求出来啦！！一定可以经过计算得到**（5）**这个式子的分布：

这里省略计算过程：
$$
P(X_{t-1}|X_t,X_0)\sim N(F(X_0,X_t),G(a_t,\beta_t))
$$

1. 因为F、G函数分别表示N的均值和方差，那我们给他俩换个符号吧！
   $$
   F\rightarrow\mu,G\rightarrow\widehat{\beta}
   $$

2. 因为$\widehat\beta(a_t,\beta_t)$中的$a_t、\beta_t$是定值，所以$\widehat{\beta}$实际上也是个常数

##### 最终公式

$$
P(X_{t-1}|X_t,X_0)\sim N(\mu(X_0,X_t),\widehat{\beta})
$$

未完待续......