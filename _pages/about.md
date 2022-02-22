---
permalink: /about/
title: "About"
author_profile: true
layout: single
mathjax: true
---
Hi! I'm heheHwang


```python
from math import gcd

```

$$
\begin{align}
\mathcal{L}_r &= 
\frac{1}{N_r}\sum^{N_r}_{i=1} 
\left| 
\alpha \frac{\partial^2 f(x, t, h_1, h_2)}{\partial x^2} - \frac{\partial f(x, t, h_1, h_2)}{\partial t} \; \right|^2
\\[2ex]
\mathcal{L}_b &= 
\frac{1}{N_b}\sum^{N_b}_{i=1}
\left| 
-(T_{\infty}(t) - f(x_{b}, t, h_1, h_2)) + \left. \frac{k}{h_1} \frac{\partial f(x, t, h_1, h_2)}{\partial x} \; \right|_{x = b}
\right|^2
\\[2ex]
\mathcal{L}_0 &= 
\frac{1}{N_0}\sum^{N_0}_{i=1}
\left| 
T_{\infty}(0) - f(x, 0, h_1, h_2)
\right|^2
\end{align} 
$$

$$
\begin{cases}
\frac{\partial T}{\partial t} - \alpha \frac{\partial^2 T}{\partial x^2} = 0, \;\; \alpha = \frac{k}{\rho C_p} \\[2ex]
h(T_{\infty}(0)-T_{initial}) = 0 \\[2ex]
h(T_{\infty}(t)-T_{boundary}) = k \left. \frac{\partial T}{\partial x} \right|_{boundary}
\end{cases}
$$