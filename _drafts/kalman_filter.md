---
title: 칼만 필터 Kalman Filter 
categories:
  - Develop
  - Devnote
tags:
  - SystemControl
  - Writing
mathjax: true
---

# 개요
칼만 필터는 시스템의 다음 상태를 예측하고, 예측한 수치에 측정치를 잘 보정하여 추정치를 계산하는 필터입니다.

이 때 잘 보정하기 위해 얼마나 이 예측치가 믿을만한지와 관측치가 추정치에 비해 얼마나 믿을만한지 매 단계마다 평가하는 과정이 들어갑니다.

# 구조 Structure

## 알고리즘 개요
0. 초기값 선정 Initialize 
  * 초기 추정값 $$\hat{x}_0$$
  * 초기 공분산 $$P_0$$
1. 예측 Predict (Extrapolate) (1) (2)
  * 예측값 $$ \hat{x}_{\bar{k}} $$
  * 공분산 예측값 $$ P_{\bar{k}} $$
2. 칼만 게인 Kalman gain (3)
  * 칼만 게인 $$K_k$$
3. 추정값 갱신 Update (4) (5)
  * 추정값 $$\hat{x}_{k}$$
  * 공분산 $$P_{k}$$


## 칼만 필터를 구성하는 주요 5개 식

$$
\begin{align}
\hat{x}_{\bar{k}} &= A\hat{x}_{k-1} \tag{1} \label{predictor}\\
P_{\bar{k}} &= AP_{k-1}A^T+Q \tag{2} \label{covariance predictor} \\
K_k &= P_{\bar{k}}H^T(HP_\bar{k}H^T+R)^{-1} \tag{3} \label{kalman gain}\\
\hat{x}_{k} &= \hat{x}_{\bar{k}} + K_k(z_{k}-H\hat{x}_{\bar{k}}) \tag{4} \label{updator}\\
P_{k} &= (I-K_kH)P_{\bar{k}} \tag{5} \label{covariance updator}
\end{align}
$$

* $$\hat{x}$$: 추정치
* $$z$$: 측정치
* $$P$$: 오차 공분산
* $$K$$: 칼만 게인

시스템 모델과 관련된 상, 변수: $$A, H, Q, R$$
* $$A$$: 시스템 행렬
* $$H$$: 출력 행렬
* $$Q$$: 노이즈 오차
* $$R$$: 측정치 오차

# 시스템 모델
## 선형 시스템 모델
칼만 필터는 다음과 같은 선형 상태 모델을 대상으로 합니다.

$$
\begin{align}
x_{k+1} &= Ax_k + w_k \tag{sys.1} \label{sys.1}\\
z_k &= Hx_k + v_k \tag{sys.2} \label{sys.2}
\end{align}
$$

* $${x_k}$$: 상태 변수, ($$n\times 1$$ 벡터) 
  * state space variables
* $${z_k}$$: 측정값, ($$m\times 1$$ 벡터)
* $${A}$$: 시스템 행렬, ($$n\times n$$ 행렬)
  * 현재 시스템과 다음 시스템간의 관계를 규정, 상수
* $${H}$$: 관측 행렬, ($$n\times 1$$ 벡터)
  * 측정값과 상태 변수의 관계를 규정, 상수
* $${w_k}$$: 시스템 잡음, ($$n\times 1$$ 벡터)
  * 시스템에 유입되어 상태 변수에 영향을 주는 잡음
* $${v_k}$$: 측정 잡읍, ($$m \times 1$$ 벡터)
  * 센서에서 유입되는 측정 잡음

$$\ref{sys.1}$$ 은 $$\ref{predictor}$$번식에, $$\ref{sys.2}$$ 은 $$\ref{updator}$$번식에 사용된 것을 확인할 수 있습니다.

여기서 시스템 잡음 $$w$$는 공분산 행렬 $$Q$$로 나타내고, 마찬가지로 측정 잡음 $$v$$는 공분산 행렬 $$R$$로 나타냅니다.

## 잡음의 공분산

만약 각 시스템 변수의 잡음이 서로 독립적이라면, $$Q$$는 대각 행렬을 이루며, 다음과 같은 형태를 띕니다.

$$
\begin{bmatrix}
\sigma_1^2 & 0 & \cdots & 0 \\
0 & \sigma_2^2 & 0 & \cdots \\
\vdots & \vdots & \vdots & \vdots \\
0 & 0 & \cdots & \sigma_n^2
\end{bmatrix}
$$

하지만 만약 각 시스템 변수의 잡음이 서로 결합되어 있다면 $$Q$$는 대각 행렬이 아닌 행렬을 이루며,\
몇가지 해석적 방법을 사용해볼 수 있습니다.

[참고: Q 행렬 해석적으로 구하기](https://www.kalmanfilter.net/covextrap.html)

위 과정은 측정치 오차 $$R$$에 대해서도 동일하게 적용됩니다.

$$Q$$와 $$R$$에 대해서 대략적인 감을 잡아봅시다.

만약 $$Q$$가 증가한다면, $$\ref{covariance predictor}$$번 식에서 공분산 예측값이 증가합니다.\
그리고 증가한 공분산은 칼만 게인을 더 크게 만들고, 이는 측정값의 영향을 더 받도록 유도합니다.

$$
\begin{aligned}
P_{\bar{k}} &= AP_{k-1}A^T+Q \\
K_k &= P_{\bar{k}}H^T(HP_{\bar{k}}H^T+R)^{-1} 
\end{aligned}
$$

반대로 $$R$$이 증가하면 칼만 게인을 작게 만들어 측정값의 영향을 덜 받도록 유도합니다.


# 참고문서
* [칼만 필터는 어렵지 않아](http://www.yes24.com/Product/Goods/73621194)
* [kalmanfilter.net](https://www.kalmanfilter.net/default.aspx)