---
layout: posts
title: 동기화 비용
date:   2021-05-02 17:50:00 +0900
comments: true
categories: note
---

멀티 코어 시스템에서 애플리케이션의 처리 성능을 향상 시키기 위한 방법으로 스레드를 이용하여 작업을 병렬로 수행하는 방법이 있다. 

하나의 자원을 다수의 스레드가 공유하며, 해당 자원이 **thread safe** 성격을 가져야 한다면 올바른 처리를 위해, 공유자원을 임계영역에서 처리 하도록 해야한다.  
임계영역은 하나의 스레드만 접근할 수 있는 공간으로, 멀티 스레드 환경에서 임계영역으로 접근하는 순간에는 동기화 되어 처리된다.

즉, 임계영역은 자원을 멀티 스레드로부터 안전하게 보호하지만, 베타적 접근만 허용하기 때문에 처리성능에 부정적인 영향을 주게된다.

병렬처리시 임계영역의 크기에 따라 처리 속도가 향상되는 범위가 다를 것이다. (*CPU 주기에만 의존하는 작업이라는 가정하에*)  
멀티 스레드 환경에서 얻을 수 있는 처리속도는 암달의 방정식을 이용하여 이론적인 수치를 구할 수 있다.

## 암달의 법칙

$$
S = \frac{1}{(1 - p) + \frac{P}{N}}
$$

P = 병렬로 실행되는 코드의 백분율  
N = 스레드의 개수

멀티 스레드에 노출된 코드중 임계영역에서 동작하는 코드의 비율이 20%(병렬 코드 80%), 스레드의 개수가 4이면 이렇게 계산된다.

$$
\begin{align}
    S & =\frac{1}{(1- 0.8) + \frac{0.8}{4}} \\[2ex]
    & = \frac{1}{0.2 + 0.2} \\[2ex]
    & =  2.5
\end{align}
$$
