# HOW TO USE DW1xxx SERIES ?

## 🟥 MESSAGE TRANSMISSION

#### ⏩ BASIC TRANSMISSION

송신 시퀀스는 아래 그림과 같다.

<img width="752" height="188" alt="image" src="https://github.com/user-attachments/assets/947b2c4e-c5d5-4f1c-b877-a8e501851173" />

송신(Transmit)을 위해, 호스트 컨트롤러가 송신할 데이터를 0x09 (Transmit Data Buffer) 레지스터에 기록해야 한다.

<img width="752" height="114" alt="image" src="https://github.com/user-attachments/assets/0b640ff1-6846-4ca2-9eb0-1fbab401737d" />

또한, 프리앰블(Preamble) 길이, 데이터 전송 속도(Data rate), PRF(Pulse Repetition Frequency) 등을 0x08 (Transmit Frame Control) 레지스터에 기록해야함.

<img width="752" height="120" alt="image" src="https://github.com/user-attachments/assets/50bfe177-d6bf-4267-8fb5-542cbd358de9" />

송신기 설정은 "IDLE" 상태에서 수행되지만, 프레임 설정(Frame Configuration)은 송신 중에도 수행할 수 있음.

(OFF, WAKEUP, INIT, IDLE 등의 Operational State에 대해서는 User Manual 15page 참고하면 된다.)

모든 관련 설정이 완료되었다는 가정 하에, 호스트는 0x0D (Ssyem Control Register) 레지스터에 "TXSTRT" 제어비트를 설정하여 송신을 시작.

<img width="752" height="440" alt="image" src="https://github.com/user-attachments/assets/e546e6e3-80e9-4b38-ac71-66d386105dad" />

송신이 요청되면, DW1000은 자동으로 전체 프레임을 전송하며 Preamble, SFD, PHR, Payload(Data) 를 순서대로 송신하게 됨.

프레임 전송의 완료는 0x0F (System Event Status Register) 레지스터의 TXFRS 이벤트 상태 비트를 통해 호스트에게 신호로 전달됨.

이 후, DW1000은 IDLE 모드로 돌아가 다음 명령을 기다리게 됨.

<img width="752" height="125" alt="image" src="https://github.com/user-attachments/assets/69623a59-ce75-491a-9984-86f77d52a357" />

------

#### ⏩ Transmission Timestamp

  안테나로 부터 PHR의 첫 번째 심볼이 전송되는 시점이 송신 타임스탬프가 찍히는 시점으로 **RMARKER** 라고 정의함.

  <img width="752" height="220" alt="image" src="https://github.com/user-attachments/assets/8cd103c0-a3cf-476b-a1d1-a5a1537628ed" />

  따라서, DW1000의 디지털 소인 회로는 PHR 전송을 시작하는 시점의 **시스템 클록 카운터값**을 RAW 송신 타임스탭프에 기록한다.

  이후, RAW 스탬프에 "송신 안테나 지연값(Transmit Antenna Delay)"을 보정한 송신 타임스탭프를 계산한다.

  RAW TIMESTAMP 보정에 사용되는 안테나 딜레이 값과 보정된 TIMESTAMP는 각각 0x18 (Transmitter Antenna Delay)과 0x17 (Transmit Time Stamp)에 기록된다.
  
  <img width="752" height="119" alt="image" src="https://github.com/user-attachments/assets/efbf9457-0b63-43d3-b3ad-cb3fba5c2348" />

  <img width="752" height="119" alt="image" src="https://github.com/user-attachments/assets/e62febbf-6c25-4ce0-b07a-0c5db379b3ef" />

## 🟥 MESSAGE RECEPTION

#### ⏩ BASIC RECEPTION

프레임 수신은 호스트의 요청 or 수신기의 자동 재활성화(automatic re-enabling)에 의해 활성화 된다.

수신기기가 활성화 되면 프리앰블이 탐지될때 까지 지속적으로 프리앰블을 탐색하고,

이러한 탐색과정을 일정 시간 후 중단할 수 있도록 하는 프리앰블 탐지 타임아웃(Preamble Detection Timeout)을 설정할 수 있따.

전반적인 수신 과정은 아래와 같다.

<img width="752" height="124" alt="image" src="https://github.com/user-attachments/assets/582ce736-bb6d-4f18-95d7-2bc34ca6d295" />

**1. Preamble Detection**

프리앰블 시퀀스는 여러 개의 프리앰블 심볼로 이루어진 Chunk 단위로 상호 상관(Cross-Correlation)을 수행하여 검출된다.

이 때, 사용되는 청크의 크기는 0x27:08 (DRX_TUNE2) 레지스터의 PAC(Preamble Accumulation Count) 설정값에 의해 결정된다.

<img width="752" height="111" alt="image" src="https://github.com/user-attachments/assets/79e37338-66f3-45e8-85d6-d2f97616fc2d" />

PAC의 크기는 예상되는 프리앰블 길이에 따라 선택되어야 하며, 이상한 값을 쓰면 DW1000의 오작동을 유발할 수 있다.

- 큰 PAC 크기는 충분히 긴 프리앰블 또한 클 때 더 나은 성능을 제공함.
- 하지만, 프리앰블 길이에 비해 PAC가 너무 크면, 수신 성능이 저하되거나 극단적인 경우 수신이 실패할 수 있음.

  (예를 들면, 64-symbol 프리앰블에서 PAC=64 인 경우, 프레임은 절대 수신되지 않음)

권장되는 설정값은 아래와 같으며, 프리앰블 길이 선택은 USER MANUAL 210page에 나타나 있음.

<img width="400" height="369" alt="image" src="https://github.com/user-attachments/assets/24c6747e-f1e6-4334-91d2-731f9001de01" />

수신 중, 유효한 프리앰블이 설정된 시간 내에 검출되지 않으면 수신을 중단(Abort)할 수 있다.

- 이는 프리앰블 검출 타임아웃(Preamble Detection Timeout)을 이용하여 수행함
- 0x27:24 (DRX_PRETOC) 레지스터를 통해 Cofiguration이 가능함.

이러한 기능은 특히 메시지 송신 후 응답을 기다리는 상황에서 유용하게 동작함

- 응답 프리앰블이 검춛되지 않으면, 기다리던 응답이 오지 않는것으로 판단함.
- 가능한 빠른 시점에 수신을 포기(Abandon)하여 전력 소비를 줄일 수 있음

또한 프리앰블 검출 상태는 Pulse Preamble Detection Mode (PPDM) 또는 SNIFF 모드로 동작할 수 있다.

- 이 모드는 0x1D (SNIFF Mode) 레지스터를 통해 설정 가능함
- 수신기가 일정 시간 간격으로 Air를 주기적으로 샘플링(Sniffing)하여 일부 시간만 활성화함으로써 프리앰블 검출 시 필요한 전력을 줄이는 기법임.

**2. Preamble Accumulation**

프리앰블 시퀀스가 검출되며느 수신기는 **상호 상관(Cross-Correlated)**된 프리앰블 심볼을 누적(Accumulation)하면서 동시에 SFD 시퀀스를 찾기 시작함.

(SFD 시퀀스에 대한 내용은 아래에서 더 자세하게 다룸)

이 누적(Accumulation)은 SFD가 검출되면 종료됨.

- 그러나 아주 가까운 거리의 LOS 환경에서는 누적이 매우 빠르게 진행되어 SFD가 검출되기도 이전에 조기 종료되기도 함
- 이 경우, 수신기는 여전히 프리앰블은 수신하지만 누적은 하지 않고 계속하여 SFD 시퀀스를 탐색함

**3. SFD Detection**

SFD 검출은, 프레임 수신에서 가장 핵심적인 이벤트.

> PHY 헤더의 시작을 나타내기도 하며 이 위치는 RMARKER가 정의되고 타임스탬프가 기록되는 시점임

SFD 검출 시점부터 프리앰블 복조가 아닌, PHR  및 이후 데이터의 BPM/BPSK 복조로 전환함 (Demodulation)

SFD가 일정 시간 내에 검출되지 않으면 수신을 중단할 수 있으며, 0x27:20 (DRX_SFDTOC) 에서 Timeout을 설정 가능함.

(프리앰블이 잘못 검출된 경우, 불필요하게 긴 시간 동안 수신 상태에 머무는 것을 방지하기 위함)

SFD의 길이(8심볼 or 64심볼) 및 수신기 설정은 0x04 (System Configuration) 레지스터를 통해 설정 가능

위에서 말한 "표준 SFD" 이외에도, "비표준 SFD" 시퀀스를 프로그래밍하여 성능을 향상시킬 수 있음.

(이는 0x21 (User Defined SFD sequence) 레지스터를 통해 설정할 수 있음)

**4. PHR Demodulation**

PHY 헤더(PHR)은 데이터 부분의 길이와 속도를 알려주는 역할을 함.

850 kbps 및 6.8 Mbps 데이터 속도의 경우, PHR은 850kbps 방식으로 변조/복조 (Modulate/Demodulate) 됨.

PHR의 속도에 맞게 데이터도 복조됨.

(만약 PHR이 850 kbps를 나타내면, 이후 데이터도 850 bps로 복조됨)

**5. Data Demodulation**

데이터 수신 및 복조과정은 간략히 아래와 같음

> 데이터 복조 -> 오류 정정 -> 프레임 필터링(선택) -> 수신이벤트 알림

- 수신기에서는 Viterbi 디코더를 사용하여 데이터 비트를 복원함 (이는 PHR 수신에도 동일하게 적용)
- 복원된 데이터는 Read-Solomon 디코더를 통과하여 가능한 추가 오류를 교정. (각 옥텟(Octet)은 CRC 검사를 통과하며, 프레임의 FCS와 비교)
- 수신된 데이터 옥텟은 프레임 필터링 기능이 활성화 된 경우 필터링 됨 (USER MANUAL 47page)
- 최종적으로 0x0F (System Event Status Register) 내의 RXDFR 및 RXFCG 이벤트 비트로 호스트에 신호 전달

**6. RX Message Timestamp**

프레임 수신 과정에서, SFD 검출 이벤트는 프리앰블 종료 및 PHR 시작을 표시하고 이 시점이 IC에서 타임스탬프가 찍히는 기준점이 됨.

(IEEE 802.15.4 UWB 표준에서는 RMARKER가 안테나에 도착하는 순간을 중요한 이벤트로 지정하고, 이 시점을 타임스탬프 기준으로 함)

- RMARKER 이벤트가 발생한 심볼에 대해 디지털 수신 회로가 Timestamp 기록
- 수신 안테나 지연(LDE_RXANTD)를 "빼고", Leading Edge 검출 알고리즘에 의해 결정된 보정값을 "추가"
- 최종적으로 보정된 RX 타임 스탬프를 기록 (RMARKER가 안테나에 도착한 실제 시간을 말함 0x15 (Receive Time Stamp))

만약, 특정 수신 오류/타임아웃 이벤트가 발생시 반드시 수신기의 리셋이 필요함

> PXPHE (PHY Header Error), RXRFSL (Read-Solomon Error), RXRFTO (Frame Wait Timeout)

이는 다음 정상 프레임의 타임스탬프가 정확하게 계산되도록 보장하기 위함이며 위 경우를 제외하고는 리셋 필요 없음

이러한 리셋은 0x36:00 (PMSC_CTRL0 의 SOFTRESET) 레지스터 필드를 통해 설정할 수 있다.

## 🟥 BASIC FOR UNDERSTANDING
 
#### ⭐ PRF (Pulse Repetition Frequency)

<img width="330" height="185" alt="image" src="https://github.com/user-attachments/assets/d429f17d-e0a3-4c7e-a764-2293128dfac3" />

쉽게, 단위 시간동안 반복되는 신호의 펄스 수를 나타낸다.

DW1000의 경우 선택할 수 있지만, 64MHz를 예로 1초에 64,000,000 개의 펄스가 송신된다고 볼 수 있다.

16MHz와 비교하여 펄스 간 간격이 더 짧기 때문에 시간 분해능이 더 높다 = 더 조밀한 시간 단위로 수신 신호 누적 가능

## 🟥 FRAME STRUCTURE

<img width="752" height="245" alt="image" src="https://github.com/user-attachments/assets/6cf782ac-db92-46e5-8d8e-9489b61abf05" />

<img width="752" height="393" alt="image" src="https://github.com/user-attachments/assets/c2bc4449-6bb2-4f5e-93a8-72b758ea79b3" />

UWB 통신은 기본적으로 프레임(Frame) 단위 전송 및 수신을 기반으로 함.

- **Preamble**과 **SFD**(Start of frame delimiter)는 "동기화 헤더(SHR : Synchronisation Header)"를 구성
- **PHR or PHY**은 데이터 페이로드의 길이와 전송 속도(rate)
- 마지막으로 **Data**에는 실제 전송되는 정보가 포합되어 있음

<img width="725" height="315" alt="image" src="https://github.com/user-attachments/assets/e831874e-109f-471e-9101-60fbed88d2df" />

대충 그려봤는데, 이런 느낌으로 Preamble이 구성되는 것 같다. 이는 안테나 설정에 따라 길이나 duration 등이 달라질 수 있음.

<details>
  <summary> 🔹Preamble (SYNC Field) </summary><br/>  

  프리앰블은, 통신 시스템에서 데이터 프레임이나 패킷의 시작 부분의 특별한 비트 패턴이나 신호 시퀀스를 의미함.

  목적은, 수신기가 수신 신호의 동기화를 수행하고 데이터를 올바르게 해석할 수 있도록 준비시키는 데 있음.   

  DW1000의 경우, 여러 설정값이 있지만 본 분석에서는 128을 활용하였음.

  내부에 어떤 패턴들로 Preamble을 구성하고 있는지는 알 수 없음.

  이 수신기가 Preamble을 감지하면, 해당 프리앰블 심볼들을 상관(Correlation)시켜 누적하기 시작함.

  (이는, SFD를 찾는기 전까지 계속 수행됨)

  다만, 신호가 매우 강한 경우 (LOS, 근거리 환경), 이 상관 값이 매우 빠르게 누적되므로 SFD를 찾기도 전에 멈출 수 있음.

  더이상 누적은 하지 않지만, 여전히 SFD는 탐색함.

  ------

  PHR과 Payload 구간에서 사용되는 BPM/BPSK 변조와 달리, Preamble + SFD (SHR) 부분은 단일 펄스(Single Pulse)로 구성된다.

  <img width="752" height="473" alt="image" src="https://github.com/user-attachments/assets/cf583835-c851-4f73-99bf-b247e7afce71" />

  하나의 심볼(symbol)은 약 500개의 "칩(chip)" 시간 간격으로 나뉘며, 64MHz 의 경우 508개의 칩으로 구성되어 있다.

  각 칩 구간에는 양(+)의 펄스, 혹은 음(-)의 펄스, 그리고 펄스 없음 중 하나가 전송될 수 있다. 

  - 칩 간격(chip interval)은 499.2 MHz = 2ns 이며, 이는 UWB PHY의 기본 주파수임.
  - 심볼 길이는 다음과 같음. (508개의 chip으로 이루어진 한 심볼의 시간)
    - 16MHz PRF : 496 / 499.2 us = $0.9 us = 993.59 ns$
    - 64MHz PRF : 508 / 499.2 us = $1.01 us = 1017.63 ns$
      (쉽게 2ns의 Chip이 508개 있으므로 하나의 심볼에는 1017.63 ns 가 소요된다.)
   
  <img width="752" height="168" alt="image" src="https://github.com/user-attachments/assets/94285065-45f8-40f9-9050-3310b3d63063" />

  심볼 간 실제로 전송되는 펄스 시퀀스는 프리앰블 코드(Preamble Code)에 의해 결정된다. 

  <img width="752" height="340" alt="image" src="https://github.com/user-attachments/assets/5b0aecb4-3cd6-43e0-877f-345cef129a35" />

  64MHz PRF 에서는 길이가 127인 코드(length-127 preamble code) 16개를 정의한다.

  표준은 특정 채널마다 사용할 코드를 지정하며 64MHz PRF 에서는 채널 당 4개 코드중 선택 가능하다.

  코드의 확산(Spread) 방식은 아래와 같다.
    - 길이 31 코드 : 각 코드 포인트마다 15개의 0을 삽임 -> 심볼당 496칩
    - 길이 127 코드 : 각 코드 포인트마다 3개의 0을 삽임 -> 심볼당 508칩 ( 127 + 127*3 )

  프리앰블 길이(Preamble length)는 시퀀스가 몇 번 반복되는지, 즉 몇 개 심볼을 반복하는지로 정의된다.

  이는 PSR (Preamble Symbol Repetitions) 설정값을 16, 64, 1024, 4096 으로 정의되지만

  추가로 128, 256, 512, 1536, 2048 설정도 지원한다. ( 64 심볼 미만의 프레임은 수신하지 못함 )

  ------

  이 프리앰블 시퀀스는 완벽한 주기적 자기상관 (Perfect Periodic Autocorrelation) 특성을 가짐

  즉, Coherent Receiver가 송신기와 수신기 사이 RF 채널에 대한 정확한 Impulse Response를 결정할 수 있게함.

  - 자기 상관 (Auto Correlation)

    상관(Correlation)은 "수신 신호"와 "프리앰블 패턴"을 비교하는 과정을 말함.

    자기 상관은 자기 자신의 신호 시간 지연을 두고 곱해서 합산한 값을 말함.

    $C[k] = \sum_{n} p[n+k] \cdot p[n]$

    (시간 지연이 없을 때, 자기 신호와의 자기 상관은 가장 최대값이 되며 이를 "완벽한 자기상관" 이라고 함)

    수신기에서의 시간 지연이 존재하더라도, 자기상관 계산으로 정확히 심볼 시작 위치를 찾을 수 있음
  
    수신 신호 $r[n]$이 들어오면, 미리 알고 있는 프리앰블 $p[n]$과 같은 위치에 있는 비트/샘플끼리 곱하고 합산함.
  
    $C[k] = \sum_{n} r[n+k] \cdot p[n]$
  
    이렇게 되면, 수신 신호에서 프리앰블과 잘 맞는 위치에서 상관값이 크게 나타남.
  
    다만, 한번만 누적하면 잡음으로 인해 신호가 잘 안보일 수 있어, 여러 프리앰블 신호를 차례로 쌓아 누적함.
  
    (반복되는 프리앰블 신호들을 모두 더해, 신호 대 잡음비(SNR)을 높임)

</details>

<details>
  <summary> 🔹SFD (Start of Frame Delimeter) </summary><br/>

  데이터 프레임이 본격적으로 시작됨을 알려주는 시그널 패턴.

  <img width="752" height="464" alt="image" src="https://github.com/user-attachments/assets/08af03aa-fbc9-4ee7-8932-3a5dfdc64c2f" />

  이 SFD가 검출되는 순간이 PHY 헤더의 시작(RMARKER의 시작점)이자 **RX_TIMSTAMP**가 기록되는 순간이므로 매우 중요함.

  (즉 , 실제 신호가 도착한 기준 시점이 됨)  

  ------

  **FROM USER MANUAL 216 page**

  - SFD 

      프리앰블의 마지막 및 PHY 데이터의 시작을 알려줌.

      이 SFD는 프레임 동기화 및 Ranging 카운터 관리 역할을 수행함.

      <img width="752" height="303" alt="image" src="https://github.com/user-attachments/assets/f0a45b7f-7331-4e64-93bd-55d17e9d9776" />

      SFD의 길이와 시퀀스는 데이터율에 따라 달라짐

      > 110kbps의 경우 64 심볼, 그리고 시퀀스는 8개를 예시로 [0 +1 0 -1 +1 0 0 -1] 이런식임.
  
</details>

<details>
  <summary> 🔹PHY or PHR (Physical Layer Header) </summary><br/>

  SFD 다음에 전송되며 데이터율과 프레임 길이 정보를 포함하고 있음.

  <img width="752" height="200" alt="image" src="https://github.com/user-attachments/assets/4b6e5c17-6a47-45d2-8aa3-af2e1d43f8ca" />
  
</details>

<details>
  <summary> 🔹Payload or Data </summary><br/>  

  통신 데이터를 전달하는 부분임. IEEE 802.15.4 에서 사용되는 UWB는 고속 RF(무선 주파수) 펄스를

  기반으로 하기 때문에 종종 임펄스 라디오 UWB (IR-UWB : Impulse Radio UWB) 라고 불림.

  <img width="752" height="390" alt="image" src="https://github.com/user-attachments/assets/9e3675ca-ad9f-4945-809f-7824956913f4" />

  프레임의 PHR 및 데이터 구간은, 정보를 담는 비트들이 버스트(Burst)위치에 의해 표현되며 이러한 변조방식을 

  **버스트 위치 변조(BPM: Burst Position Modulation)** 라고 함.

  각 데이터 비트는 컨볼루션 인코더(Convolution Encoder)를 통과하며, 이 인코더는 패리티 비트(Parity Bit)를 생성함.

  이 패리티 비트는 버스트의 위상(Phase)를 양(+) 또는 음(-)으로 설정하는데 사용하며, 이러한 위상 변조 방식을

  **이진 이상 편이 변조(BPSK : Binary Phase-Shift Keyring)** 라고 부른다. 

</details>

## 🟥 CIR (Channel Impulse Response)

> Channel impulse response (CIR) represents the propagation of a signal in response to the combined effects of scattering, fading, and power decay as it travels through a transmission medium, such as a wireless or acoustic channel.

채널 임펄스 응답은 신호가 전송되면서 산란, 페이딩, 전력 감쇠 등의 복합적인 영향을 받아 어떻게 전파되는지를 나타내는 특성.

디지털 통신에서 다중경로(multi-path)와 페이딩(fading) 효과를 이해하는 데 매우 중요한 역할을 수행함. <br/><br/>

전송 신호를 $S(t)$, 수신된 신호를 $R(t)$ 라고 하면 채널 임펄스 응답 $h(t)$와의 관계는 다음과 같다.
(이 떄, * 은 합성곱(Convolution) 연산을 의미함)

$R(t) = S(t) * h(t)$

즉, CIR은 전송 매체가 신호의 주파수 스펙트럼 형태(spectral shape)와 시간적 특성(temporal properties)을 어떻게 변화시키는지를 나타냄.

------

그 CIR 인덱스는 정말로 의미가 없을까?

> FP_INDEX는 항상 750 근방에 잡히기 때문에, 이걸 그대로 쓰는 건 의미 없다.

그러면 CIR로 뭘 알 수 있을지 한번 생각해보자.

DW1000은 내부 자체 알고리즘에서 FP_INDEX를 Direct Path로 판단하여 거리 계산에 사용한다. 

그러면 FP_INDEX(예 : 750)를 기준 0s 로 잡고, 만약 760에 Peak가 하나 존재한다면 1ns * (760-750) = 10ns 만큼 지연되어 신호가 들어왔음을 말함.

따라서, 기존 TOF 값에 보정하여 Multipath 길이를 계산할 수 있을 뿐만 아니라, 실제 길이를 안다면 해당 신호가 NLOS 인지 판단 또한 할 수 있다는 점에서 의의를 가짐.

## 🟥 DIAGNOSTICS

What can we obtaion from "**dwt_readaccdata()**" and "**dwt_readdiagnostics()**" function ??

<img width="1429" height="620" alt="image" src="https://github.com/user-attachments/assets/74e8dd03-b417-4e7c-9179-03b9875cfcb5" />

- **✅ dwt_readdiagnostics()**

  함수는, CIR 데이터를 분석하여 사용자에게 몇가지 필요한 정보를 제시함.

  ```
  Typedef struct
  {
    uint16 maxNoise ; // LDE max value of noise
    uint16 firstPathAmp1 ; // Amplitude at floor(index FP) + 3
    uint16 stdNoise ; // Standard deviation of noise
    uint16 firstPathAmp2 ; // Amplitude at floor(index FP) + 2
    uint16 firstPathAmp3 ; // Amplitude at floor(index FP) + 1
    uint16 maxGrowthCIR ; // Channel Impulse Response max growth CIR
    uint16 rxPreamCount; // count of preamble symbols accumulated
    uint16 firstPath ; // First path index
  }dwt_rxdiag_t ;
  ```

  각 파라미터를 하나씩 보면 아래와 같다.

  <details>
    <summary> 🔹maxNoise </summary><br/>

    LDE 알고리즘이 누산기(Accumulator) 데이터를 분석하는 동안 관찰된 가장 큰 노이즈 수준을 말함.
  
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

    <img width="752" height="240" alt="image" src="https://github.com/user-attachments/assets/b91b6900-f22b-4369-b5b2-4162c3f06a59" />

    일반적으로, **RX_TIMESTAMP**는 첫 번째 신호가 내부 디지털 회로에서 검춛된 시점을 나타내며, 이는 LDE 검출 시점을 기준으로 기록된다.

    헷갈리면 안되는게, **FP_INDEX**는 실제로 DW1000에서 LDE가 예상 First Path 위치를 3/4 정도 지점으로 초기 추정을 두기 때문에, (왜인지는 Qorvo가 암)

    대부분 740~750 주변에서 나오지만 이는 절대값이 아닌 상대적 위치임. 따라서 누적기 샘플 0부터 센 위치에 대한 절대 시간을 의미하지않음.

    ----

    이 x축에 Index로 나타나는 값을 보자.

    "Zero Index"은 DW1000이 Preamble이 감지되는 순간이며, 내부적으로 accumulator의 0번 인덱스를 잡음.

    즉, PRF와 SFD에 따라, 누적을 시작한 시점이 0이 되고, CIR 버퍼(1016샘플) 증에서 약 750 근처에 FP가 들어오도록 설계되어 있음.

    그리고 Accumulator 1심볼 전체는 약 1ns 단위로 샘플링 되며, 기준은 **보정된 RX_STAMP**를 사용함.

    RX_STAMP = RX_RAWST + LDE 보정값
  
  </details> 

  <details>
    <summary> 🔹stdNoise </summary><br/>

    16비트 값으로, 누산기(accumulator) 데이터를 분석하는 동안 관찰된 노이즈 수준의 표준 편차를 나타냄.

    수신된 신호의 품질 및 LDE가 생성한 수신 타임스탬프의 신뢰도를 평가하는데 사용할 수 있음.

    <img width="752" height="118" alt="image" src="https://github.com/user-attachments/assets/de531793-773f-4b57-8c3c-f658e456d552" />

    <img width="752" height="452" alt="image" src="https://github.com/user-attachments/assets/96b4ad37-7711-41f9-b29a-d6526f01a645" />
  
  </details> 

  <details>
    <summary> 🔹firstPathAmp1,2,3 </summary><br/>

    실제 코드에서는 아래처럼 주석처리 되어있다.

    > firstPathAmp1 // Amplitude at floor(index FP) + 1

    하지만, 함수가 0x15 레지스터에서 읽어오는 FP_AMPL1 부분을 Manual에서 가져와 보면 아래와 같이 설명되어 있다.

    <img width="752" height="260" alt="image" src="https://github.com/user-attachments/assets/3cddb149-121b-4268-859e-29bc08f2aa3e" />

    즉, FP_INDEX 기준으로 3번째 인덱스 이후의 Rising Edge에 대한 Amplitude를 말하고 있으므로 차이가 있다.

    따라서, 쉽게 이해하기 위해 정리하면 다음과 같다.

    ```
    uint16 firstPathAmp1 ; // Amplitude at floor(index FP) + 3
    uint16 firstPathAmp2 ; // Amplitude at floor(index FP) + 2
    uint16 firstPathAmp3 ; // Amplitude at floor(index FP) + 1
    ```

    16비트 부호없는 값으로, LDE 알고리즘이 누적기 데이터 메모리를 분석할 때, 감지된 Leading Edge 신호 세기를 말함.

    이 값은 LDEDONE 상태 비트가 설정되었을 때, 즉 LDE 실행이 완료된 후 갱신된다.
  
  </details> 

  <details>
    <summary> 🔹maxGrowthCIR </summary><br/>

    0x12 (Rx Frme Quality Information) 레지스터의 CIR_PWR 필드에서 받아오는 값으로 설명은 아래와 같음.

    16비트 값으로, 수신 신호의 채널 중에서 가장 강한 전력이 존재하는 구간으로 추정된 부분이 있고,

    해당 부분의 누산기 값 크기(Magnitude)를 제곱한 후 모두 더한 합(sum of squares)을 나타냄.

    <img width="752" height="103" alt="image" src="https://github.com/user-attachments/assets/2457e82f-5b9a-446e-bf15-9752a7f8dcd1" />

  </details>

  <details>
    <summary> 🔹rxPreamCount </summary><br/>

    수신기가 누적한 프리앰블(Preamble) 심볼의 갯수를 의미함.

    쉽게 수신된 신호에서 얼마나 많은 프리앰블 심불을 누적했는지를 말하며 0x10 (RX Frame Infomation) 레지스터의 RXPACC 필드에 존재

    <img width="752" height="463" alt="image" src="https://github.com/user-attachments/assets/7a953721-ed45-4750-997e-dcb1a81682a5" />

    이 값은, 전송된 프리앰블 (안테나 설정시 128) 길이보다 약간 크게 나올 수 있음.
  
    - 프리앰블을 조기검출 한 경우
    - 누적이 SFD(Symbol Frame Delimiter)까지 된 경우
    <br/>

    예를 들어, 128 Preamble을 송신하고 118 Preamble이 누적되었다면 거의 전체를 수신했다고 볼 수 있음.

    일부는 신호가 약해서 심볼이 충분히 누적되지 않았을 가능성으로 인해 누락되었음.

    누적심볼이 충분히 많을수록 SNR이 개선되고 LDE에서 First Path를 "정확히" 잡을 가능성이 높음.

    뿐만 아니라, 수신 타임스탬프의 정확도가 높아짐.

    ------

    RXPACC 카운터는, 수신기에서 프리앰블을 찾으면 포화(Saturate)되며, CIRE(CIR Estimator)는 더이상 심볼을 누적하지 않음.

    RXPACC와 RXPACC_NOSAT (From 0x27:2C)를 비교하여 RXPACC 값 조정이 필요한지 여부를 판단할 수 있음

    ( RXPACC_NOSAT 은 포화되지 않는 디버그용 심볼 카운터임)

    자세한 내용은, USER MANUAL 96~97p 에 설명되어 있음.

  </details>
  
- **✅ dwt_readaccdata()**

  > **이거 4065 Byte 한번에 함수로 읽으려고 하면, 프로그램이 내부에서 멈춰버립니다.. 유의**

  함수를 통해 0x25 (Accumulator CIR Memory) 레지스터에 접근하여 누적된 CIR 데이터를 가져올 수 있음 ( 64MHz PRF의 경우 1016 샘플 ) 
  
  <img width="752" height="117" alt="image" src="https://github.com/user-attachments/assets/6008bffc-601b-4368-bc3f-6f1607f292d9" />

  CIR 각 샘플은 실수부와 허수부로 구성되어 있음. (a.k.a I/Q Signal -> [link](https://en.wikipedia.org/wiki/In-phase_and_quadrature_components))

  따라서 그래프로 plot 하기 위해서는 Amplitude 계산을 아래와 같이 수행해야함. (**단위가 없는점 주의**)

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
      uint32 lo_tx = dwt_readtxtimestamplo32();
      uint32 hi_tx = dwt_readtxtimestamphi32() && 0x000000FF;

      uint64 tx_stamp = (uint64)hi_tx << 32 | (uint64)lo_tx;

      double tx_sec = tx_stamp / div_val;

      // float div_val = 128.0 * 499.2 * pow(10.0,6.0);  
      printf("[TAG-RECEIVE] %llu, %f \r\n", tx_stamp, tx_sec);
    ```

    코드는 위와같이 구성하여, rx_stamp를 확인할 수 있고 점차 증가하다가 약 17.2 sec 근방에서 계속 초기화되는것을 볼 수 있음.
      
    (이는, $2^40$ 을 rx_sec 에 나눠준 값으로 나눠주면 약 19.12 가 나와서, 최대 19.1 까지 표현할 수 있음을 의미하나 조금 차이는 있음)

  </details>
