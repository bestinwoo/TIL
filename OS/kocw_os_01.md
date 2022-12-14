# 1강

**이화여대 반효경 교수님의 운영체제 강의를 듣고 정리한 내용입니다.**

좁은 의미의 운영체제 : 커널

- 운영체제의 핵심 부분으로 메모리에 상주하는 부분

넓은 의미 운영체제

- 커널 뿐 아니라 각종 주변 시스템 유틸리티를 포함하는 개념

운영체제의 목적 : 컴퓨터 시스템의 자원을 효율적으로 관리하고 컴퓨터 시스템을 편리하게 사용할 수 있도록 도움

- 프로세서, 기억장치, 입출력 장치 (하드웨어 자원)등의 효율적 관리
- 사용자 및 운영체제 자신의 보호
- 프로세스, 파일, 메시지 등 (소프트웨어 자원)을 관리

## 운영체제의 분류

**동시 작업 가능 여부**

- 단일 작업(single tasking)
    - 한번에 하나의 작업만 처리
    - MS-DOS 프롬프트 상에는 한 명령의 수행을 끝내기 전에 다른 명령을 수행시킬 수 없음
- 다중 작업(multi tasking)
    - 동시에 두 개 이상의 작업 처리
    - 예) UNIX, MS Windows 등에서는 한 명령의 수행이 끝나기 전에 다른 명령이나 프로그램을 수행할 수 있음

**사용자의 수**

- 단일 사용자
    - MS-DOS, MS Windows
- 다중 사용자
    - UNIX, NT server

**처리 방식**

- 일괄 처리(batch processing)
    - 작업 요청의 일정량 모아서 한번에 처리
    - 작업이 완전 종료될 때까지 기다려야 함
    - 예) 초기 Punch Card 처리 시스템
    - 현대에는 찾아보기 힘듦
- 시분할(time sharing)
    - 여러 작업을 수행할 때 컴퓨터 처리 능력을 일정한 시간 단위로 분할하여 사용
    - 일괄 처리 시스템에 비해 짧은 응답시간을 가짐
    - interactive한 방식
    - UNIX 등 현대의 운영체제에서 사용
- 실시간(Realtime OS)
    - 정해진 시간 안에 어떤 일이 반드시 종료됨이 보장되어야 하는 실시간 시스템을 위한 OS
    - 예) 원자로/공장 제어, 미사일 제어, 반도체 장비, 로보트 제어
    - 실시간 시스템의 개념 확장
        - Hard realtime system(경성 실시간 시스템)
        - Soft realtime system(연성 실시간 시스템) (영화,음악 재생)

### 몇 가지 용어

- Multitasking
- Multiprogramming
- Time sharing
- Multiprocess
- 구분
    - Multiprogramming은 여러 프로그램이 메모리에 올라가 있음을 강조
    - Time Sharing은 CPU의 시간을 분할하여 나누어 쓴다는 의미를 강조
    - Multiprocessor : 하나의 컴퓨터에 CPU가 여러 개 붙어 있음을 의미

## 운영체제의 예

- 유닉스
    - 코드의 대부분을 C언어로 작성
    - 높은 이식성
    - 최소한의 커널 구조
    - 복잡한 시스템에 맞게 확장 용이
    - 오픈 소스
    - 프로그램 개발에 용이
    - 다양한 버전
- DOS
    - 단일 사용자용 운영체제
- MS Windows
    - 다중 작업용 GUI 기반 운영 체제
    - Plug and Play, 네트워크 환경 강화
    - DOS용 응용 프로그램과 호환성 제공
    - 풍부한 지원 소프트웨어
- Handheld device를 위한 OS
    - PalmOS, Pocket PC (WinCE), Tiny OS

## 운영 체제의 구조

CPU(CPU 스케줄링) ↔ memory(메모리 관리) ↔ Disk(파일 관리) & I/O device(입출력 관리)

![Untitled](https://user-images.githubusercontent.com/33615669/199058243-9b018fbc-d7a5-4fe0-bc50-efa2a60d8580.png)
