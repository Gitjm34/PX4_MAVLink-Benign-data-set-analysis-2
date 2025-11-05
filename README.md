# PX4_MAVLink-Benign-data-set-analysis-2

# RC Data Analysis & MAVLink Log Processing  
**PX4 기반 드론의 RC 입력 분석 및 MAVLink 데이터 로깅**  

## Introduction  
본 프로젝트는 **PX4 드론의 RC 입력 데이터(`RC_CHANNELS`), MAVLink 로그 분석(`tlog`, `vehicle.csv`) 및 GPS 스푸핑 실험**을 수행하는 연구의 과정이다.  
**조종기 입력값을 분석하여 드론의 상태를 평가**하고, **GPS 신호 변조 및 전파 교란(GPS Spoofing & Jamming)이 비행 데이터에 미치는 영향을 분석하는 것**을 목표로 한다.  

---

## 📡 **MAVLink & RC Channels Data Analysis**  
### 🔹 **MAVLink의 RC_CHANNELS 데이터란?**  
**RC_CHANNELS 메시지**는 **MAVLink 프로토콜을 통해 드론이 수신한 조종기(RC) 입력 값을 나타내는 데이터**이다.  
각 채널 값은 **PWM 신호(1000~2000 범위)로 표현**되며, 이는 드론의 비행 동작(스로틀, 피치, 롤, 요 등) 및 특정 기능(모드 전환, LED ON/OFF, 서보모터 조작 등)에 직결된다.  

### 🔹 **RC 채널 값 분석**  
- **정상적인 비행 패턴:**  
  - RC 입력값이 일정한 범위를 유지하며 변화가 자연스럽게 발생  
  - 예: **스틱 입력 시 1000~2000 범위에서 부드럽게 변화**  

- **이상 징후 및 공격 탐지:**  
  - **GPS Spoofing** → 조종기 입력 없이 **위치 정보가 급격히 변동**  
  - **GPS Jamming** → 특정 RC 채널이 급격히 변하거나, **고정된 값으로 유지됨**  
  - **데이터 변조 공격** → 예상과 다른 RC 채널 값 변동, **조종기 입력값과 불일치하는 패턴** 발생  

### **RC 채널 데이터 예시**  
#### **정상적인 RC 입력값 (수동 조종)**
| Timestamp | CH1 (Roll) | CH2 (Pitch) | CH3 (Throttle) | CH4 (Yaw) | CH5 (Mode) |
|-----------|------------|-------------|----------------|-----------|------------|
| 00:01:23  | 1500       | 1498        | 1000           | 1500      | 2000       |
| 00:01:24  | 1520       | 1510        | 1100           | 1495      | 2000       |

#### **GPS 스푸핑 발생 후 (이상 징후)**
| Timestamp | CH1 (Roll) | CH2 (Pitch) | CH3 (Throttle) | CH4 (Yaw) | CH5 (Mode) |
|-----------|------------|-------------|----------------|-----------|------------|
| 00:02:30  | 3748       | 4690        | 3495           | 2375      | 3455       |
| 00:02:31  | 3801       | 4570        | 3444           | 2687      | 3568       |
➡ **조종기 입력이 없는 상태에서 GPS 좌표가 급격히 변동됨 (스푸핑 징후).**

---

## 🛰️ **GPS Spoofing & Jamming Experiment**  
### 🔹 **실험 개요**  
본 연구에서는 **HackRF 및 GPS-SDR-SIM을 활용하여 PX4 드론의 GPS 신호를 변조하거나 방해(Jamming)하여 비행 데이터에 미치는 영향을 분석**한다.  

### 🔹 **실험 목표**  
**GPS Spoofing**: 가짜 GPS 신호를 송출하여 드론이 잘못된 위치로 이동하도록 유도  
**GPS Jamming**: GPS 신호를 차단하여 드론의 위치 인식을 방해  
**MAVLink 데이터 비교**: 정상 비행과 공격 후 비행 데이터를 비교하여 변화를 분석  

### 🔹 **실험 방법**  
1 **기본 환경 설정**  
   - PX4 드론, QGroundControl(QGC), HackRF 세팅  
   - 정상적인 비행 경로 및 `tlog` 데이터 확보  

2 **GPS Spoofing 신호 생성 및 송출**  
   - **GPS-SDR-SIM을 이용해 변조된 GPS 데이터 (`.bin → .c16`) 생성**  
   - HackRF로 **GPS L1(1575.42MHz) 대역에 스푸핑 신호 송출**  
   - 드론이 목표 좌표로 이동하는지 관찰  

3 **비행 로그 분석**  
   - **tlog(MAVLink 메시지)와 vehicle.csv 데이터 비교**  
   - **GPS 데이터 변조 성공 여부 및 비행 패턴 변화 분석**  
   - MAVLink 메시지 중 **GPS_RAW_INT, GLOBAL_POSITION_INT** 분석  

### 🔹 **실험을 위한 주요 명령어**
```bash
# GPS-SDR-SIM을 사용하여 가짜 GPS 신호 생성
./gps-sdr-sim -e brdc0720.25n -l 37.23918,127.0849,100 -d 120 -b 8 -o gpssim.bin

# HackRF를 이용한 GPS 스푸핑 실행
hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 40
