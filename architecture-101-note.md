# Architecture 101

아키텍처 101을 보면서 알아낸 내용을 여기에 정리하자

1. Sequencer

Architecture

general vs special

deep learning inferencing accelerator!

deep neural network?

<img width="1281" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/110d3aef-c67a-4dab-885a-319eff8ad9b3">

deep neural network weighted sum 과 activation function을 가속화한다

dot product + non linear function

모든 것을 커버하려고 general하게 만드려고 한다면 어려울 것이다.

<img width="1278" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/21d94091-215d-4a6f-90c7-9f8a33050f19">

이 레이어들이 tensor라는 단위로 이루어져 있고 이것들로 연결되어있다. tensor 단위의 regular한 access로 이루어진다.

두 가지 중요한 축

1. tensor로 이루어져 있다.
2. dot product + non linear function

<img width="1279" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/de2b4d4d-94f5-463e-b6c3-3f39943e8f48">

tensor를 3차원이라고 가정하면 Channel, Height, Width로 나타내진다. 하지만 메모리는 1차원으로 구성된다. 그래서 tensor를 메모리에 나타내려면 차원을 정렬하는 순서에 따라서 레이아웃이 달라진다. 

<img width="1280" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/f97ac191-95c6-416c-b8c4-fd4fbbb212bf">

위와 같이 tensor가 메모리 상에 레이아웃 되어 있을 때 루프를 통해서 각 메모리에 접근할 수 있다. 만약 이러한 루프를 돌 수 있는 장치가 있다고 가정하자. 그리고 그 장치는 다음과 같이 작동할 것이다. 각 dimension을 돌면서 count를 증가시키되 limit보다 크면 다시 0으로 돌아간다. 그리고 count를 증가 시킬때 마다 stride 만큼 메모리 주소를 이동한다. 장치에 limit와 stride를 잘 넣어줄 수 있다면 tensor의 값들을 내가 원하는 순서대로 traverse할 수 있다. 예를 들어 CHW 순서로 접근하고 싶다면 위쪽의 표대로 접근할 수 있고 HCW 순서대로 접근하고 싶다면 아래 표대로 세팅하여 가능하다.

<img width="1279" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/d1df3fa1-c43a-42a4-a8c2-6ff120bae775">

pointwise convolution을 예시로 들어보자. pointwise convolution 연산이란 해당 tensor의 값에 접근해서 각각 weight 값을 곱한 뒤에 해당하는 output 값에 accumulation하는 연산이다. 위의 예시를 생각하면 tensor는 3x4x2 크기이고 weight 는 2x2 이다.

만약 Fetch, MAC, Commit을 할 수 있는 장치가 있다고 가정하자. 이 장치는 값을 하나 불러와서 (Fetch) weight를 곱하고 (weight를 불러오는 부분은 나중에..) 그 값을 Acc.에 속해서 쌓는 과정 (MAC) 그리고 완성된 값을 메모리에 저장하는 과정(Commit)을 수행할 수 있다. 사실상 Dot product를 하는 장치이다.

실제로 그림에서는 out_h, out_w, out_c, in_c 의 순서로 값을 읽는다. 특이한 점은 Fetch sequencer 에는 out_c에는 stride가 0이다. 이는 out_c가 증가해도 메모리 주소를 옮기지 않고 같은 값을 fetch하려는 의도이다. 

기본적으로 차원의 순서는 HWC로 되어있다.

정리하자면 tensor가 메모리 상에 저장되어 있고 pointwise convolution을 수행한다면 Fetch Sequencer는 저 MAC에 계산하고자하는 메모리 값을 원하는 순서대로 넣어주고 MAC는 Fetch된 값을 하나씩 계산해서 output 값을 만들고 Commit sequencer에서는 순서대로 계산된 값을 원하는 메모리 위치에 저장하는 역할을 한다.

<img width="1277" alt="image" src="https://github.com/SidongYang/architecture-study/assets/142791878/5eb0ab6a-9c92-417a-900d-9e8433ae4407">

실제 SRAM은 하나씩 작은 단위로 접근하는 것에 적합하지 않다.
