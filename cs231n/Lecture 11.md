# CS231N Lecture 11

> 주제 : Segmentation, Localization, Detection

### Other Computer Vision Tasks

- Semantic Segmentation 
- Classification + Localization
- Object Detecton
- Instance Segmentation

# Semantic Segmentation

> 입력은 이미지, 출력으로 이미지의 모든 픽셀에 카테고리를 정함

- classification 처럼 semantic segmentation에도 카테고리가 존재
- 이미지 전체에 카테고리 하나가 아니라 모든 픽셀에 카테고리가 매겨짐
- Semantic segmentation은 픽셀의 카테고리만 정해지기 때문에 카테고리가 정해진 픽셀 덩어리만 뽑아낼 수 있다.
- 이미지를 아주 작은 단위로 쪼개어 classification 하는 것은 비용측면에서 비효율적이다.

**Fully Convolutional Network**

- 3x3 zero padding을 수행하는 ConvLayer들을 쌓아올리면 이미지의 공간정보를  손실하지 않음
- 네트워크 출력 Tensor는 `C X H X W`
  - C :  카테고리 수
  - 출력 Tensor는 입력 이미지의 모든 픽셀 값에 대해 Classification scores를 매긴 값
  - Conv Layer만 쌓아올린 네트워크를 통해 계산 가능
  - 모든 픽셀의 `Classification loss` 를 계산하고 평균값을 구함
  - 이후 `back propagation` 을 수행



Q. Training data는 어떻게 만드는가?

> 비용이 아주 큰 작업. 입력 이미지의 모든 피셀에 대해서 레이블링을 해야함

Q. Loss function은 어떻게 작성하는가?

> 이 문제에서는 모든 픽셀을 classification한다. 따라서 출력의 모든 픽셀에 `cross entropy` 를 적용한다. 출력의 모든 픽셀에는 `Ground Truth` 가 존재하고, 
>
> 출력의 모든 픽셀과 `Ground Truth`  간의 `Cross Entropy` 를 계산한다.

Q. 모든 픽셀의 카테고리를 알고 있다고 가정해야 하는가?

> 모든 픽셀의 카테고리를 알고 있다는 가정이 있어야 한다.

 이 네트워크의 경우에는 입력 이미지의 spatial size를 계속 유지시켜야 하기 때문에, 비용이 아주 크다.

-> 이 네트워크에 고해상도의 이미지가 입력으로 들어오면 계산량과 메모리가 엄청 커서 감당할 수 없음



## Downsampling / Upsampling

> 특징 맵을 `Downsampling` / `Upsampling` 한다.

Spatila resolution 전체를 가지고 Convolution을 수행하기 보다는 Original Resolution에서는 convolutional layer는 소량만 사용한다.

`Max pooling` , `Stride Convolution` 등으로 특징맵을 다운샘플링 한다.

> 결론적으로 `convolution in downsampling`  을 반복해서 수행한다.

### image classification과 다른 점

- image classification에서는 FC-Layer가 있었지만 `Semantic Segmentation`  에서는 `Spartial Resolution` 을 다시 키운다.
  - 결국 다시 입력 이미지의 해상도와 같아짐
  - 이 방법을 통해 네트워크 `lower resolution`  을 처리하도록 하여 네트워크를 더 깊게 만들 수 있다.

### UpSampling을 위한 전략

- unpooling
  - Nearest Neighbor unpooling
  - Bed nails Unpooling : unpooling region에만 값을 복사하고 다른 곳에는 모두 0을 채워 넣음 
  - Max Unpooling
    - 대부분의 네트워킄 대칭적인 경향이 있음
    - unpooling과 pooling을 연관짓는 방법
    - downSampling시에 Max Pooling에 사용했던 요소를 잘 기억하고 있어야함
    - 같은 자리에 값을 넣는 것이 아니라 이전 Maxpooling에서 선택된 위치에 맞게 넣어준다.
    - 남은 자리는 0으로 채워준다.

Q. 왜 이 방법이 좋은 방법이고 중요한가?

> Semantic segmentation에서는 모든 픽셀들의 클래스를 모두 잘 분류해야 한다.
>
> 예측한 Segmentation 결과에서 객체들 간의 디테일한 경계가 명확할수록 좋음
>
> Maxpooling을 하게 되면 특징맵의 비균진성이 발생 
>
> MaxPooling 후의 특징 맵만 봐서는 이 값들이 Receptive field 중 어디에서 왔는지 알 수 없음
>
> UnPool 시에 기존 MaxPool에서 뽑아온 자리로 값을 넣어주면 공간 정보를 조금은 더 디테일하게 다룰 수 있음

Q. MaxUnppoling이 Backdrop을 더 수월하게 해주나요?

> 이 방법이 Backprob dynamics를 크게 바꾸지는 않는다.
>
> Maxpool indices를 저장하는 비용이 그렇게 크지 않음



## Transpose Convolution

= **deconvolution**

= **upconvolution**

= **stride 1/2 convolution** (input : ouput = 1 : 2)

= **fractionally strided convolution**

= **backwards strided convolution** 

(transpose convolution의 forward pass를 수학적으로 계산해보면 일반적인 Convolution의 backward pass와 수식이 동일하기 때문)





 **Strided convolution** : 어떤 식으로 Downsampling을 해야할지 네트워크가 학습할 수 있다. 

> `Strided Convolution` 과 비슷하게 Unsampling에서도 학습 가능한 방법

- 내적을 수행하지 않음
- 특징맵에서 값을 하나 선택
- 뽑아낸 스칼라값을 필터와 곱한다.
- 그리고 출력의 3x3 영역에 그 값을 넣는다.
- 입력값이 필터에 곱해지는 가중치 역할
- 입력에서 한 칸씩 움직이는 동안 출력에서는 두 칸씩 움직임
- 출력에서 Transpose convolution 간에 receptive field가 겹칠 수 있음
  - 겹치는 경우에는 간단하게 2개의 값을 더해준다.  

이 과정을 끝내면 학습 가능한 unsampling을 수행한 것

> 출력값 : 필터 * 입력(가중치)

> Spatial size를 키워주기 위해서 학습된 필터 가중치를 이용함



 Q. 왜 평균을 구하지 않고 더하나요?

> 그냥 더하는 이유는 transpose convolution 수식 때문 
>
> 하지만 분명히 sum은 문제가 될 수 있음
>
> Receptive field의 크기에 따라서 magnitudes가 달라짐

