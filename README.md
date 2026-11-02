# sliide1

## Arduino 코드

아래 코드는 내장 LED를 1초 간격으로 켜고 끄는 예제입니다.

```cpp
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  delay(1000);
}
```
