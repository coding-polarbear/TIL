# U-Net
> Convolutional Networks for Biomedical Image Segmentation

CS231n 11강에서 말했던데로, 이미지 세그멘테이션을 위해서는 각각의 픽셀에  클래스 라벨이 할당되어야 한다.

![](https://postfiles.pstatic.net/MjAxODA4MDZfMjkg/MDAxNTMzNTUxOTUxOTU0.YzYd-ho-1jFLlBmDWRTlnxRjjKlA2XX0wmutkUXARrcg.r_RiV19V9ocbF_9jM_D9kze0TdFf5oWKY7rnZHQYLIUg.PNG.worb1605/image.png?type=w773)
### U-Net의 핵심
1. Convolution Encoder에 해당하는 Contraction Path + Convolution Decoder에 해당하는 Expanding Path의 구조로 구성 (해당 구조는 Fully Convolution + DeConvolution 구조의 조합)
2. Expanding Path에서 Upsampling 할 때, 좀 더 정확한 Localization을 하기 위해서 Contraction Path의 Feature를 Copy and Crop하여 Concat하는 구조
3. Data Agumentation

### 용어 정리
#### Patch
> 이미지 인식 단위

#### Contracting Path, Expanding Path
![](https://postfiles.pstatic.net/MjAxODA4MDZfOSAg/MDAxNTMzNTUyMzUxMjI0.BGLNzpU6JtmP8Jy43qpgLaSzAUWTCdtOiBSkFERltxcg.JZPXg332u0zTZLCv_OM0WYtdrgJQ7QzAba-zcrN1K14g.PNG.worb1605/image.png?type=w773)

* Contracting Path   
  * 이미지를 줄여 나가는 부분
* Expanding Path
  * 이미지를 키워나가는 부분

#### Context
> 이웃한 픽셀 간의 관계.

### Introduction
> 기존의 CNN들이 단순한 classification에서 주로 쓰였다면, U-Net은 Classification + Localization에서 주로 사용됨

##### 기존 논문의 문제점 해결
1. 기존에 검증했던 부분을 다시 검증하는 `sliding window` 방식과는 다르게, 이미 검증이 끝난 부분은 건너뛰고 다음 `patch` 부분부터 검증하기 때문에 속도면에서 유리함

2. Trade Off의 늪에 빠지지 않음
`Patch size`가 커진다면 더 넓은 범위의 이미지를 한번에 인식하다보니 `context` 인식에는 탁월하지만, `localization    에서 패널티를 같게 됨
`Patch size`가 작아지면 반대의 효과를 가짐

> U-Net은 여러 layer의 output을 동시에 검증하면서 `localization`과 `context` 인식 두 가지 토끼를 다 잡을 수 있게 됨

### 네트워크 구조
![](https://postfiles.pstatic.net/MjAxODA4MDZfMjkg/MDAxNTMzNTUxOTUxOTU0.YzYd-ho-1jFLlBmDWRTlnxRjjKlA2XX0wmutkUXARrcg.r_RiV19V9ocbF_9jM_D9kze0TdFf5oWKY7rnZHQYLIUg.PNG.worb1605/image.png?type=w773)

`FCN(Fully Connected Network)`가 없음. 속도 측면에서 많이 빨라짐

##### Mirror Padding
> contracting path에서 padding이 없었기 때문에 외곽부분이 잘려나간 것과 같게 되었고, 이를 해결하기 위해 `mirroring`이라는 방법을 사용

![](https://postfiles.pstatic.net/MjAxODA4MDZfMTk4/MDAxNTMzNTU0MzU2MDQx.qpLw2IrxBmA4cet6gP0YIj2CMJO5KRHSlqSgzbmSwQ8g.RcwcpV96vzLZp2B2OQuXDeN1wYHT-vF8SuUqmXbc6yAg.PNG.worb1605/image.png?type=w773)

> 바깥쪽 없어진 부분이 안쪽에 있던 이미지와 비교해 거울에 반사된 형태를 갖는다.



### Networkd Architecture
#### Contracting Path
* 3x3 convolution 사용
* 활성화 함수로 relu 사용
* Pooling 계층에서는 2x2 max pooling -> 매 계층이 내려갈때마다 1/2 Down sampling
* 계층이 내려갈땐 채널 개수를 2배씩 늘림

#### Expanding Path
* 2x2 up convolution 사용
* 같은 계층 안에서는 3x3 convolution 사용

#### concat
**회색선** : input이 output에 영향을 끼치도록 만든 선.
`mirror padding`을 진행할 때 손실되는 path를 살리기 위해서 `contracting path`의 데이터를 적당한 크기로  crop한 후 concat 하는 방식으로 이미지 보상처리

### Data Augmentation

> 데이터가 한정되어 있을 때 데이터를 회전, 반전과 같은 여러가지 효과를 주어 데이터를 키우는 방식
* training data가  한정되어 있을때 주로 사용하는 방식
  