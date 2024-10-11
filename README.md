# 🚗 Pilot 로봇 프로젝트 보고서
### 이 보고서는 Pilot 라이브러리를 활용한 로봇 프로젝트의 각 단계와 기능을 상세히 설명합니다. 각 기능은 로봇의 자율 주행, 객체 추적, 충돌 회피 등을 포함합니다.


## 📸 1. 카메라 설정
프로젝트의 첫 단계는 로봇이 주변 환경을 인식할 수 있도록 카메라를 초기화하는 것입니다.

```
from pop import Pilot

# 카메라 초기화 (너비 300, 높이 300)
com = Pilot.Camera(width=300, height=300)

```
## 설명
 - Pilot.Camera: 로봇의 시각적 데이터를 수집하기 위해 카메라 객체를 생성합니다. 📷
 - width & height: 카메라의 해상도를 설정합니다. 📏

## 🛠 2. 데이터 수집기 생성
충돌 회피 모델의 데이터를 수집하기 위해 데이터 수집기를 생성합니다.
```
# 데이터 수집기 생성
dataCollector = Pilot.Data_Collector(Pilot.Collision_Avoid, camera=com)

# 데이터 수집기 표시
dataCollector.show()

```

## 설명
- Pilot.Data_Collector: 주어진 모델(Pilot.Collision_Avoid)과 함께 카메라 데이터를 수집합니다. 📊
- show(): 데이터 수집기 UI를 표시합니다. 👀

## 🤖 3. 로봇 초기화
로봇을 초기화하고 기본 속도를 설정합니다.

```
# 로봇 초기화
bot = Pilot.SerBot()
bot.setSpeed(50)  # 속도 설정
```
## 설명
- Pilot.SerBot: 로봇 객체를 생성합니다. 🤖
- setSpeed(50): 로봇의 주행 속도를 50으로 설정합니다. 🚀

## 🎯 4. 객체 추적
사람을 탐지하고 그에 따라 로봇이 이동하도록 합니다.
```
# 객체 추적 객체 생성
OF = Pilot.Object_Follow(cam)
OF.load_model()  # 모델 로드

while True:
    v = OF.detect(index='person')  # 사람 탐지
    if v is not None:
        steer = v['x'] * 4  # 감지된 객체의 위치에 따라 조향 조정
        bot.steering = max(-1, min(1, steer))  # 조향 범위 제한
        if v['size_rate'] < 0.20:
            bot.forward(50)  # 객체가 가까운 경우 전진
        else:
            bot.stop()  # 객체가 멀리 있는 경우 정지
    else:
        bot.stop()  # 객체가 없을 경우 정지

```

## 설명
- Pilot.Object_Follow: 객체 추적 기능을 위한 객체를 생성합니다. 🎯
- detect(index='person'): 지정된 인덱스(여기서는 'person')를 사용하여 객체를 탐지합니다. 🕵️
- steering: 로봇의 조향을 조정하여 감지된 객체의 위치에 따라 방향을 설정합니다. ➡️

## 🚧 5. 충돌 회피
장애물을 감지하고 회피하는 기능을 구현합니다.
```
# 충돌 회피 객체 생성
CA = Pilot.Collision_Avoid(cam)
CA.load_datasets()  # 데이터셋 로드
CA.train(times=10)  # 학습 진행
CA.show()  # 결과 표시

def drive(value):
    if value <= 0.5:  # 장애물이 가까운 경우
        bot.steering = 0  # 직진
        bot.forward()
    else:  # 장애물이 멀리 있는 경우
        bot.steering = 1  # 후진
        bot.backward()

# 충돌 회피 루프 시작
try:
    while True:
        CA.run(callback=drive)  # 장애물 감지 및 드라이브
except KeyboardInterrupt:
    print("충돌 회피 루프가 중단되었습니다.")

```

## 설명
- Pilot.Collision_Avoid: 충돌 회피 기능을 위한 객체를 생성합니다. ⚠️
- load_datasets(): 필요한 데이터셋을 로드합니다. 📁
- train(times=10): 모델을 10번 학습합니다. 📚
- drive(value): 장애물의 거리에 따라 로봇의 이동 방향을 결정합니다. 🚦

## 🌟 6. 트랙 추적
로봇이 트랙을 따라 움직이도록 설정합니다.
```
# 트랙 추적 객체 초기화
CF = Pilot.Track_Follow(camera=com)
CF.load_datasets()  # 데이터셋 로드
CF.train(times=5)  # 학습 진행
CF.show()  # 결과 표시

def drive(value):
    bot.forward()
    steer = value['x']
    
    if steer > 1:
        steer = 1
    elif steer < -1:
        steer = -1
    bot.steering = steer * 1.5
    
    while True:
        CF.run(callback=drive)  # 트랙 따라 주행

```

## 설명
- Pilot.Track_Follow: 트랙 추적 기능을 위한 객체를 생성합니다. 🛤️
- load_datasets(): 트랙 데이터셋을 로드합니다. 📊
- train(times=5): 5번 학습합니다. 📈
- drive(value): 트랙의 위치에 따라 로봇의 방향을 조정합니다. 🔄

## 🎶 7. 조도 센서 및 음향 경고
조도 센서를 사용하여 주변 밝기를 감지하고, 조도가 낮을 때 경고음을 재생합니다.
```
from pop import Cds, delay

# 조도 센서 설정
cds = Cds(7)

try:
    with Tone() as tone:
        while True:
            value = cds.readAverage()  # 조도 값을 읽어오기
            print(f"조도 값: {value}")

            if value < 50:  # 조도가 너무 낮을 경우
                print("주의: 조도가 너무 낮습니다!")
                tone.play(3, 10, 1)  # 경고음 재생

            # 조도 값에 따라 자동차 속도 설정
            speed = value / 16  # 조도를 기반으로 속도 조정
            car.setSpeed(speed)

            # 조도 값에 따른 동작 결정
            if value >= 700:
                car.forward()  # 밝으면 전진
            elif value >= 50:
                car.forward()  # 중간 밝기에서 전진
            else:
                car.stop()  # 어두워지면 정지
                print("어두워졌습니다. 자동차가 정지합니다.")

            delay(500)  # 500 밀리초 대기
except KeyboardInterrupt:
    car.stop()  # 정지 버튼을 누르면 즉시 멈춤
    print("프로그램이 종료되었습니다.")

```

## 설명
- Cds(7): 조도 센서를 초기화합니다. 💡
- readAverage(): 평균 조도 값을 읽어옵니다. 📊
- tone.play(3, 10, 1): 3옥타브의 도음을 1초 동안 재생합니다. 🎶

## 🔄 8. 사용자 입력으로 카메라 제어
사용자로부터 입력을 받아 카메라의 팬과 틸트 각도를 설정합니다.
```
# 사용자 입력으로 팬 및 틸트 각도 설정
while True:
    try:
        pan_angle = int(input("팬 각도 (0-180) 설정: "))
        if 0 <= pan_angle <= 180:
            Car.camPan(pan_angle)  # 팬 각도 설정
            print(f"카메라 팬 각도: {pan_angle}도")
        else:
            print("잘못된 각도입니다. 0에서 180 사이의 값을 입력하세요.")
        
        tilt_angle = int(input("틸트 각도 (0-180) 설정: "))
        if 0 <= tilt_angle <= 180:
            Car.camTilt(tilt_angle)  # 틸트 각도 설정
            print(f"카메라 틸트 각도: {tilt_angle}도")
        else:
            print("잘못된 각도입니다. 0에서 180 사이의 값을 입력하세요.")
        
        continue_operation = input("계속하시겠습니까? (예/아니요): ").strip().lower()
        if continue_operation != '예':
            break
    except ValueError:
        print("유효한 숫자를 입력하세요.")

# 카메라 팬과 틸트를 0도로 되돌리기
Car.camPan(0)
Car.camTilt(0)
print("카메라 팬과 틸트를 0도로 되돌렸습니다.")

```

## 설명
- 사용자로부터 팬과 틸트 각도를 입력받아 카메라를 조정합니다. 🎛️
- 입력값의 유효성을 검사하여 잘못된 값은 처리합니다. ⚠️

## 🚦최종 정리
이 프로젝트는 로봇의 자율 주행, 객체 추적, 충돌 회피 및 환경 감지를 위한 다양한 기능을 통합합니다. 각 단계는 로봇이 안전하고 효과적으로 작동할 수 있도록 설계되었습니다. 🛠️


https://github.com/user-attachments/assets/b988bcd7-29b7-4de1-bffe-d4be62ffa73c



https://github.com/user-attachments/assets/63ff7e08-0c0f-4d24-bbbe-a13a71049ca4



https://github.com/user-attachments/assets/e098f051-8240-46f3-875f-6064831a5110



https://github.com/user-attachments/assets/258e0f0a-6afd-4eed-baaf-b162f7c004b4



https://github.com/user-attachments/assets/03d61a75-757c-4617-b620-4f479aa5a443



https://github.com/user-attachments/assets/23f62322-9272-474e-8afc-606ec215226c

