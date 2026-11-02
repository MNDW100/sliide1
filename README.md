# p5.js 슬라이더로 Arduino LED 밝기 조절해보겠습니다 
프로젝트 소개: p5.js의 `createSlider()`로 0~255 범위의 밝기 값을 선택하고 Web Serial API를 이용하여 Arduino Uno로 전송하고 Arduino는 수신한 값을 `analogWrite()`에 적용하여 LED의 PWM을 변경하는 방식이다
#회로도
<img width="951" height="615" alt="스크린샷 2026-11-02 212757" src="https://github.com/user-attachments/assets/68054242-9c2d-466f-bc67-d957bce1ccbf" />

## Arduino 코드
```arduino
#include <TaskScheduler.h>
#include <stdlib.h>

// 1. 핀과 상태 변수
const uint8_t LED_PIN = 9;

int ledBrightness = -1;

char command[8];
uint8_t commandLength = 0;
bool invalidCommand = false;

// 2. 함수 선언
void setLED(int value);
void taskSerialCallback();

// 3. 작업 관리자와 시리얼 작업
Scheduler runner;

Task tSerial(10, TASK_FOREVER, &taskSerialCallback, &runner, true);

// 4. 초기 설정
void setup()
{
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);

  Serial.println("System Ready");
  setLED(0);
}

// 5. 실행할 시간이 된 작업 처리
void loop()
{
  runner.execute();
}

// 6. LED 출력은 이 함수에서만 변경
void setLED(int value)
{
  value = constrain(value, 0, 255);

  if (value == ledBrightness) {
    return;
  }

  ledBrightness = value;
  analogWrite(LED_PIN, ledBrightness);

  // 적용한 PWM 값을 p5.js에 보고
  Serial.print("LED:");
  Serial.println(ledBrightness);
}

// 7. 수신 문자를 모아 줄바꿈에서 처리
void taskSerialCallback()
{
  while (Serial.available() > 0) {
    const char c = Serial.read();

    if (c == '\n' || c == '\r') {
      if (invalidCommand) {
        Serial.println("ERROR:Invalid brightness");
      } else if (commandLength > 0) {
        command[commandLength] = '\0';

        const int value = atoi(command);

        if (value >= 0 && value <= 255) {
          setLED(value);
        } else {
          Serial.println("ERROR:Range 0-255");
        }
      }

      commandLength = 0;
      invalidCommand = false;
    } else if (!invalidCommand) {
      // 0~255는 최대 세 자리
      if (c >= '0' && c <= '9' && commandLength < 3) {
        command[commandLength++] = c;
      } else {
        invalidCommand = true;
      }
    }
  }
}
```
# p5.js코드
<iframe src="https://editor.p5js.org/MNDW100/full/Avq2FrCt6"></iframe>
# 구현영상 

# 고찰 
이번 프로젝트를 통해 p5.js 슬라이더의 값을 시리얼 통신으로 Arduino에 전달하고 PWM으로 LED 밝기를 조절하는 원리가 이해됐다. 특히 통신 속도와 줄바꿈 규칙을 맞추는 것이 중요하다느꼇고 값이 변경될 때만 전송하여 불필요한 통신을 줄일 수 있음을 알게되었다. 향후 Arduino의 응답을 받아 실제 적용값을 화면에 표시하면 연결 상태와 동작 여부를 더욱 명확하게 확인할 수 있을 것이다 

