<!-- TOC -->

- [14.1 Integration](#141-integration)
  - [14.1.1 Measures and Averages](#1411-measures-and-averages)

<!-- /TOC -->

**Sampling采样**

<a id="markdown-141-integration" name="141-integration"></a>
### _14.1 Integration

没太看懂, 抄两个公式:
$$A(S) \equiv \int_{s\in S}dA(x) $$
means take all points x in the region Sampling, and sum their associated differential areas  
$$A([a, a+1]\times[b, b+1]) = 1$$
(a, b)代表的是左下角的点, 所以上面的等式代表一个单位square

<a id="markdown-1411-measures-and-averages" name="1411-measures-and-averages"></a>
#### _14.1.1 Measures and Averages


### _14.2 Continuous Probability

#### _14.2.1 One_Dimensional Continuous Probability Density Functions

一个continuous random variable连续随机变量x在$\mathbb{R}=[-\infty, +\infty]$范围内随机取值, 我们要计算其在a-b范围内取值的概率, 定义一个probability density function(pdf):
$$Probability(x\in[a, b]) = \int_a^bp(x)dx$$
相当于$x$在$[a,b]$范围内所有采样点的概率相加, 采样间隔是$dx$, 这个pdf满足这样的特性:
$$
\begin{aligned}
p(x) \ge 0 \\
\int_{-\infty}^{+\infty}p(x)dx = 1
\end{aligned}
$$
举一个例子, 某一个变量$\xi$只在$[0, 1]$范围内取值, 在范围内的取值概率是一样的, 那么它的pdf就是:
$$
q(\xi) = \left\{
  \begin{matrix}
  1\quad 0 \le \xi \lt 1 \\
  0\quad otherwise
  \end{matrix}
\right.
$$
对于$0\le a \lt b \lt 1$, $a-b$范围内的概率:
$$Probability(a\le\xi\le b) = \int_a^b1d_x = b - a$$