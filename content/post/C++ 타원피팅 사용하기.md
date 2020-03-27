---
title: "C++ 타원피팅 사용하기"
date: 2020-03-10T15:55:12+09:00
categories: ["programming"]
subcategories: ["c++","opencv","image processing"]
tags: ["C++", "타원 피팅"]
cover: ""
draft: false
---

개발을 진행하다 보면 산재한 데이터를 하나의 공식으로 표현해야 하는 경우가 많이 생긴다. 예를 들면 이미지의 특징점들을 연결한다거나, 불규칙해 보이는 실험값들을 근사한다거나 하는 일들 말이다. 이때 유용하게 쓸만한 것이 타원피팅(Ellipse Fitting)이다. 구현도 어렵지 않기 때문에 요새는 일단 원과 비슷해 보이면 한번 적용해보는 정도로 자주 쓰고 있다. 

# 최소자승법을 활용한 피팅
주어진 데이터들로 하나의 방정식을 만드는 가장 쉬운 방법은 `최소자승법`을 이용하는 것이다. 자승이라는 단어가 익숙하지 않다면 `최소제곱법`이라고 불러도 무방하다. 최소제곱법이라는 단어에서 알 수 있듯 우리는 무언가를 제곱했을 때 최소가 되는 방정식을 찾을 것이다. 무엇을 제곱하느냐? 바로 주어진 값과 그 값으로 근사한 방정식의 오차를 제곱한다. 아래 그래프를 보자.   

![](/images/blog_image/Linear_least_squares.svg#center75)   
우리에게 위 그래프의 빨간 점과 같은 데이터가 주어졌다고 했을 때 이 데이터들을 가장 잘 만족하는 2차 방정식을 찾고 싶다면 어떻게 해야 할까? 단순하게 생각해보면 모든 점을 지나는 2차 방정식을 찾으면 된다. 그러나 알다시피 2차 방정식으로 주어진 점을 모두 지나는 것은 불가능하다. 따라서 우리가 할 수 있는 최선은 방정식의 값과 주어진 데이터 값의 오차를 제곱하여 합한 값이 가장 적은 방정식을 찾는 것이다. 이것이 바로 최소제곱법의 원리다.   
그럼 이제 이 방정식을 구해보자. 과정은 간단하다 우리가 찾고자 하는 방정식을 행렬 형태로 표현한 뒤 역행렬을 이용해 방정식의 계수를 구하면 된다. 이를 수식으로 표현하면 다음과 같다.

<center>$\LARGE ax_1^2 + bx_1 + c = y_1$</center>
<center>$\LARGE ax_2^2 + bx_2 + c = y_2$</center>
<center>$\LARGE \vdots$</center>
<center>$\LARGE ax_n^2 + bx_n + c = y_n$</center> 

이를 행렬식으로 표현하면 아래와 같다.   

<center>$\LARGE \begin{align}
A = 
\begin{bmatrix}
    \mathbf{x_1^2} & \mathbf{x_1} & \mathbf{1} \newline
    \vdots &  \vdots  &\vdots \newline
    \mathbf{x_n^2} & \mathbf{x_n} & \mathbf{1} \newline
   \end{bmatrix} 
, \;\;
X = 
\begin{bmatrix}
    \mathbf{a} \newline
    \mathbf{b} \newline
    \mathbf{c} \newline
   \end{bmatrix}
, \;\;
B=
\begin{bmatrix}
    \mathbf{y_1} \newline
    \vdots \newline
    \mathbf{y_n} \newline
   \end{bmatrix}
\end{align}$</center> 

 행렬식  $\large AX = B$는 $\large X ={(A^T A)}^{-1} A^TB$를 만족하므로 우리는 $\large A$의 유사역행렬 $\large pinv(A)$만 계산하면 2차 방정식의 계수 $\large a,b,c$를 구할 수 있다.  만약 위 과정이 잘 이해되지 않거나 더 자세한 설명을 원하는 분들은 이 [블로그](https://darkpgmr.tistory.com/56)를 방문해 보길 바란다. 이 글보다 훨씬 자세하고 수준 높은 설명을 찾을 수 있을 것이다.   
    
## 최소자승법을 활용한 타원피팅   
이제 우리가 원하는 타원을 피팅해보자. 타원의 방정식을 일반형으로 표현하면 아래와 같다.   

<center>$\LARGE 
\begin{matrix} ax^2 + bxy + cy^2 + dx + ey + f &=& 0 \newline
x^2 + b'xy + c'y^2 + d'x + e'y + f' &=& 0
\end{matrix} \tag{Dividing a}
$</center>   

이제 $\large x^2$을 우변으로 넘긴 뒤에 행렬식으로 표현하면 다음과 같다.

<center>$\LARGE \begin{align}
\begin{bmatrix}
    \mathbf{x_1y_1} & \mathbf{y_1^2} & \mathbf{x_1} & \mathbf{y_1} & \mathbf{1} \newline
    \vdots & \vdots  &\vdots & \vdots & \vdots \newline
    \mathbf{x_ny_n} & \mathbf{y_n^2} & \mathbf{x_n} & \mathbf{y_n} & \mathbf{1} \newline
   \end{bmatrix} 
\begin{bmatrix}
    \mathbf{b'} \newline
    \mathbf{c'} \newline
    \mathbf{d'} \newline
    \mathbf{e'} \newline
    \mathbf{f'} \newline
   \end{bmatrix} =
\begin{bmatrix}
    \mathbf{-x_1^2} \newline
    \vdots \newline
    \mathbf{-x_n^2} \newline
   \end{bmatrix}
\end{align}$</center> 

<center>$\LARGE
\begin{matrix} AX &=& B \newline
X &=& {(A^T A)}^{-1} A^T B
\end{matrix}
$</center> 

아래는 이를 구현한 C++ 코드이다.   
   

{{<gist ellipseFitting>}}


# OpenCV의 fitEllipse 활용하기   
위와 같은 방식으로 간단하게 타원 피팅을 시도해 볼 수 있지만 OpenCV를 사용하고 있다면 굳이 따로 함수를 만들 필요가 없다. OpenCV에는 이미 `fitEllipse` 메소드가 구현되어 있기 때문이다. fitEllipse는 `cv::Point`를 매개변수로 받아 `RotatedRect` 클래스를 반환한다.   
RotatedRect 클래스는 세 가지 인스턴스를 가지고 있는데 타원의 중심점인 __center__, 타원의 장축과 단축을 알 수 있는 __size__, 마지막으로 타원의 회전 각도를 나타내는 __angle__이다. 이 세 가지만 알고 있으면 타원에 대한 모든 정보를 유추할 수 있다.   
![](/images/blog_image/ellipse.png#center75)
<center>_a = 장축의 반지름, b = 단축의 반지름, θ = 회전 각도_</center>   
바로 코드를 살펴보자.   

{{<gist ellipseFittingOpenCV>}}   
   
여기서 회전 각도 $\large θ$를 구하는 부분은 주의할 점이 있다. RotatedRect의 angle은 x축과 장축이 이루는 각도를 시계방향으로 회전하는 degree로 표시한다. 즉 아래 그림과 같다.   
   
![](/images/blog_image/rotatedRect.png#center75)   
<center>_나의 삽질을 비웃는 듯한 그래프.._</center>    

따라서 우리가 앞서 구현한 ellipseFitting함수와 동일한 결과를 얻기 위해선 180도에서 angle을 빼고 radian으로 변환해주어야 한다. ~~이걸 몰라서 한참 헤맸다~~   

![](/images/blog_image/data.JPG#center75) 
   
이제 구현 결과를 한번 살펴보자, 위와 같은 데이터를 최소자승법으로 타원피팅한 결과는 아래와 같고
   
![](/images/blog_image/최소자승.JPG#center75) 
   
OpenCV의 fitEllipse를 이용해 타원피팅한 결과는 아래와 같다.
   
![](/images/blog_image/fitEllipse.JPG#center75) 
   
보다시피 나름 괜찮은 결과를 보여주고 있다.  
   
![](/images/blog_image/박수.gif#center75) 
   

그러나 현실에서는 데이터에 포함된 수많은 오차 때문에 저렇게 깨끗하게 피팅되는 경우는 거의 없다. 최소자승법의 한계가 바로 이러한 오차들을 걸러내기 어렵다는 것인데, 이러한 데이터들을 다루기 위한 방법은 이어지는 포스팅에서 다룰 예정이다.