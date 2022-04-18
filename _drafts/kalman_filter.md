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

# 소개 Introduction
칼만 필터는 시스템의 다음 상태를 예측하고, 예측한 수치에 측정치를 잘 보정하여 추정치를 계산하는 필터입니다.

예를 들어 $$v$$의 속도로 움직이는 어떤 물체의 움직임을 $$\Delta t$$마다 추적하는 시스템을 생각해봅시다.

우리가 흔히 배운 물리법칙으로 생각해 볼 것 같으면, $$x_k$$의 다음 상태인 $$x_{k+1}$$는 $$x_k + \Delta t v_k$$가 되어야 할 것으로 추측할 수 있습니다.\
그러나 실제로 현실 물체에서 위치를 추적해보면, 실험실 환경에서 모든 외부 변수를 차단하고 정밀도가 높은 측정장비를 사용해야 겨우 예측과 비슷한 값을 얻을 수 있습니다.

이는 달리 말하면 다양한 외부 변인에 노출된 시스템에서, 값싼 센서를 이용해 측정하는 현실에서는\
물리법칙으로 계산한 결과와 측정값 모두 실제값과 차이가 발생하게 됩니다.

그렇다면 여기서 우리는 두 결과를 조합하여 각 결과에 포함된 노이즈를 줄이는 방법을 생각해볼 수 있는데, 이 방법 중 가장 유명하고 기본적인 방법이 칼만 필터입니다.\
즉, 여기서 말하는 필터는 노이즈를 줄인다는 의미입니다.


가장 처음 개발된 칼만 필터는 선형-시간불변 시스템(Linear, Time-Invariant System)을 대상으로 기술되었으며,\
이후 비선형 시스템에 대응하기 위해 확장 칼만 필터(Extended Kalman Filter; EKF), 무향 칼만 필터(Unscented Kalman Filter; UKF)가 개발되었습니다.

# 배경지식 Background
* 기초적인 통계 및 선형대수학 
* 베이즈 필터 Bayesian Filter
* 상태 공간 State Space
* 제어 공학 Control Engineering

# 칼만 필터의 구조 Structure

## 구조 개요 Outline
(구조도)
칼만 필터는 기본적으로 측정치(Measurement)을 입력으로 받고 상태 변수들의 추정치(Estimation)을 출력으로 내보냅니다.

## 주요 변수 소개 Variables

| 변수 Variable | 설명 Description | 차원 Dimension |
| :---: | :---: | :---: |
| $$z_k$$ | 측정치 Measurement | $$n_z, 1$$ |
| $$x_k$$ | 실제 상태 (숨겨짐) State vector | $$n_x, 1$$ |
| $$u_k$$ | 입력값 Input vector | $$n_u, 1$$ |
| $$\hat{x}_k$$ | 상태 추정치 Estimated state | $$n_x, 1$$ |
| $$\hat{x}_{\bar{k}}$$ | 상태 예측치 Predicted state | $$n_x, 1$$ |
| $$w_k$$ | 상태 전이 노이즈 Process noise | $$n_x, 1$$ |
| $$v_k$$ | 측정 노이즈 Measurement noise | $$n_z, 1$$ |
| $$F$$ | 상태 전이 행렬 State transition matrix | $$n_x, n_x$$ |
| $$G$$ | 제어 행렬 Control matrix | $$n_x, n_u$$ |
| $$H$$ | 측정 행렬 Opervation matrix | $$n_z, n_x$$ |
| $$Q$$ | 상태 전이 노이즈 공분산 Process noise uncertainty matrix | $$n_x, n_x$$ |
| $$R$$ | 측정 노이즈 공분산 Measurement uncertainty matrix | $$n_z, n_z$$ |
| $$P_k$$ | 상태 추정치 공분산 Covariance matrix | $$n_x, n_x$$ |
| $$P_{\bar{k}}$$ | 공분산 예측치 Predicted covariance matrix | $$n_x, n_x$$ |
| $$K_k$$ | 칼만 게인 Kalman gain | $$n_x, n_z$$ |


## 문제 정의 Problem definition
본격적으로 문제를 해결하기에 앞서, 칼만 필터로 해결하려는 문제에 대해서 좀 더 정확하게 묘사해보겠습니다.

우리가 처한 상황은 아래와 같습니다.

$$
\begin{cases}
\begin{aligned}
x_{k+1} &= F x_k + G u_k + w_k \\
z_{k+1} &= H x_k + v_k \\
\end{aligned}
\end{cases}
$$

위 식이 묘사하는 바는 아래와 같습니다.
* 상태 변수 $$x_k$$의 다음 상태 $$x_{k+1}$$는 아래 요소들에 의해 결정됩니다.
  * 현재 상태 $$x_k$$가 일정한 전이 방식$$F$$에 의한 영향
  * 입력 $$u_k$$가 일정한 제어 방식$$G$$에 의한 영향
  * 노이즈 $$w_k$$에 의한 영향
* 측정값 $$z$$는 아래 요소들에 의해 결정됩니다.
  * 현재 상태 $$x_k$$가 일정한 측정 방식$$H$$에 의한 영향
  * 노이즈 $$v_k$$에 의한 영향

즉, 우리는 상태를 직접적으로 계산하거나 관측할 수 없고,\
노이즈 $$w, v$$가 섞인 값들만을 계산하거나 관측할 수 있습니다.


여기서 한가지. 노이즈 $$w$$와 $$v$$는 가우시안 확률분포를 따른다고 가정합니다.\
다시말해 각 노이즈의 평균은 0입니다.


## 알고리즘 흐름 Algorithm flow
칼만 필터는 크게 2가지 행정으로 구성되어 있는데, 하나는 예측(Predict)이고,\
다른 하나는 갱신(Update)입니다. 이를 주지하고 아래 도식과 알고리즘 개요를 살펴봅시다.

(알고리즘 도식)
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

## 주요 5개 식 Five main equations
칼만 필터는 결국 아래 5개 식으로 구현됩니다.\
즉, 이 5개 식이 칼만 필터의 전부이며, 이 식이 이해가 된다면 무리없이 응용 및 구현이 가능합니다.

$$
\begin{align}
\hat{x}_{\bar{k}} &= F\hat{x}_{k-1} + Gu_k \tag{1} \label{predictor}\\
P_{\bar{k}} &= AP_{k-1}A^T+Q \tag{2} \label{covariance predictor} \\
K_k &= P_{\bar{k}}H^T(HP_\bar{k}H^T+R)^{-1} \tag{3} \label{kalman gain}\\
\hat{x}_{k} &= \hat{x}_{\bar{k}} + K_k(z_{k}-H\hat{x}_{\bar{k}}) \tag{4} \label{updator}\\
P_{k} &= (I-K_kH)P_{\bar{k}} \tag{5} \label{covariance updator}
\end{align}
$$

| 단계 Step | 식 Equation | 설명 Description |
| :---: |:--- | :--- |
| Predict | $$\hat{x}_{\bar{k}} = F\hat{x}_{k-1} + Gu_k $$ | 상태 예측 State predict equation |
| | $$P_{\bar{k}} = AP_{k-1}A^T+Q $$ | 공분산 예측 Covariance predict equation |
| Update | $$K_k = P_{\bar{k}}H^T(HP_\bar{k}H^T+R)^{-1} $$ | 칼만 게인 Kalman gain |
| | $$\hat{x}_{k} = \hat{x}_{\bar{k}} + K_k(z_{k}-H\hat{x}_{\bar{k}}) $$ | 상태 갱신 State update equation |
| | $$P_{k} = (I-K_kH)P_{\bar{k}} $$ | 공분산 갱신 Covariance update equation |


## 공분산 $$P, Q, R$$

앞서 언급하였듯, 칼만 필터는 물리 법칙을 이용하여 계산한 값(예측값)과 측정값을 적절히 합쳐서 추정값을 계산합니다.\
이를 위해서는 예측값과 측정값 그리고 추측값 모두에 대해서 각각 얼마나 **믿을 만 한지**에 대한 정량적인 정보가 있어야 하겠죠.

이에 대한 정보가 담겨있는 변수가 바로 $$P, Q, R$$입니다.\
이 $$P, Q, R$$은 모두 (공)분산에 대한 정보를 담고 있는 행렬로, 실제 값이 추측한 값과 차이가 날 확률을 기술합니다.

* $$P$$ : 추정한 상태 변수 $$\hat{x}$$와 실제 상태 변수 $$x$$간의 공분산. $$n_x, n_x$$
* $$Q$$ : 상태 전이 노이즈 $$w$$의 공분산. $$n_x, n_x$$
* $$R$$ : 측정 노이즈 $$v$$의 공분산. $$n_z, n_z$$

이 중 $$P$$는 매 Iteration마다 변화하지만, $$Q$$, $$R$$은 상수값으로 유지됩니다.\
$$Q$$, $$R$$값을 어떻게 정해야 할지에 대해서는 아래에서 설명하겠습니다.




# 설명 Description

칼만 필터 알고리즘의 세부 사항에 대해 알아봅시다.

(도식)\
칼만 필터는 예측과 갱신을 번갈아가면서 진행합니다.


## 예측 Predict

예측 단계에서는 이전 단계에 추정한 추정값을 시스템의 역학을 이용해 다음 상태와 그 공분산을 추측합니다.

이렇게 예측된 값은 $$k$$에 바를 씌워서 $$\bar{k}$$라고 표현합니다.\
($$k$$개의 관측이 이루어진 뒤 $$k+1$$번째 상태를 추측했다고 해서 $$k|k+1$$, 혹은 $$k, k+1$$라 표기하기도 합니다.) 

### 상태 예측 State predict equation

$$
\hat{x}_{\bar{k}} = F\hat{x}_{k-1} + Gu_k 
$$

보다시피, 예측은 State transition matrix $$F$$에 현재 상태를 곱한 후,\
Input vector $$u$$의 영향을 더하여 계산합니다.

다만 여기서 문제가 되는 것은 보통 어떤 시스템의 역학을 기술 할 때\
dynamic system을 이용해 그 변화량을 기술한다는 점입니다.


LTI system을 기술하는 state space equation은 다음과 같습니다.

$$
\begin{cases}
\begin{aligned}
\dot{x} &= A\hat{x} + Bu \\
y &= C\hat{x} + Du
\end{aligned}
\end{cases}
$$

위 연속미분방정식을 $$\Delta t$$에 대해서 $$A$$와 $$B$$를 적분해야\
Discrete한 state의 변화를 나타내는 $$F$$, $$G$$를 구할 수 있습니다.

아무튼 그래서 적분하면 다음과 같습니다.\
[과정](https://www.kalmanfilter.net/modeling.html#sde)

$$
\begin{cases}
\begin{aligned}
F &= e^{At} \\
&= I + A\Delta t + \frac{(A\Delta t)^2}{2!} + \frac{(A\Delta t)^3}{3!} + \cdots \\
\end{aligned}\\
G = \displaystyle \int^{\Delta t}_0 e^{At} dt B
\end{cases}
$$


### 공분산 예측 Covariance predict equation

$$P_{\bar{k}} = AP_{k-1}A^T+Q $$ 

공분산 예측은 말 그대로 공분산의 크기가 다음 상태에서는 어떻게 될 지 예측하는 식입니다.

식의 구성을 살펴보면, 다음 상태가 어떻게 변화할지 기술하는 $$F$$와,\
process noise에 따른 불안정성인 $$Q$$의 영향을 받음을 알 수 있습니다.

#### 유도 Derivation



## 갱신 Update

갱신 파트에서는 아까 예고한대로 예측값과 관측값을 잘 절충해서 값을 추산하는 단계입니다.


### Gaussian function의 곱에 대해서

Gaussian function의 곱은 Gaussian function입니다.

### 칼만 게인 Kalman gain

$$K_k = P_{\bar{k}}H^T(HP_\bar{k}H^T+R)^{-1} $$


#### 칼만 게인의 뜻 고찰

### 상태 갱신 State update equation 

$$\hat{x}_{k} = \hat{x}_{\bar{k}} + K_k(z_{k}-H\hat{x}_{\bar{k}}) $$


### 공분산 갱신 Covariance update equation 

$$P_{k} = (I-K_kH)P_{\bar{k}} $$


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