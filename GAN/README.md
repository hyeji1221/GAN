# GAN - 생성적 적대 신경망

## GAN 소개

적대적인 **생성자와 판별자** 네트워크 두 개가 경쟁하는 것

- 생성자 : 랜덤한 잡음을 원본 데이터셋에서 샘플링한 것처럼 보이는 샘플로 변환

- 판별자 : 원본 데이터셋에서 추출한 샘플인지 생성자가 만든 가짜인지 구별

**반복 과정**

1. 처음에 생성자는 잡음투성이인 이미지를 출력하고 판별자는 무작위로 예측

2. 생성자는 판별자를 속이는데 더 능숙해지고 판별자가 가짜 샘플을 정확하게 구별하는 능력 유지하도록 적응해 나간다.

3. 다시 생성자가 판별자를 속이기 위한 새로운 방법을 찾도록 만든다

**GAN의 핵심** : 두 네트워크를 어떻게 교대로 훈련하는지에 있다

## 첫번째 GAN

### 판별자

**판별자의 목표** : 이미지가 진짜인지 가짜인지 예측하는 것 -> 지도학습의 분류 문제 

-> 합성곱 층을 쌓고 완전 연결 층을 출력층으로 놓은 네트워크 구조 사용 가능

### 생성자

GAN의 생성자는 **VAE의 디코더**와 동일한 목적 수행 -> 잠재 공간의 벡터를 이미지로 변환

**UpSampling2D + Conv2D와 Conv2DTranspose** 모두 원본 이미지 차원으로 되돌리는 데 사용할 수 있는 변환 방법 -> 두 방법 모두 사용해보고 더 잘 맞는것 선택하기

### GAN 훈련

- 훈련 세트에서 진짜 샘플을 랜덤하게 선택하고 생성자의 출력을 합쳐서 훈련 세트를 만들어 판별자를 훈련
- 입력은 랜덤하게 생성한 100차원 잠재 공간 벡터이고 출력은 1인 훈련 배치를 만들어 전체 모델 훈련
- 손실 함수는 이진 크로스엔트로피 손실
- 학습률은 GAN 문제에 따라 주의 깊게 튜닝해야 하는 파라미터

진짜 이미지가 잠재 공간의 어떤 포인트에 매핑되는지 알려 주는 훈련 세트가 없기 때문에 생성자 훈련이 더 어려움 대신 판별자를 속이는 이미지 생성       

​	-> 이미지가 판별자의 입력으로 주입될 때 1에 가까운 값이 출력되어야 함

전체 모델 훈련 시 생성자의 가중치만 업데이트 되도록 **판별자의 가중치를 동결**해야 함       

​	-> 동결하지 않으면 생성된 이미지를 진짜라고 여기도록 조정됨

판별자가 약해서가 아닌 생성자가 강해서 생성된 이미지가 1(진짜 이미지)에 가까운 값으로 예측되어야 함

원본 픽셀 이외에는 모델에 어떤 추가적인 정보를 제공하지 않고도 랜덤한 잡음을 의미 있는 것으로 바꾸는 것이 가능

## GAN의 도전 과제

GAN이 생성 모델링 분야의 커다란 혁신이지만 훈련이 어렵다

### 진동하는 손실

판별자와 생성자의 손실이 장기간 안정된 모습을 보여주지 못하고 큰 폭으로 진동하기 시작할 수 있음

그러나 심하게 출렁이는 것이 아닌 손실이 안정되거나 점진적으로 증가하거나 감소하는 형태를 보여야 함

### 모드 붕괴

**모드 붕괴**는 생성자가 판별자를 속이는 적은 수의 샘플을 찾을 때 일어난다. -> 한정된 이 샘플 이외 다른 샘플 생성 불가

이유 : 생성자는 판별자를 항상 속이는 하나의 샘플을 찾으려는 경향이 있고 잠재 공간의 모든 포인트를 이 샘플에 매핑 가능 -> 손실 함수의 그레이디언트가 0에 가까운 값으로 무너진다는 뜻

### 유용하지 않은 손실

딥러닝 모델은 생성자의 손실이 작을 수록 생성된 이미지 품질이 더 좋다고 생각할 수 있다

그러나 생성자는 현재 판별자에 의해서만 평가되고 판별자는 계속 향상 되기에 훈련 과정의 다른 지점에서 평가된 손실 비교불가

-> 생성자의 손실과 이미지 품질 사이의 연관성 부족은 GAN 훈련 과정을 모니터링하기 어렵게 만든다

### 하이퍼파라미터

간단한 GAN이여도 튜닝해야 할 하이퍼파라미터의 개수가 상당히 많음

-> 판별자와 생성자의 전체 구조, 배치 정규화, 드롭아웃, 학습률, 활성화 층, 합성곱 필터, 커널 크기, 배치 크기, 스트라이드, 잠재 공간 크기 결정 등

GAN은 이런 파라미터의 작은 변화에도 매우 민감

## WGAN - 와서스테인 GAN

와서스테인 GAN은 안정적인 GAN 훈련을 위한 첫 번째 큰 발전 중 하나

이 논문의 저자 들은 아래 두 가지 속성을 가지는 GAN 훈련 방법 제시

- 생성자가 수렴하는 것과 품질을 연관 짓는 의미 있는 손실 측정 방법
- 최적화 과정의 안전성 향상

이 논문에서는 이진 크로스엔트로피 대신 새로운 손실함수 소개

### 와서스테인 손실

#### 이진 크로스엔트로피 손실

판별자를 훈련하기 위해 진짜 이미지에 대한 예측과 타깃 y=1을 비교하고 생성된 이미지에 대한 예측과 타깃 y=0을 비교하여 손실 계산

#### GAN 판별자 손실 최소화

생성자를 훈련하기 위해 생성된 이미지에 대한 예측과 타깃 y=1을 비교하여 손실 계산

#### GAN 생성자 손실 최소화

와서스테인 손실은 1과 0 대신에 y=1, y=-1 사용

판별자의 마지막 층에서 시그모이드 활성화 함수를 제거하여 [0, 1] 범위에 국한되지 않고 [-∞, ∞] 범위의 어떤 숫자도 될 수 있도록 만듬 -> WGAN의 판별자를 보통 **비평자**라고 부름

#### 와서 스테인 손실

비평자를 훈련하기 위해 진짜 이미지에 대한 예측과 타깃 y=1을 비교하고 생성된 이미지에 대한 예측 y=-1을 비교하여 손실 계산

#### WGAN 비평자 손실 최소화

WGAN 비평자는 진짜 이미지에 대한 점수를 높임으로써 진짜 이미지와 생성자에 대한 예측 사이의 차이를 최대화한다

WGAN 생성자를 훈련하려면 생성된 이미지에 대한 예측과 타깃 y=1을 비교하여 손실 계산

#### WGAN 생성자 손실 최소화

WGAN의 비평자와 생성자를 훈련하는 모델을 컴파일할 때 이진 크로스엔트로피 대신 와서스테인 손실 지정 가능

WGAN은 더 작은 학습률을 사용하는 경우가 많음

### 립시츠 제약

비평자가 [-∞, ∞] 범위의 숫자를 출력하므로 추가적인 제약을 가하는 것이 필요

-> 특히 비평자는 1-립시츠 연속 함수여야 한다.

**1-립시츠** : 비평자는 하나의 이미지를 하나의 예측으로 변환하는 함수로 임의의 두 지점의 기울기가 어떤 상수값 이상 증가하지 않는 함수 (이 상수가 1일 때 1-립시츠)

### 가중치 클리핑

판별자의 가중치를 작은 범위인 [-0.01, 0.01] 안에 놓이도록 훈련 배치가 끝난 후 가중치 클리핑을 통해 립시츠 제약을 부과

### WGAN 훈련

와서스테인 손실 함수를 사용할 때 생성자가 정확히 업데이트 되도록 판별자를 훈련하여 수렴시켜야 함 

-> GAN과 대조적 (기본 GAN은 그레이디언트 소실을 피하기 위해 판별자가 너무 강해지지 않도록 하는것이 중요함)

WGAN에서는 생성자를 업데이트하는 중간에 판별자를 여러 번 훈련하여 수렴에 가깜게 만들 수 있음 

-> 일반적으로 생성자 한 번에 판별자 다섯 번

### WGAN 분석

#### :star: 기본 GAN과 WGA의 핵심 차이점

- WGAN은 와서스테인 손실 사용
- WGAN은 진짜 레이블 1, 가짜에는 레이블 -1 사용하여 훈련
- WGAN 비평자의 마지막 층에는 시그모이드 활성화 함수가 필요 없음
- 매 업데이트 후 판별자의 가중치를 클리핑
- 생성자를 업데이트 할 때마다 판별자를 여러 번 훈련

WGAN의 주요 단점 중 하나 : 비평자에서 가중치를 클리핑하기에 학습 속도 크게 감소

강한 비평자는 WGAN 성공의 중심 -> 정확한 그레이디언트가 없다면 생성자가 가중치를 바꾸어야 할지 학습 불가

## WGAN-GP

WGAN의 파생 논문 중 하나로 **와서스테인 GAN-그레이디언트 패널티 (WGAN-GP)**

생성자는 WGAN과 정확히 동일하나 비평자의 정의와 컴파일 단계를 바꾸어야 한다.

**WGAN의 비평자를 WGAN-GP의 비평자로 변환하기 위해 바꾸어야 할 세 가지**

- 비평자 손실 함수에 그레이디언트 페널티 항 포함
- 비평자의 가중치 클리핑 X
- 비평자에 배치 정규화 사용 X

### 그레이디언트 패널티 손실

비평자에 립시츠 제약을 강제하는 방법으로 비평자의 가중치를 클리핑하는 대신 비평자의 그레이디언트 노름이 1에서 크게 벗어날 때 모델에 페널티를 부과하는 항을 손실 함수에 포함시켜 제약을 직접 부과

그레이디언트 패널티 손실은 입력 이미지에 대한 예측의 그레이디언트 노름과 1 사이의 차이를 제곱한 것

한 쪽으로 치우치지 안힉 위해 진짜 이미지와 가짜 이미지 쌍을 연결한 직선을 따라 무작위로 포인트를 선택해 보간한 이미지들 사용