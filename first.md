# 운영체제 만들기
## 0부터 시작하는 OS자작 입문
이 책을 공부하며 기록한 글입니다.
## UEFI BIOS

- BIOS란 컴퓨터 전원을 처음 켰을 때 실행되는 펌웨어
```
펌웨어란 특정 하드웨어 장치에 포함된 소프트웨어로 하드웨어의 구동을 담당하는 작은 운영체제 역할을 수행
```
- Basic Input Output System의 줄인말로 OS kernel이 메모리에 올라오기 전 컴퓨터 동작을 담당
- 기본적으로 컴퓨터 내부를 초기화하고, OS의 부트로더를 스토리지로부터 읽어들이는 기능 제공
```
부트로더란 운영체제를 시동시키는 기능을 하는 프로그램
```
- UEFI란 'Unified Extensible Firmware Interface'의 약자로 회사마다 달랐던 BIOS구조를 단일화한 인터페이스로 맞추기 위한 규약
- 본 운영체제 만들기에서는 'GOP(Graphics Output Protocol)'를 사용하여 그래픽 출력이 가능한 운영체제로 제작

## Hello World
아래의 바이너리 파일은 UEFI BIOS에서 동작하는 가장 기초적인 프로그램이다.
화면에 Hello world를 출력하는 기능만을 수행한다.

![image](https://user-images.githubusercontent.com/30014736/183399959-7633c9c3-9675-4183-850f-08871abf6ea7.png)

---
- 원래는 안쓰는 컴퓨터가 한 대 있으면 그 컴퓨터에 USB로 해당 파일을 넣은 후 부팅 우선순위를 USB를 올려주어서 실행하면 된다.
- 하지만 서브 컴퓨터가 없는 사람들을 위해 리눅스 환경에서 에뮬레이터를 사용하여 실행할 수 있다.
- 우리는 QEMU라는 시뮬레이터와 이를 그래픽으로 구현해줄 XLaunch라는 프로그램을 사용하려고 한다.
```
QEMU는 가상 PC를 재현해주는 에뮬레이터이다. 
처음에는 우리가 이때까지 써왔던 VMware와 VirtualBox를 사용해 보려고 시도했다.
하지만 두 툴은 등록된 상용 운영체제들의 이미지 파일만을 인식할 수 있었기에 우리가 만든 운영체제는 실행할 수 없었다.

XLaunch는 윈도우상에서 WSL2를 이용하여 리눅스를 사용할 때 단순 텍스트 인터페이스만을 지원하는 리눅스에서 그래픽을 사용할 수 있게 지원해주는 툴이다.
나는 윈도우상에서 WSL2를 이용해서 코딩을 진행했기에 XLaunch로 결과를 확인하고 있다.
```
UEFI애플리케이션인 hello.efi를 실행한 결과이다. 

![image](https://user-images.githubusercontent.com/30014736/183401777-4334d42f-a69d-47ce-9cdb-1b13d1cc1113.png)

```
elf파일은 리눅스에서의 실행파일 형실을 의마한다.
윈도우에서는 pe형식의 파일을 사용하며 보통 확장자는 .exe로 표현된다.
```
---
#### EDK2
- EDK2는 intel이 UEFI와 관련된 개발 키트를 만들어 오픈소스로 공개한것이다.
- UEFI BIOS자체개발과, UEFI BIOS상에서 돌아가는 애플리케이션 개발에 사용되는 개발 키트이다.
- EDK2 툴을 사용하여 앞으로 c, cpp파일을 실행가능한 파일로 컴파일 해 줄 것이다.

## 메모리 맵 취득
- 이제는 UEFI 애플리케이션에서 메모리를 취득하는 방법에 대해서 살펴볼 계획이다.
- 여기 나오는 gBS는 edk2툴에서 제공하는 전역변수이다. 
- 부트 서비스에서 사용할수 있는 다양한 기능을 제공한다. 

![image](https://user-images.githubusercontent.com/30014736/183404740-d6761060-30ab-481a-9b19-15742e9f8c5a.png)

- 첫번째 파라미터는 메모리맵 싸이즈이다. 맵 쓰기용 영역의 크기를 넘기면 실제 메모리맵 크기가 반환된다.
- 두번째 파라미터는 메모리 영역의 시작점이다. 
- 세번째 파라미터는 메모리맵을 식별하기 위한 변수이다. 
- 네번째는 메모리 디스크립터(메모리 맵의 행의 크기)의 바이트수이다.
- 다섯번째는 메모리 디스크립터의 버전이다.(사용 x)

해당 함수로 얻은 메모리 맵을 화면에 출력하면 다음과 같다.
![image](https://user-images.githubusercontent.com/30014736/183406100-2578cbf4-1c80-46c0-9285-f966cb60c986.png)

#### QEMU 모니터
우리가 사용하고 있는 qemu는 다양한 기능을 지원한다.
운영체제 제작에서 디버깅을 위한 기능을 지원한다.
예를들어 CPU 레지스터 값을 확인하기 위해서 `info registers`명령 사용하면 각종 레지스터의 현제 값을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/30014736/183406545-0758c22b-8a15-49cc-8fbc-271839a17fde.png)

- 보통 우리가 알고 있는 PC(Program Counter)는 RIP레지스터이다.

디버깅에서 자주 사용하는 메모리 덤프기능도 사용할 수 있다.
`x /format address` 형식을 이용해서 확인할 수 있다.
![image](https://user-images.githubusercontent.com/30014736/183406986-5b81b280-4e93-4fd4-9401-34636f511013.png)
![image](https://user-images.githubusercontent.com/30014736/183407062-ad1dfdf8-3484-4365-a12a-e8c58b9d81f1.png)

이처럼 포멧에 따라서 메모리 자체의 값도 확인할 수 있고, 어셈블리 명령어도 확인할 수 있다.

# 커널
이때까지 살펴본 프로그램은 UEFI BIOS에서 돌아가는 프로그램이다. 즉 커널을 동작시키기 위한 부트로더였다.
이제 부트로더를 이용해서 커널을 메모리에 올리고 실행시켜본다.

![image](https://user-images.githubusercontent.com/30014736/183407720-3107722c-6707-49bc-adb2-c2be0ab8cfe4.png)

부트로더에서 커널을 메모리에 할당시키는 가장 중요한 코드이다. 실행순서는 아래와 같다.
- 먼저 커널 실행파일을 열어서 파일의 정보를 얻는다.
- 그 다음 커널파일 사이즈만큼을 메모리에 할당한다. 
- 할당한 이후 모든 파일을 읽어들인다.
- 다음으로 커널 함수의 시작 포인터를 지정해준다.
- 커널 함수를 실행한다.

![image](https://user-images.githubusercontent.com/30014736/183409324-f56bf24f-a672-478a-acd6-2d2491ea07a6.png)

이제 커널파일의 엔트리포인트를 설정해주고 실행하는 부분이다. 
엔트리포인트의 이름은 고정되어 있지 않기 때문에 위치를 지정해 주어야 한다.
커널 시작 메모리주소는 위의 코드에서 0x100000으로 고정해 놓았고, 엔트리포인트는 elf파일의 0x1000번째에 적혀있다.
그래서 해당 부분의 위치를 기록하고 이를 함수 포인터를 이용해서 실행함으로써 커널함수를 실행할 수 있다.


