# HOW TO USE DW1xxx SERIES ?

## FRAME STRUCTURE

<img width="752" height="245" alt="image" src="https://github.com/user-attachments/assets/6cf782ac-db92-46e5-8d8e-9489b61abf05" />

UWB 통신은 기본적으로 프레임(Frame) 단위 전송 및 수신을 기반으로 함.

- **Preamble**과 **SFD**(Start of frame delimiter)는 "동기화 헤더(Synchronisation Header)"를 구성
- **PHR or PHY**은 데이터 페이로드의 길이와 전송 속도(rate)
- 마지막으로 **Data**에는 실제 전송되는 정보가 포합되어 있음

## CIR (Channel Impulse Response)

> Channel impulse response (CIR) represents the propagation of a signal in response to the combined effects of scattering, fading, and power decay as it travels through a transmission medium, such as a wireless or acoustic channel.

채널 임펄스 응답은 신호가 전송되면서 산란, 페이딩, 전력 감쇠 등의 복합적인 영향을 받아 어떻게 전파되는지를 나타내는 특성.

디지털 통신에서 다중경로(multi-path)와 페이딩(fading) 효과를 이해하는 데 매우 중요한 역할을 수행함. <br/><br/>

전송 신호를 $S(t)$, 수신된 신호를 $R(t)$ 라고 하면 채널 임펄스 응답 $h(t)$와의 관계는 다음과 같다.
(이 떄, * 은 합성곱(Convolution) 연산을 의미함)

$R(t) = S(t) * h(t)$

즉, CIR은 전송 매체가 신호의 주파수 스펙트럼 형태(spectral shape)와 시간적 특성(temporal properties)을 어떻게 변화시키는지를 나타냄.

## DIAGNOSTICS

What can we obtaion from "**dwt_readaccdata()**" and "**dwt_readdiagnostics()**" function ??

<img width="1429" height="620" alt="image" src="https://github.com/user-attachments/assets/74e8dd03-b417-4e7c-9179-03b9875cfcb5" />

- **✅ dwt_readdiagnostics()**

  함수는, CIR 데이터를 분석하여 사용자에게 몇가지 필요한 정보를 제시함.

  ```
  Typedef struct
  {
    uint16 maxNoise ; // LDE max value of noise
    uint16 firstPathAmp1 ; // Amplitude at floor(index FP) + 1
    uint16 stdNoise ; // Standard deviation of noise
    uint16 firstPathAmp2 ; // Amplitude at floor(index FP) + 2
    uint16 firstPathAmp3 ; // Amplitude at floor(index FP) + 3
    uint16 maxGrowthCIR ; // Channel Impulse Response max growth CIR
    uint16 rxPreamCount; // count of preamble symbols accumulated
    uint16 firstPath ; // First path index
  }dwt_rxdiag_t ;
  ```

  각 파라미터를 하나씩 보면 아래와 같다.

  <details>
    <summary> 🔹maxNoise (작성중)</summary><br/>

    asdasd
  </details>
  
  <details>
    <summary> 🔹firstPath </summary><br/>

    First Path Index는 16비트 값으로, LDE 알고리즘이 누적기 내부에서 첫번째 경로(First Path)라고 판단한 위치를 나타냄.
  
    LDE 알고리즘이 CIR을 분석하는 동안 설정되며, LDEDONE 상태 비트가 Set 될 때 갱신된다.
  
    단위는 1ns의 sample time 을 곱하여, 공기 중 전파 이동거리로 약 30cm 정도를 구분지을 수 있지만,,
  
    더 정밀하게 표현하기 위해 인덱스 값은 정수부와 소수부로 구성할 수 있음
  
    - 상위 10비트(MSB 10bit) -> 정수 부분
    - 하위 6비트(LSB 6비트) -> 소수 부분
    <br/><br/>
      
    따라서, 이는 정수부(10비트) + 소수부(6비트)로 구성된 16비트 고정소수점 값으로 First Path의 위치를 표현함.
  
  </details> 

  <details>
  <summary> 🔹stdNoise </summary><br/>

    16비트 값으로, 누산기(accumulator) 데이터를 분석하는 동안 관찰된 노이즈 수준의 표준 편차를 나타냄.

    수신된 신호의 품질 및 LDE가 생성한 수신 타임스탬프의 신뢰도를 평가하는데 사용할 수 있음.

    <img width="752" height="118" alt="image" src="https://github.com/user-attachments/assets/de531793-773f-4b57-8c3c-f658e456d552" />

    <img width="752" height="452" alt="image" src="https://github.com/user-attachments/assets/96b4ad37-7711-41f9-b29a-d6526f01a645" />

  
</details> 

- **✅ dwt_readaccdata()**

  > **이거 4065 Byte 한번에 함수로 읽으려고 하면, 프로그램이 내부에서 멈춰버립니다.. 유의**

  함수를 통해 0x25 (Accumulator CIR Memory) 레지스터에 접근하여 누적된 CIR 데이터를 가져올 수 있음 ( 64MHz PRF의 경우 1016 샘플 ) 
  
  <img width="752" height="117" alt="image" src="https://github.com/user-attachments/assets/6008bffc-601b-4368-bc3f-6f1607f292d9" />

  CIR 각 샘플은 실수부와 허수부로 구성되어 있음. (a.k.a I/Q Signal -> [link](https://en.wikipedia.org/wiki/In-phase_and_quadrature_components))

  따라서 그래프로 plot 하기 위해서는 Amplitude 계산을 아래와 같이 수행해야함.

  $A = \sqrt{ (I^2 + Q^2) }$
  
  <img width="752" height="364" alt="image" src="https://github.com/user-attachments/assets/146c3adc-8ba4-4ce5-a586-914319651f5b" />  
  <br/><br/>
  <details>
    
    <summary>👍 Accurate TimeStamp Determination</summary><br/>
    
    수신 시간(RMARKER timestamp)를 결정하기 위해서 DW1000은 내부 LDE(Leading Edge Detection) 알고리즘을 사용함.

    이 LDE 알고리즘은 CIR 누적 메모리에서 신호의 첫 도착 경로인 Leading Edge를 찾으며 이는 0x15 (Receive Time Stamp) 레지스터에 기록됨.
  
    <img width="752" height="116" alt="image" src="https://github.com/user-attachments/assets/931d6bcd-1765-4678-98cb-99c1a311ebc8" />
  
    <img width="752" height="255" alt="image" src="https://github.com/user-attachments/assets/3bec1ea0-3002-41f9-8cc5-b751e35ed6c6" />

    ```
    uint8_t rx_tarray[5] = {};
        
    dwt_readrxtimestamp(rx_tarray);
  
    uint64_t rx_stamp = ((uint64_t)rx_tarray[4] << 32) |
                        ((uint64_t)rx_tarray[3] << 24) |
                        ((uint64_t)rx_tarray[2] << 16) |
                        ((uint64_t)rx_tarray[1] << 8)  |
                        ((uint64_t)rx_tarray[0]);
  
    double rx_sec = (double)rx_stamp / (128 * 499.2 * pow(10,6));
  
    printf("%f \r\n", rx_sec);
    ```

    코드는 위와같이 구성하여, rx_stamp를 확인할 수 있고 점차 증가하다가 약 17.2 sec 근방에서 계속 초기화되는것을 볼 수 있음.
      
    (이는, $2^40$ 을 rx_sec 에 나눠준 값으로 나눠주면 약 19.12 가 나와서, 최대 19.1 까지 표현할 수 있음을 의미하나 조금 차이는 있음)

  </details>
