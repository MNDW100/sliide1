# sliide1

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
