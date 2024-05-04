# DMA(Direct memory access)

입출력 장치가 CPU를 거치지 않고 메인 메모리와 직접 데이터를 주고 받을 수 있도록 하는 기술입니다.   
불필요한 데이터 흐름을 최적화하고 CPU의 데이터 복사 오버헤드를 줄여줍니다.

## DMA가 없을 떄의 동작 방식

DMA가 없을 떄의 동작 방식부터 DMA가 어떠한 이점이 있어 도입하는지 알아보겠습니다.

![alt text](<image/DMA가 없을 때의 동작 방식.png>)

위 그림은 DMA가 없을 때의 동작 방식 순서입니다.   

1. RAM에 입출력 버퍼 공간을 할당합니다.
2. 입출력 장치의 RAM에 입출력이 완료되면, CPU가 입출력 장치의 RAM로부터 메인 메모리로 데이터를 복사합니다.

CPU는 입출력 데이터를 사용하기 위해 입출력 램 -> 메인 메모리로 복사하는 불필요한 동작을 해야합니다. 단순히 이뿐만 아니라 입출력 장치에 따라 추가적인 동작도 수행해야 할 수 있습니다.

## DMA 없을 때의 구체적인 예시(TCP/IP)

TCP/IP 통신 시에 CPU는 데이터 복사 뿐만 아니라 추가적인 동작을 수행해야 합니다.

![alt text](<image/DMA 구체적인 예시 TCPIP.png>)

위 그림이 TCP/IP 시의 저장 과정입니다.   
입력 그림을 보게 되면, 단순히 메인 메모리에 저장된 데이터를 NIC의 RAM에 저장해주는 것 뿐만 아니라, TCP/IP 시에 일어나는 Segment도 해주어야 합니다.

## DMA를 추가하여 오버헤드를 줄여보자

![alt text](<image/DMA 전체 구조.png>)

DMA가 추가 되면 위와 같은 구조가 됩니다. 지금까지 CPU가 해줘야 했던 복사와 TCP/IP Segment와 같은 동작을 DMA에 위임하고 CPU는 메인 메모리의 데이터만 가지고 작업을 합니다. 이렇게 되면 CPU 부하를 줄여줄 뿐만 아니라 DMA는 입출력 장치와 메인 메모리에 특화된 장치이기 때문에 최적화된 동작을 수행할 수 있습니다.

## DMA의 단점

1. 복잡성 증가: DMA라는 추가적인 컴포넌트가 필요하고, CPU와 DMA가 소통을 하며 동작해야 하기 때문에 복잡성이 증가합니다.
2. 버스 경쟁: CPU와 DMA가 동시에 메모리에 접근하려 할 때, 버스를 경쟁할 수 있습니다.
3. 보안 취약성: CPU만 메인 메모리에 접근하는 권한을 가지는 것이 아니라, DMA도 메인 메모리에 대한 접근 권한을 갖기 때문에 보안에 주의를 기울여야 합니다.
4. 캐시 일관성: CPU의 캐시의 값이 DMA가 최신화한 값이 아닌 이전의 값을 가지고 있다면 일관성이 깨질 수 있습니다.

## 버스 모드

버스의 경쟁에서 효율적인 동작을 하기 위해 몇가지 모드가 있습니다.

- Burst mode: DMA 컨트롤러가 CPU로부터 액세스 권한을 부여받으면, 전체 데이터 블록을 하나의 연속 시퀀스로 전송합니다. 완료되면 버스 제어권이 다시 CPU로 넘어갑니다.

- Cycle stealing mode: 액세스 권한을 획득하는 방식은 동일하지만, 1바이트만 전송한 뒤 버스 제어권이 CPU로 넘어갑니다.

- Transparent mode: CPU가 시스템 버스를 사용하지 않는 작업을 실행하는 경우에만 데이터를 전송합니다.

## 참조

- [DMA를 알면 고성능 소켓이 보인다! - 널널한 개발자](https://www.youtube.com/watch?v=VmclwfKzNO0)

- [Direct Memory Access (DMA) - techtarget](https://www.techtarget.com/whatis/definition/Direct-Memory-Access-DMA)

- [Direct Memory Access - techopedia](https://www.techopedia.com/definition/2767/direct-memory-access-dma)

- [Direct memory access - wikipedia](https://en.wikipedia.org/wiki/Direct_memory_access)

- [What Is Direct Memory Access (DMA)? - spiceworks](https://www.spiceworks.com/tech/hardware/articles/direct-memory-access/)
