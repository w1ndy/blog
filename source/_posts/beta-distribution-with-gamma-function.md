---
title: Beta Distribution with Gamma Function
tags:
  - mathematics
  - probability
id: 19
categories:
  - Mathematics
date: 2015-07-23 14:40:01
---

[Beta distribution](https://en.wikipedia.org/wiki/Beta_distribution): {% math %}f(x;\alpha,\beta)=\frac{x^{\alpha -1}(1-x)^{\beta -1}}{\int_0^1u^{\alpha -1}(1-u)^{\beta -1}}{% endmath %}

Prove:
{% math %}
\begin{aligned}\frac{x^{\alpha -1}(1-x)^{\beta -1}}{\int_0^1u^{\alpha -1}(1-u)^{\beta -1}}=\frac{\Gamma(\alpha +\beta)}{\Gamma(\alpha)\Gamma(\beta)}x^{\alpha -1}(1-x)^{\beta -1}
\end{aligned}{% endmath %}

, where [gamma function](https://en.wikipedia.org/wiki/Gamma_function) {% math %}\Gamma(\alpha)=\int_0^\infty t^{\alpha -1}e^{-t}dt{% endmath %}.
<!-- more -->

{% math %}
\begin{aligned}
\Gamma(\alpha)\Gamma(\beta) = \int_0^\infty x^{\alpha -1}e^{-x}dx\int_0^\infty y^{\beta -1}e^{-y}dy
\end{aligned}
{% endmath %}
Let {% math %}t=x+y{% endmath %}, so we have {% math %}y=t−x{% endmath %} and {% math %}dy=dt{% endmath %}:
{% math %}
\begin{aligned}
\Gamma(\alpha)\Gamma(\beta) &= \int_0^\infty x^{\alpha -1}e^{-x}\int_x^\infty (t-x)^{\beta -1}e^{-(t-x)}dtdx \\
&= \int_0^\infty x^{\alpha -1}\int_x^\infty (t-x)^{\beta -1}e^{-t}dtdx \\
&= \int_0^\infty e^{-t}\int_0^t (t-x)^{\beta -1}x^{\alpha -1}dxdt
\end{aligned}
{% endmath %}
Let {% math %}x=\mu t{% endmath %}, thus {% math %}dx=td\mu{% endmath %} and {% math %}\mu \in(0,1){% endmath %}:
{% math %}
\begin{aligned}
\Gamma(\alpha)\Gamma(\beta) &= \int_0^\infty e^{-t}\int_0^1 (t-\mu t)^{\beta -1}(\mu t)^{\alpha -1}td\mu dt \\
&= \int_0^\infty e^{-t}t^{\alpha +\beta -1}dt\int_0^1 (1-\mu)^{\beta -1}\mu^{\alpha -1}d\mu \\
&= \Gamma(\alpha +\beta)\int_0^1 \mu^{\alpha -1}(1-\mu)^{\beta -1}d\mu \\
\frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha +\beta)} &= \int_0^1 \mu^{\alpha -1}(1-\mu)^{\beta -1}d\mu \\
\frac{x^{\alpha -1}(1-x)^{\beta -1}}{\int_0^1u^{\alpha -1}(1-u)^{\beta -1}} &= \frac{\Gamma(\alpha +\beta)}{\Gamma(\alpha)\Gamma(\beta)}x^{\alpha -1}(1-x)^{\beta -1}
\end{aligned}
{% endmath %}