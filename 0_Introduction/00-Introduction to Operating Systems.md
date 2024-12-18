# Introduction to Operating Systems

## 프로그램이 실행될 때 어떤 일이 일어날까?

매우 단순하다. 명령어를 실행한다!

1. 명령어 가져오기 (fetch)
2. 명령어 해석하기 (decode)
3. 명령어 실행하기 (execute)
4. 다음 명령어로 이동하며 프로그램이 종료될 때까지 반복한다.

이것이 폰 노이만 컴퓨팅 모델의 기본 개념이다.

## 운영체제의 역할

이외에도 주요 목표인 시스템을 '사용하기 쉽게하기 위해' 프로그램 실행 시 다양한 많은 일들이 발생한다.

- 프로그램을 쉽게 실행할 수 있게 해준다.
- 프로그램 간 메모리를 공유할 수 있게 해준다.
- 장치와의 상호작용을 지원해준다.
- 그 외 다양한 기능들을 가능하게 해준다.

이러한 소프트웨어의 집합체를 운영 체제라고 한다.
운영 체제는 시스템이 올바르고 효율적으로 동작하면서 사용하기 쉽게 만들어주는 역할을 담당한다.

### Virtualization

운영 체제는 이런 작업을 하기 위해서 가상화라는 기법을 사용한다.
물리적 리소스(프로세서, 메모리, 디스크 등)을 가져와서 보다 일반적이고, 강력하며 사용하기 쉬운 가상 형태로 변환한다.
그래서 운영체제를 virtual machine이라고 부르기도 한다.

### System Calls

사용자가 운영 체제에게 명령을 내리고, 가상 머신의 기능(프로그램 실행, 메모리 할당, 파일 접근 등)을 사용할 수 있도록 API를 제공한다.
운영체제는 보통 수백 개의 system calls를 제공한다.
이러한 system calls를 제공해주기 때문에, 운영체제가 standard library를 제공한다고 말하기도 한다.

### Resource manager

가상화를 통해 여러 프로그램이 (CPU를 공유해서) 동시에 실행되고, 각 프로그램이 (메모리를 공유해서) 동시에 각자의 명령어와 데이터에 접근하고, 여러 프로그램이 (디스크 등의) 장치를 공유할 수 있기 때문에
운영 체제를 Resource Manager라고 부르기도 한다.
CPU, 메모리, 디스크는 시스템의 자원으로, 운영체제는 이러한 자원들을 효율적이고 공정하게 관리하는 역할을 한다.

## 1. Virtualizing The CPU

- 하나의 CPU(또는 소수의 CPU)를 마치 무한히 많은 CPU가 존재하는 것처럼 만들어서, 여러 프로그램이 동시에 실행되는 것처럼 보이게 하는 것이 CPU 가상화이다.
- 다수의 프로그램을 동시에 실행시키는 기능은 여러 문제를 발생시킨다.
  - 특정 순간에 두 개의 프로그램이 실행되어야 한다면, 어떤 것이 실행되어야 하는지와 같은 문제 -> 이 질문의 답은 운영체제의 정책에 의해 결정된다.
- 운영체제는 Resource manager로서의 역할을 수행하게 된다.

## 2. Virtualizing Memory

- 컴퓨터에서의 물리 메모리(physical memory) 모델은 매우 단순한 바이트의 배열이다.
- 프로그램이 실행되는 동안 메모리는 끊임없이 사용된다(accessed).
- 프로그램은 모든 데이터 구조를 메모리에 저장하고 명령어를 통해 데이터를 처리한다. 이 명령어 자체도 메모리에 저장되어 있기 때문에, 명령어를 가져오는(fetch) 과정에서도 메모리는 지속적으로 접근된다.

### Virtual Address Space (address space)

- 각 프로세스는 자신만의 가상 주소 공간(virtual address space)을 갖는다.
- 운영체제는 이 가상 주소 공간을 컴퓨터의 물리 메모리로 매핑한다.
- 각 프로그램의 입장에서는 마치 자신만이 물리적 메모리를 독점적으로 사용하는 것 같지만, 실제로는 물리 메모리는 운영체제가 관리하는 공유 자원이다.

## 3. Concurrency

- 프로그램이 동시에 여러 작업을 수행할 때 발생하는 문제를 가리킬 때 이 용어를 사용한다.
- 병행성 문제는 처음에는 운영체제 자체에서 발생했다.
  - 예를 들어, 한 프로세스를 실행하다가 다른 프로세스를 실행하는 방식으로 여러 작업을 번갈아가며 처리하는 작업을 관리하다가 발생하는 문제들..
- 병행성 문제는 운영체제에만 국한되지 않고 멀티 스레드 프로그램에서도 동일한 문제가 발생한다.

## 4. Persistence

- 시스템 메모리에서는 데이터를 쉽게 잃을 수 있다.
  - DRAM과 같은 장치는 데이터를 휘발성(volatile) 방식으로 저장하기 때문에 전원 공급이 끊어지거나 시스템이 고장나면 모든 데이터가 사라지므로, 데이터를 영속적으로 저장할 수 있는 하드웨어와 소프트웨어가 필요하다.
- 하드웨어 측면에서는 입출력(I/O) 장치가 이를 담당한다.
  - 현대 시스템에서는 하드 드라이브가 일반적으로 담당하며, SSDs(solid-state drives)도 많이 사용되고 있다.
- 소프트웨어 측면에서는 운영체제에서 디스크를 관리하는 소프트웨어를 파일 시스템이라고 부른다.
  - 파일을 디스크에 신뢰성 있고 효율적인 방식으로 저장하는 역할을 한다.
- CPU나 메모리 가상화와 달리 가상 디스크는 프로그램 별로 따로 생성하지 않는다.

## 5. Design Goals

- 운영체제가 하는 일
  - 운영체제는 CPU, 메모리, 디스크와 같은 물리 자원을 가상화한다.
  - 운영체제는 병행성과 관련된 복잡한 문제를 처리한다.
  - 운영체제는 파일을 영속적으로 저장하여 오랜 시간 동안 안전한 상태에 있게 한다.
- 이런 시스템을 구현하려면 몇 가지 목표를 세워야 한다.
  - (1) 가장 기본적인 목표: 시스템을 편리하고 사용하기 쉽게 만드는 데 필요한 개념(abstraction) 들을 정의한다.
    - 추상화를 통해 큰 프로그램을 이해하기 쉬운 작은 부분들로 나누어 구현할 수 있다.
  - (2) 설계와 구현에 중요한 목표: 성능. 오버헤드를 최소화한다.
  - (3) 응용 프로그램 간의 보호, 운영체제와 응용 프로그램 간의 보호
    - 운영체제의 원칙 중 고립(isolation) 원칙의 핵심
  - (4) 신뢰성(reliability)을 제공해야 한다.
    - 운영체제는 계속 실행되어야 한다.
    - 운영체제가 실패하면 그 위에서 실행되는 모든 응용 프로그램도 실패하게 되므로, 운영체제는 높은 수준의 신뢰성을 제공해야 한다.
