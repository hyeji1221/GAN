# 그리기

지금까지는 완전히 새로운 샘플을 생성하였음 (VAE, GAN) -> 어떤 입력도 없이 잠재 공간에서 샘플링한 벡터를 이용해 이미지 생성

**스타일 트랜스퍼** : 주된 목적은 스타일 이미지가 주어졌을 때 같은 부류에 속한 것 같은 느낌을 주도록 베이스 이미지 변환하는 모델 훈련

-> 이미지에 내재된 분포를 모델링 하는 것이 아닌 스타일을 결정하는 요소만 추출하는 베이스 이미지에 주입

## CycleGAN

- 이 논문은 샘플 쌍으로 구성된 훈련 세트 없이도 참조 이미지 세트의 스타일을 다른 이미지로 복사하는 모델을 훈련 할 수 있는 방법을 보임
- pix2pix 같은 이전 스타일 트랜스퍼는 훈련 세트의 각 이미지가 소스와 타깃 도메인에 모두 존재해야 함
- pix2pix는 한 방향으로 작동하지만 CycleGAN은 양방향으로 동시에 모델을 훈련 -> 모델이 소스에서 타깃으로 뿐만 아니라 타깃에서 소스로 이미지 변환 가능

## 첫 번째 CycleGAN

도메인 A를 B로, B를 A로 바꾸는 것이 목적

### 개요

CycleGAN은 실제 4 개의 모델로 구성 -> 2개의 생성자와 2개의 판별자

- 첫 번째 생성자 : 도메인 A의 이미지를 도메인 B로 바꾼다
- 두 번째 생성자 : 도메인 B의 이미지를 도메인 A로 바꾼다
- 첫 번째 판별자 : 도메인 A의 진짜 이미지와 두 번째 생성자가 만든 가짜 이미지를 구별할 수 있도록 훈련
- 두 번째 판별자 : 도메인 B의 진짜 이미지와 첫 번째 생성자가 만든 가짜 이미지를 구별할 수 있도록 훈련

### 생성자 (U-Net)

- 일반적으로 cycleGAN 생성자는 U-Net 또는 ResNet 두 형태 중 하나를 선택

VAE와 비슷한 방식인 U-Net은 다운샘플링과 업샘플링으로 구성

- 다운샘플링 : 이미지를 공간 방향으로는 압축하지만 채널 방향으로는 확장
- 업샘플링 : 공간 방향으로는 표현을 확장시키지만 채널의 수 감소

다운샘플링에서 이미지 스타일을 감지하고 업샘플링에서 스킵 연결을 통해 (원본 이미지 크기로)콘텐츠를 복원한다. 

### 판별자

지금까지 본 판별자는 하나의 숫자를 출력 -> 입력한 이미지가 진짜일 예측 확률

CycleGAN의 판별자는 숫자 하나가 아닌 16 x 16 크기의 채널 하나를 가진 텐서를 출력

판별자는 내용이 아닌 스타일로 두 이미지가 다른지 구분해야 한다.

### CycleGAN 컴파일

목적 : 도메인 A의 이미지를 도메인 B의 이미지로 혹은 그 반대로 바꾸는 일련의 모델을 훈련하는 것

-> **네 개의 다른 모델을 다음과 같이 컴파일 해야 함**

- 첫 번째 생성자 : 도메인 A의 이미지를 도메인 B의 이미지로 바꾸는 것을 학습
- 두 번째 생성자 : 도메인 B의 이미지를 도메인 A의 이미지로 바꾸는 것을 학습
- 첫 번째 판별자 : 도메인 A의 진짜 이미지와 두 번째 생성자가 만든 가짜 이미지의 차이점을 학습
- 두 번째 판별자 : 도메인 B의 진짜 이미지와 첫 번째 생성자가 만든 가짜 이미지의 차이점을 학습

입력(각 도메인의 이미지)은 출력(이진 타깃, 도메인 이미지이면 1, 아니면 0)

생성자는 쌍을 이루는 이미지가 데이터셋에 없기 때문에 바로 컴파일 불가 -> 아래 세 가지 조건으로 생성자 동시에 평가

- 유효성 : 각 생성자에서 만든 이미지가 대응되는 판별자를 속이는가?
- 재구성 : 두 생성자를 교대로 적용하면 원본 이미지를 얻는가?
- 동일성 : 각 생서자를 자신의 타깃 도메인에 있는 이미지에 적용했을 때 이미지가 바뀌지 않고 그대로 남아 있는가?

### CycleGAN 훈련

판별자와 생성자를 교대로 훈련

판별자를 훈련하기 위해 먼저 생성자를 사용해 가짜 이미지 배치를 만든 후 가짜 이미지와 진짜 이미지 배치로 각 판별자를 훈련

## CycleGAN으로 모네 그림 그리기

코드 참조

### 생성자 (ResNet)

**잔차 네트워크 또는 ResNet** 

- 이전 층의 정보를 네트워크의 앞쪽에 있는 한  개 이상의 층으로 스킵한다는 점에서 U-Net과 비슷 
- 그러나 다운샘플링 층을 이에 상응하는 업샘플링 층으로 연결하여 U 모양 대신 잔차 블록을 차례대로 쌓아 구성
- 각 블록은 다음 층으로 출력을 전달하기 전 입력과 출력을 합하는 스킵 연결을 가짐
- ResNet 구조는 수백 수천 개의 층도 훈련할 수 있다고 알려져 있음
- 앞쪽의 층에 도달하는 그레이디언트가 작아져 매우 느리게 훈련되는 그레이디언트 소실 문제가 없다
- 층을 추가해도 모델의 정확도 떨어뜨리지 않음

## 뉴럴 스타일 트렌스퍼

**뉴럴 스타일 트랜스퍼** : 훈련 세트를 사용하지 않고 이미지의 스타일을 다른 이미지로 전달

- 베이스 이미지 + 스타일 이미지 -> 합성 이미지

아래 세 부분으로 구성된 손실 함수의 가중치 합을 기반으로 작동

- 콘텐츠 손실 : 합성된 이미지는 베이스 이미지의 콘텐츠를 동일하게 포함해야 함
- 스타일 손실 : 합성된 이미지는 스타일 이미지와 동일한 일반적인 스타일을 가져야 함
- 총 변위 손실 : 합성된 이미지는 픽셀처럼 격자 무늬가 나타나지 않고 부드러워야 함

경사하강법을 이용해 이 손실을 최소화 -> 반복이 진행되면서 손실은 점차 줄어들어 베이스 이미지의 콘텐츠와 스타일 이미지를 합친 합성 이미지를 얻게 된다

### 콘텐츠 손실

- 콘텐츠 손실은 콘텐츠의 내용과 전반적인 사물의 배치 측면에서 두 이미지가 얼마나 다른지를 측정
- 비슷한 이미지를 담은 두 이미지는 완전히 다른 이미지를 포함하는 두 이미지보다 손실이 작아야 함
- 베이스 이미지와 현재 합성된 이미지에 대한 출력을 계산하여 그 사이의 평균 제곱 오차를 측정한 것이 콘텐츠 손실 함수가 된다

### 스타일 손실

- 스타일이 비슷한 이미지는 특정 층의 특성 맵 사이에 상관 관계 패턴을 가진다는 아이디어를 기반
- 두 개의 특성 맵이 얼마나 동시에 활성화되는지 수치적으로 측정하기 위해 특성 맵을 펼치고 스칼라곱을 계산
- 그람 행렬 : 층에 있는 모든 특성 사이의 스칼라곱을 담은 행렬

#### 스타일 손실 계산

1. 베이스 이미지와 합성된 이미지에 대해 네트워크의 여러 층에서 그람 행렬 계산
2. 두 그람 행렬의 제곱 오차 합을 사용하여 유사도 비교

### 총 변위 손실

- **총 변위 손실** : 단순히 합성된 이미지에 있는 잡음을 측정한 것
- 변위 손실은 생성된 이미지의 픽셀에 격자무늬가 나타나지 않고 연속성을 갖도록 도와줌

#### 총 변위 손실 계산

1. 이미지의 잡음을 측정하기 위해 오른쪽으로 한 픽셀 이동 후 원본 이미지와 이동한 이미지 간 차이를 제곱 후 더한다
2. 균형을 맞추기 위해 동일한 작업을 한 픽셀 아래로 이동하여 수행
3. 이 두 항의 합이 총 변위 손실



