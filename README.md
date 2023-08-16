## 프로젝트 정보

매직미러를 통해 다른 컴퓨터, 라즈베리파이, 혹은 TOPST보드와 통신하여 vlc와 GPIO를 통해 라디오를 들려줍니다.

## 프로젝트 배경

텍스트를 사용자에게 보여주면서 라디오나 음악을 라인트레이서와 같은 차에서 함께 재생할 수 없을까? 라는 생각에서 이 프로젝트가 시작되었습니다.

## 프로젝트 진행과정

nodejs를 통해 서버를 구동하고 클라이언트 측에서 Text 및 image를 소켓통신을 활용하여 서버에게 전송합니다. 또한 받은 메세지를 통해 라디오를 재생시킵니다.

## 소프트웨어 기능 리스트

- socketio를 활용한 통신
- 디자인 css
- node.js를 활용한 서버
- Text 및 image를 화면에 실시간으로 보여주는 js파일
- vlc를 활용한 라디오 재생
- Thread를 활용한 버튼 상태 확인

## 하드웨어 구성 리스트

- TOPOST 보드

Config 구성

```jsx
{
			module: "Socket-Text",
			position: "top_center",
			config: {
				Text : 'hello world'
			}
		},
```

Radio.py파일을 해당하는 주소로 수정해야 합니다.

```python
import os
import socketio
import time
import threading

# 소켓 클라이언트 생성
sio = socketio.Client()

@sio.on('connect')
def on_connect():
    print('Connected to server')

@sio.on('disconnect')
def on_disconnect():
    print('Disconnected from server')

# GPIO 핀 번호 설정
button_pin = 90  # 버튼의 GPIO 핀 번호

# GPIO 핀 모드 설정
def setup_gpio(pin):
    os.system(f"echo {pin} > /sys/class/gpio/export")
    os.system(f"echo in > /sys/class/gpio/gpio{pin}/direction")

# 버튼 상태를 읽는 함수
def read_button_state(pin):
    with open(f"/sys/class/gpio/gpio{pin}/value", "r") as file:
        return int(file.read())

# 버튼을 눌렀을 때 실행할 함수
def Play_Radio():
    sio.emit('radio', 'play_radio')
    print('Button pressed! Sending play_radio')

def Stop_Radio():
    sio.emit('radio', 'stop_radio')
    print('Button pressed again! Sending stop_radio')

# GPIO 설정 및 버튼 이벤트 처리
def setup_and_poll_button(pin):
    setup_gpio(pin)
    prev_button_state = 0
    button_press_count = 0  # 버튼 눌림 횟수
    while True:
        button_state = read_button_state(pin)
        if button_state == 1 and prev_button_state == 0:  # 버튼이 눌렸을 때
            button_press_count += 1
            if button_press_count == 1:
                Play_Radio()
            elif button_press_count == 2:
                Stop_Radio()
                button_press_count = 0  # 초기화
        prev_button_state = button_state
        time.sleep(0.1)

if __name__ == "__main__":
    try:
        button_thread = threading.Thread(target=setup_and_poll_button, args=(button_pin,))
        button_thread.start()

        sio.connect('http://##############:3000')  # 서버 IP 주소로 변경

        while True:
            time.sleep(1)

    except KeyboardInterrupt:
        sio.disconnect()
        os.system(f"echo {button_pin} > /sys/class/gpio/unexport")
```

1. 터미널을 열고 다음 명령을 입력하여 Node.js 설치 스크립트를 내려받습니다.
    
    ```bash
    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
    ```
    
2. 다음 명령으로 Node.js를 설치합니다.
    
    ```bash
    sudo apt-get install -y nodejs
    ```
    
3. 설치가 완료되면 다음 명령으로 Node.js와 npm의 버전을 확인합니다.
    
    ```bash
    node -v
    npm -v
    ```
    

**Python-SocketIO 라이브러리 설치**:
Python에서 Socket.IO를 사용하기 위해 **`python-socketio`** 라이브러리를 설치해야 합니다. 터미널에서 다음 명령어를 사용하여 라이브러리를 설치하세요

```bash
pip install python-socketio
```

- http통신을 위한 requests를 설치하십시오

```bash
pip install requests
```
