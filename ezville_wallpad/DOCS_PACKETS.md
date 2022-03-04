# 삼성SDS 월패드 RS485 패킷 분석

### 차례

  * [인터폰: A1~A6](#인터폰-A1A6)
  * [가스밸브: AB](#가스밸브-AB)
  * [조명: AC](#조명-AC)
  * [현관스위치: AD](#현관스위치-AD)
  * [난방: AE](#난방-AE)
  * [환기: C2](#환기-C2)
  * [대기전력차단 플러그: C6](#대기전력차단-플러그-C6)
  * [실시간 에너지사용량: AA](#실시간-에너지사용량-AA)
  * [현관스위치 신형: CC](#현관스위치-신형-CC)
  * ???: CE
  * ???: C1
  * ???: C3

### 약어

#### `PT`: parity(checksum)
* 패킷의 모든 Byte를 XOR한 후에 최상위 bit를 0으로 설정한 값입니다. 비트 오류가 있을 때 패킷을 무시하기 위한 방법입니다.

#### `GR`: Group ID
* 같은 종류의 장치를 구분하는 임의의 번호입니다. 대부분의 장치는 1부터 시작하는 연속 번호입니다.

#### `TG`: Toggle
* 1: 켜짐, 0: 꺼짐인 경우 `TG` 로 표기하고 설명 생략합니다.

### 공통 사항

#### B0 ~: 장치 응답
* 모든 장치의 응답은 항상 B0 으로 시작합니다.

=> Ezville은 F7 + "장치 분류" + "그룹/서브그룹" + "요청 or 응답" + 데이터 길이 + 데이터 + XOR + ADD 로 구성


#### `XX` 5A 00 `PT`: 장치 스캔
* 정전 등으로 월패드가 재시작하거나, 수 초간 장치가 제대로 된 응답을 하지 못하면 월패드는 해당 장치에 대해 스캔 모드로 전환합니다.
* `XX` 5A 00 `PT` 에 대해 해당 장치가 B0 5A ~ 로 응답해야 이후 정보를 주고받기 시작합니다.
* 집에 어떤 장치가 있는지는 따로 설정되어 있습니다 (방법 불명). 예외도 있지만, 기본적으로 집에 없는 장치는 아예 스캔을 하지 않습니다.

=> Ezville은 월패드 시작시 특성 요구를 보내고 특성 응답을 받음. (표준 기준)
   장치별로 표준에 맞는 특성 요구 명령 바이트와 특성 회신 바이트를 주고 받음.

#### `XX` 41 00 `PT`: Keepalive
* 월패드와 대부분의 장치들은 전달할 정보가 아무것도 없을 때, 단순히 `XX` 41 00 `PT` 을 전송합니다.
* 이 때 장치인 경우 B0 으로 시작하므로 B0 41 00 71 이 됩니다.
* 5Byte로 통신하는 신형 기기들은 `XX` 41 01 00 `PT` (B0 41 01 00 70) 을 사용하는 것으로 보입니다.

=> Ezville은 월패드에서 주기적으로 상태 요구를 보내고 상태 응답을 받음. (표준 기준)
   장치별로 표준에 맞는 상태 요구 명령 바이트와 특성 회신 바이트를 주고 받음.

#### 월패드 ACK
* 장치에서 발생한 이벤트에 월패드가 다시 ACK를 주는 경우, 즉시 주지 않고 다음번에 그 장치 차례가 되었을 때 전송합니다.
* 이벤트 종류에 따라 장치의 ACK를 받을 때까지 반복하거나, 한번 전송하고 끝나는 경우도 있습니다.

=> 

## 인터폰: A1~A6

| 월패드 | 장치 응답 | 월패드 ACK | 장치 ACK | 설명 |
|---|---|---|---|---|
| `An` 5A 00 `PT` | B0 5A 00 6A | - | - | 장치 스캔 |
| `An` 41 00 `PT` | B0 41 00 71 | - | - | Keepalive |
| `An` 41 00 `PT` | B0 36 `GX` `PT` | `An` 42 00 `PT` | B0 41 00 71 | 통화버튼 누름 (초인종에 응답), 월패드가 통화 거절 |
| `An` 41 00 `PT` | B0 36 `GX` `PT` | `An` 36 `GX` `PT` | B0 42 00 72 | 통화버튼 누름 (초인종에 응답), 통화 시작 |
| `An` 41 00 `PT` | B0 38 00 08 | `An` 38 00 `PT` | B0 42 00 72 | 통화버튼 누름 (영상통화 시작), 통화 시작 |
| `An` 35 00 `PT` | B0 35 00 05 | - | - | 영상통화 연결됨 |
| `An` 41 00 `PT` | B0 42 00 72 | - | - | 통화 중 |
| `An` 41 00 `PT` | B0 3B `GY` `PT` | `An` 3B `GY` `PT` | B0 42 00 72 | 통화중 문열림 버튼 누름, 승인됨 |
| `An` 42 00 `PT` | B0 41 00 71 | - | - | 다른 인터폰이 통화 중 (거절 패킷과 같음) |
| `An` 31 00 `PT` | B0 31 00 01 | - | - | 집 현관 초인종 눌림 |
| `An` 32 00 `PT` | B0 32 00 02 | - | - | 공동현관 초인종 눌림 |
| `An` 3E `GZ` `PT` | B0 3E `GZ` `PT` | - | - | 초인종 꺼짐. |

* `GX`: 01: 현관, 02: 공동현관
* `GY`: 00: 현관, 01: 공동현관
* `GZ`: 01: 현관, 06: 공동현관, 07: 공동현관 (월패드가 A4는 07, A5~A6은 06으로 주는데 의미 불명)

- 인터폰의 비디오/오디오는 RS485가 아닌 별도의 라인으로 전송됩니다.
- 집의 인터폰 개수와 상관 없이 A1~A6을 전부 스캔합니다.
    * 저희 집 기준 A5: 주방 TV, A6: 안방 화장실 이었습니다.
- 집에 공동현관이 여러 층 있어도 패킷에서는 구분되지 않습니다.
- 화장실의 음성통화는 초인종이 울리는 상태에서만 시작할 수 있고, 주방 TV의 영상통화는 아무때나 시작할 수 있습니다.

## 가스밸브: AB

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| AB 5A 00 71 | B0 5A 00 6A | 장치 스캔 |
| AB 41 00 `PT` | B0 41 `XX` `PT` | 가스밸브 열림/닫힘 상태 확인 |
| AB 78 00 53 | B0 78 00 48 | 가스밸브 잠금 |

* `XX`: 00: 열림, 01: 닫힘

- RS485를 이용해 가스밸브를 여는 방법은 없습니다. 그럴듯한 패킷을 만들어서 전송해도 밸브가 무시합니다.

## 조명: AC

| 월패드 | 장치 응답 | 월패드 ACK | 장치 ACK | 설명 |
|---|---|---|---|---|
| AC 5A 00 76 | B0 5A 00 6A | - | - | 장치 스캔 |
| AC 41 00 6D | B0 41 00 71 | - | - | keepalive? |
| AC 79 00 `GR` `PT` | B0 79 `RN` `BM` `PT` | - | - | 조명 상태 확인 |
| AC 79 00 `GR` `PT` | B0 1C 00 00 2C | AC 1C 01 26 `U1 U1` 31 | B0 79 00 00 49 | ? |
| AC 7A `GR` `TG` `PT` | B0 79 `GR` `TG` `PT` | - | - | 조명 제어 |
| AC 7A 00 00 56 | B0 7A 00 00 4A | - | - | 일괄소등 |

* `RN`: 방 번호. 첫 글자(상위 4bit)는 방의 조명 개수, 다음 글자(하위 4bit)는 일련번호입니다.
    * 예시
        * 거실에 조명 스위치 4개이면 방 번호는 41 입니다.
        * 월패드에서 방 구분이 보이지 않는 경우 전부 거실에 있는걸로 취급합니다.
        * 월패드에 거실(2개), 안방(1개), 방1(1개), 방2(1개) 인 경우 <br /> 네 공간 각각 방 번호는 21, 13, 14, 15 입니다.
        * 사례가 적어 검증이 어렵습니다. 위와 다른 케이스가 있으면 제보해 주세요.
* `GR`: 조명 제어시의 그룹 번호는 방 번호와 상관 없이 연속되는 일련번호입니다.
* `BM`: 조명 상태 비트맵. 해당하는 방의 조명 켜짐/꺼짐 상태를 각각 1bit으로 나타냅니다.
    * 예시
        * 방번호 41 조명 켜짐,켜짐,꺼짐,켜짐: 2진수 1011 = 16진수 0B
* `U1 U1`: 04 1E, 22 04 관찰됨

- 집에 따라 밝기 조절 기능도 있다는 정보가 있지만 패킷은 불명입니다.

## 현관스위치: AD

| 월패드 | 장치 응답 | 월패드 ACK | 장치 ACK | 설명 |
|---|---|---|---|---|
| AD 5A 00 77 | B0 5A 00 6A | - | - | 장치 스캔 |
| AD 41 00 6C | B0 41 00 71 | - | - | keepalive |
| AD 41 00 6C | B0 56 00 66 | AD 56 `XX` `PT` | B0 41 00 71 | 현관스위치가 가스밸브 잠금 여부를 물어보면 월패드가 알려줌. <br /> 제품에 따라 keepalive 대신 항상 B0 56으로만 응답하거나 <br /> 12번에 한번만 B0 56을 쓰는 경우도 있습니다. |
| AD 41 00 6C | B0 2F 01 1E | AD 2F 00 02 | B0 41 00 71 | 현관 스위치로 엘리베이터 호출. 장치 ACK 확인하지 않음 |
| AD 41 00 6C | B0 54 `TG` `PT` | AD 54 `TG` `PT` | B0 41 00 71 | 현관 스위치로 일괄소등 제어. 장치 ACK 확인하지 않음 |
| AD 41 00 6C | B0 55 01 64 | AD 55 01 79 | B0 41 00 71 | 현관 스위치로 가스밸브 잠금. 장치 ACK 확인하지 않음 |
| AD 52 00 7F | B0 52 `TG` `PT` | - | - | 일괄소등 여부 확인 |

* `XX`: 00: 열림, 01: 닫힘

- AD 41에 B0 56으로 응답하면 AD 41 - AD 56 - AD 52 의 반복이고, B0 41로 응답하면 AD 41 - AD 52 의 반복입니다.

## 난방: AE

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| AE 5A 00 74 | B0 5A 00 6A | 장치 스캔 |
| AE 7C `GR` 00 00 00 00 `PT` | B0 7C `GR` `TG` `XX` `YY` FF `32` | 난방 상태 확인 |
| AE 7D `GR` `TG` 00 00 00 `PT` | B0 7D `GR` `TG` `XX` `YY` FF `PT` | 난방 켜기/끄기 |
| AE 7F `GR` `XX` 00 00 00 `PT` | B0 7F `GR` `XX` `XX` `YY` FF `PT` | 난방 온도 설정 |

* `XX`: 설정온도 (16진수)
* `YY`: 현재온도 (16진수)

## 환기: C2

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| C2 5A 00 18 | B0 5A 00 6A | 장치 스캔 |
| C2 4E 00 00 00 0C | B0 4E `XX` 00 `YY` `PT` | 환기 상태 확인 |
| C2 4F `ZZ` 00 00 `PT` | B0 4F `ZZ` 00 00 `PT` | 환기 상태 설정 |

* `XX`: 04: 자동, 03: 1단, 02: 2단, 01: 3단, 00: 꺼짐
* `YY`: 01: 꺼짐, 00: 켜짐
* `ZZ`: 06: 끔, 05: 켬, 04: 자동, 03: 1단, 02: 2단, 01: 3단

- 값들이 뒤집혀 있습니다.
- 꺼짐 상태에서 1~3단 속도를 입력했을 때 자동으로 켜지는 장치도 있고, 그렇지 않은 경우도 있습니다.
- 꺼짐 상태에서 XX에 기존 속도를 유지하는 장치도 있고, 00이 되는 경우도 있습니다.

## 대기전력차단 플러그: C6

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| C6 5A 00 1C | B0 5A 00 6A | 장치 스캔. 다수의 플러그가 동시에 응답하여, 패킷이 깨져 등록에 여러번 실패함. <br /> 한번만 성공하면 되므로 큰 문제는 되지 않음 |
| C6 4A `GR` 00 00 00 00 00 00 `PT` | B0 4A `GR` `XX` `YY YY` 00 `ZZ` 00 `PT` | 상태 확인 |
| C6 4B `GR` `TG` 00 00 00 00 00 `PT` | B0 4B `GR` `TG` 00 00 00 00 00 `PT` | 대기전력 차단 설정 |
| C6 6E `GR` `TG` 00 00 00 00 00 `PT` | B0 6E `GR` `TG` 00 00 00 00 00 `PT` | 플러그 전원 설정 |

* `XX`: 01: 평상시, 11: 대기전력차단 설정됨, 10: 전기 차단됨
* `YY`: 현재 전력 소모량 (16진수, 단위 W)
* `ZZ`: 불명 (16진수처럼 보임)

- 대기전력 차단 기능이 꺼진 상태에서는, 플러그가 꺼지지 않습니다.
- 플러그가 꺼진 상태에서 대기전력 차단 기능을 끄면 플러그가 바로 켜집니다.
- 대기전력 차단 기준값은 월패드와 통신하지 않고 스위치에만 저장되는 것처럼 보입니다.

## 실시간 에너지사용량: AA

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| AA 5A 00 70 | B0 5A `XX` `PT` | 장치 스캔 |
| AA 6F `YY` `PT` | B0 6F `YY` `ZZ ZZ ZZ` `PT` | 현재 소모량 확인 |

* `XX`: 14: 전기/가스/수도, 다른 사례 추가 필요
* `YY`: 00: 전기, 01: 가스, 02: 수도
* `ZZ`: 현재 소모량 (10진수 ZZZZZZ)
    * 단위는 전기는 W, 가스/수도는 /100 m³/h 입니다.
    * 예를 들어 B0 6F 00 00 34 56 2F라면, 현재 전력소모량은 3456 W 입니다.
    * 예를 들어 B0 6F 02 00 04 56 2F라면, 현재 수도 유량은 4.56 m³/h 입니다.
  
- 위 값을 리만 합으로 적분하시면 (HA에서 integration 센서 사용) 누적 사용량을 구할 수 있고, HA에서는 utility meter를 추가로 사용해서 일간/월간 사용량을 얻을 수 있습니다. ([예제 YAML](yaml/sds_energy.yaml) 참고하세요.)
- 사례가 적어서 교차검증이 어려운데, 추가 정보가 있다면 제보 부탁 드립니다.

## 현관스위치 신형: CC

| 월패드 | 장치 응답 | 월패드 ACK | 장치 ACK | 설명 |
|---|---|---|---|---|
| CC 5A 01 00 17 | B0 5A 01 00 6B | - | - | 장치 스캔 |
| CC 41 01 00 0C | B0 41 01 00 70 | - | - | keepalive |
| CC 01 16 `YY MM DD HH NN SS` <br /> `U1` `00 x15` `PT` | B0 01 01 00 30 | - | - | 스위치 디스플레이 업데이트 <br /> (날짜, 시각, 날씨) |
| CC 41 01 00 0C | B0 10 01 01 20 | CC 10 01 01 5C | B0 41 01 00 70 | 엘리베이터 호출 |
| CC 12 01 01 5E | B0 41 01 00 70 | - | - | 엘리베이터 도착? |
| CC 41 01 00 0C | B0 08 01 `XX` `PT` | CC 08 01 00 45 | B0 41 01 00 70 | 현관 스위치로 외출모드 설정 <br /> (일괄소등, 가스차단) |
| CC 41 01 00 0C | B0 13 01 00 22 | CC 13 01 01 5F | B0 41 01 00 70 | 현관 스위치로 가스밸브 차단 |
| CC 41 01 00 0C | B0 02 01 00 33 | CC 02 01 00 4F | B0 41 01 00 70 | 현관 스위치로 가스밸브 차단 |
| CC 41 01 00 0C | B0 0A 01 00 3B | CC 0A 01 00 47 | B0 41 01 00 70 | ? |
| CC 41 01 00 0C | B0 0C 01 00 3D | CC 0C 01 00 41 | B0 41 01 00 70 | ? |
| CC 09 03 00 20 00 66 | B0 09 01 00 38 | CC 07 01 00 4A | B0 07 01 01 37 | ? |
| CC 0B 04 `YY` `U2` `U2` 00 `PT` | B0 0B 01 00 3A | - | - | 월패드가 현관 스위치에 다른 기기 상태 전달 |
| CC 02 01 00 4F | B0 41 01 00 70 | - | - | ? |

* `XX`: 00: 설정, 01: 해제
* `YY`: 00: 가스밸브 차단, 01: 가스밸브 열림
* `YY MM DD HH NN SS`: 현재 시각 (16진수)
* `00 x15`: 최저, 최고기온 등 포함된 것으로 예상되는데 정보 없음
* `U1`: FE, FF 관찰됨
* `U2`: 00, 01 관찰됨

- 디스플레이가 달린 신형 현관스위치입니다.
- 사례가 적어서 패킷 분석이 제한적입니다.

## ???: CE

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| CE 5A 01 00 15 | B0 5A 01 00 6B | 장치 스캔 |
| CE 01 05 `XX` 01 01 01 00 `PT` | ? | 불명 |
| CE 41 01 00 0E | ? | 불명 |

* `XX`: 00, 01 확인됨

- 스캔 후 최초 패킷은 CE 01 ~ 이고, 여기에 B0 41 00 71 은 소용없고 (여러번 시도 뒤에 스캔으로 돌아감) <br /> B0 01 05 00 34 로 응답을 해보면 아래 두가지가 불규칙적으로 번갈아 나타납니다.
- 양상을 봤을 때 B0 01 05 00 34 에 뭔가 정보가 들어있고 CE 41 01 00 0E 는 ACK인 것 같습니다. <br /> 뭔지 모르는 정보를 월패드에 계속 던지기가 불안해서 더 이상 시도는 안해봤습니다.

## ???: C1

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| C1 5A 00 1B | ? | 장치 스캔 |

- 카페 게시글에서 존재만 확인 했습니다.

## ???: C3

| 월패드 | 장치 응답 | 설명 |
|---|---|---|
| C3 5A 00 19 | ? | 장치 스캔 |

- 카페 게시글에서 존재만 확인 했습니다.

---
* first written by nandflash("저장장치") <github@printk.info> since 2020-06-29