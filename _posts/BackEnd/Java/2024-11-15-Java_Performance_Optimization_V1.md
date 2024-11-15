---
layout: post
title: "자바 성능 최적화 시리즈 1. Thread 문제 확인과 Deadlock 탐지"
tags: [BackEnd, Java, Performance]
---

# Intro
안녕하세요 Noah입니다.

오늘부터는 JAVA의 성능을 최적화하기 위한 방법들에 대해 다뤄보겠습니다.<br/>
특히, 스레드 문제 확인과 Deadlock 탐지 방법을 중심으로, 실습과 함께 자세히 설명드릴 예정입니다.<br/>
Java 애플리케이션을 운영하다 보면 스레드 관련 문제가 성능 저하의 주요 원인으로 작용하는 경우가 많은데요, <br/>
이를 효과적으로 분석하고 해결하는 방법을 단계별로 알아보겠습니다.

그럼, 시작해보겠습니다!
<br/><br/><br/><br/>



# 목차
<br/><br/><br/><br/>


# 본문
## 1. Thread 문제 확인하는 방법
Java에서 스레드 문제를 확인하는 방법은 크게 두 가지로 나뉩니다

1. Thread Dump 분석을 통한 확인
2. 프로파일링 도구를 통한 실시간 모니터링

아래에서 각 방법을 설명드리겠습니다.

### 1-1. Thread Dump 분석
Thread Dump는 Java 애플리케이션의 스레드 상태를 캡처하여, 각 스레드가 어떤 상태에 있고 어떤 작업을 수행 중인지 확인할 수 있는 방식입니다. 
주로 **Deadlock**, **Long-running Threads**와 같은 문제를 파악하는 데 유용합니다.

1. Thread Dump 생성 방법
   - **jstack** 명령어 사용<br/>
      jstack은 JDK에 포함된 도구로, Java 프로세스의 Thread Dump를 캡처합니다.<br/>
      ```bash
      jstack <PID>
      ```
      `<PID>`에는 Java 프로세스의 ID를 입력합니다.<br/>
      `ps aux | grep java` 명령어로 Java 프로세스를 찾아서 사용할 수 있습니다.
   - **JVisualVM** 사용<br/>
      GUI 기반의 도구로, Thread Dump를 쉽게 확인할 수 있으며 스레드 상태를 시각적으로 모니터링할 수 있습니다.
2. Thread Dump 분석 방법
   Thread Dump 파일에는 각 스레드의 상태가 나타나며, 주요 상태는 다음과 같습니다. 
   
   1. RUNNABLE: 실행 중이거나 CPU 점유 중인 상태입니다.
   2. BLOCKED: 다른 스레드가 잠금을 점유하고 있어 기다리는 상태입니다.
   3. WAITING: 특정 조건이 만족될 때까지 기다리는 상태입니다.
   4. TIMED_WAITING: 주어진 시간 동안만 기다리는 상태입니다.
   
   이 정보를 통해 스레드가 어떤 리소스를 기다리거나, 멈춰있는지 확인할 수 있습니다.<br/>
   특히 **BLOCKED** 상태의 스레드가 많다면 동기화 문제로 인한 병목 현상을 의심할 수 있습니다.
   <br/><br/>

### 1-2. 프로파일링 도구를 통한 실시간 모니터링
실시간 모니터링 도구는 스레드의 상태를 계속해서 추적하며 병목 현상이나 Deadlock을 빠르게 발견하는 데 유용합니다.

**[자주 사용되는 도구 종류]**
1. JVisualVM: Java 애플리케이션의 CPU 및 메모리 사용량뿐 아니라 스레드의 상태를 실시간으로 모니터링할 수 있습니다.
2. Java Mission Control (JMC): 고성능 모니터링 도구로, Java 애플리케이션에서 실행되는 스레드의 상태를 실시간으로 분석할 수 있습니다.
3. YourKit Java Profiler: 다양한 분석 도구가 포함된 상용 프로파일러로, 스레드 사용 패턴 및 메모리 누수, 스레드 경쟁 상태 등의 문제를 감지할 수 있습니다.

**[프로파일링 도구 사용 방법 (JVisualVM 예시)]**
1. Java Process 연결: JVisualVM을 실행하고 모니터링할 Java 프로세스를 선택합니다.
2. Threads 탭 선택: 각 스레드의 상태를 그래프로 확인할 수 있으며, 특정 스레드가 장기간 BLOCKED 상태에 있는지 등을 확인할 수 있습니다.
3. Thread Dump 생성 및 분석: 문제가 되는 시점에서 Thread Dump를 생성하여 각 스레드의 상태를 분석합니다.
   <br/><br/>

### 1-3. Deadlock 탐지
Deadlock은 서로 다른 스레드가 서로가 점유한 리소스를 기다리는 상황에서 발생합니다. 이를 확인하는 방법은 다음과 같습니다.

1. Thread Dump 분석: Thread Dump에서 "Found one Java-level deadlock" 메시지가 나타난다면 Deadlock이 발생한 상태입니다.
2. JVisualVM 및 JMC 사용: 두 도구 모두 Deadlock을 탐지할 수 있으며, 이를 시각적으로 보여줍니다.

#### 1-3-1. Thread Dump 실습
이제 Thread Dump를 생성하고 간단한 예제를 통해 특정 스레드가 BLOCKED 상태에 있는 상황을 일부러 발생시켜 보겠습니다.

```java
public class ThreadBlockedExample {
    public static void main(String[] args) {
        Object lock = new Object();
        Thread thread1 = new Thread(() -> {
            synchronized (lock) {
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread thread2 = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Thread 2");
            }
        });
        thread1.start();
        thread2.start();
    }
}
```

위 코드를 실행하면 `thread2`가 `lock`을 기다리는 상태에서 BLOCKED 상태에 머물게 됩니다.
이 상태를 Thread Dump를 통해 확인해보겠습니다.

1. Thread Dump 생성
   ```bash
    # PID 확인
    ps aux | grep java
   
    # Thread Dump 생성
    jstack <PID>
   ```
   TODO - 위 코드를 실행한 이미지 추가
2. Thread Dump 분석 
   - Thread Dump에서 `thread2`의 상태 확인
   - `thread2`가 BLOCKED 상태에 있는 것을 확인할 수 있습니다.
   - `thread1`이 `lock`을 10초 동안 점유하고 있기 때문에 `thread2`는 `lock`을 얻을 수 없어 BLOCKED 상태에 머물게 됩니다. 
   - 이러한 상황을 해결하기 위해서는 `thread1`이 `lock`을 빨리 해제하도록 수정해야 합니다.
   TODO - Thread Dump 이미지 추가

이처럼 Thread Dump를 통해 스레드의 상태를 확인하고, 문제가 되는 상황을 파악하여 해결할 수 있습니다.
<br/><br/>

#### 1-3-2. JVisualVM 및 JMC 사용 실습
JVisualVM과 JMC(Java Mission Control)는 Java 애플리케이션의 스레드 상태를 실시간으로 관찰하고, Deadlock과 같은 문제를 시각적으로 탐지할 수 있는 도구입니다.
아래에서는 두 도구를 활용하여 Deadlock을 탐지하는 실습을 진행해 보겠습니다.

##### 1-3-2-1. JVisualVM을 사용한 Deadlock 탐지
**준비 사항**

- JVisualVM 설치: OpenJDK와 함께 제공되거나 JVisualVM 공식 사이트에서 다운로드할 수 있습니다.
- 위의 `ThreadBlockedExample` 코드 실행 준비.

**실습 과정**
1. **JVisualVM 실행 및 프로세스 선택**
   - JVisualVM을 실행합니다.
   - 좌측 패널에서 실행 중인 Java 애플리케이션(`ThreadBlockedExample`)을 선택합니다.
2. **Thread 상태 관찰**
   - `Threads` 탭을 클릭합니다.
   - 현재 실행 중인 모든 스레드의 상태를 확인할 수 있습니다.
   - Deadlock이 발생하면, JVisualVM에서 BLOCKED 상태의 스레드가 시각적으로 표시됩니다.
3. **Thread Dump 생성**
   - `Thread Dump` 버튼을 클릭하여 Thread Dump를 생성합니다.
   - 생성된 덤프에서 Deadlock 메시지를 확인합니다.
   - 메시지 예<br/>
       ```vbnet
       Found one Java-level deadlock:
       =============================
       "Thread-2":
         waiting to lock <0x00000000> (a java.lang.Object),
         which is held by "Thread-1"
       "Thread-1":
         waiting to lock <0x00000001> (a java.lang.Object),
         which is held by "Thread-2"
       ```
4. **분석 결과**
   - `Thread-1`과 `Thread-2`가 서로 잠금(lock)을 기다리며 Deadlock 상태에 빠졌음을 확인할 수 있습니다.

##### 1-3-2-2. JMC(Java Mission Control)를 사용한 Deadlock 탐지
**준비 사항**
- JDK 설치 시 포함된 JMC 사용 가능 (Oracle JDK 또는 OpenJDK).
- `jcmd` 명령어 사용을 위한 Java 설치.

**실습 과정**
1. **JMC 실행 및 연결**
   - JMC를 실행한 뒤, 좌측 상단에서 실행 중인 Java 프로세스(`ThreadBlockedExample`)를 선택하여 연결합니다.
2. **Thread 상태 확인**
   - `Threads` 탭을 선택하여 모든 스레드의 상태를 확인합니다.
   - BLOCKED 상태의 스레드가 있는지 모니터링합니다.
3. **Flight Recorder 활성화**
   - JMC의 `Flight Recorder` 기능을 사용하여 애플리케이션의 성능 데이터를 기록합니다.
   - 기록된 데이터를 통해 Deadlock 및 기타 병목 현상을 분석합니다.
4. **Deadlock 탐지**
   - `Thread Dump`를 생성하여 Deadlock 메시지를 확인합니다.
   - GUI를 통해 Deadlock 상태에 빠진 스레드와 점유 중인 리소스를 시각적으로 탐색할 수 있습니다.
   TODO - JMC 활용 이미지 추가

##### 1-3-2-3. 결과 분석 및 문제 해결
- JVisualVM과 JMC 모두 Deadlock 상태를 탐지하고 상세히 분석할 수 있는 강력한 도구입니다.
- 문제를 해결하려면 다음과 같은 접근법을 고려합니다:
   - **잠금 순서 재설계**: `synchronized` 블록이 서로 교차하지 않도록 순서를 정리합니다.
   - **ReentrantLock 사용**: `tryLock` 메서드로 Deadlock을 방지합니다.
   - **스레드 작업 분리**: 리소스를 독립적으로 처리하도록 설계합니다.

TODO - 해결과정 이미지 첨부
<br/><br/><br/><br/>

# Outro
오늘은 Java 애플리케이션의 성능 최적화를 위한 스레드 문제 분석 및 Deadlock 탐지 방법을 살펴보았습니다.<br/>
Thread Dump 생성부터 프로파일링 도구 사용, 그리고 Deadlock 해결 방법까지, 실무에서 바로 활용할 수 있는 내용을 중심으로 설명드렸는데요,<br/> 
이를 통해 Java 애플리케이션의 성능 문제를 보다 효과적으로 해결할 수 있을 거에요. 추가해야될 테스트 이미지들은 천천히 캡처해서 올려두겠습니다. 먼저 활용할 분들은 활용해보세요~

다음에는 MultiLock에 대해 공부해볼 예정입니다. 최적화 주제와 실습 사례를 다룰 예정이니 많은 기대 부탁드립니다.

긴 글 읽어주셔서 감사합니다! 😊
