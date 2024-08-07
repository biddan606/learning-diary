# ReentrantLock

`ReentrantLock` 은 이름에서 볼 수 있듯이 재진입이 가능한 락입니다. 락을 소유한 스레드가 다시 락을 획득하려는 경우, 성공적으로 획득합니다.   
이외에도 공정성, 시간초과 발생 등의 기능을 사용할 수 있습니다. 예제를 통해 알아보겠습니다.

## 재진입(Reentrancy)

락을 획득한 상태에서 다시 락을 획득하려고 하면 락 획득을 대기하는 큐에서 대기할 수 있습니다.   
락을 획득한 스레드가 추가로 락을 요구하지만, 반환되는 락이 없으므로 데드락이 발생합니다.   
`ReentrantLock` 은 이를 방지하고 락을 소유한 스레드가 재진입하는 경우, 성공적 락을 획득하고, 재진입 언락시 락을 반환하지 않습니다.   

### 재진입 안되는 예시

```java
public class NonReentrantLockExample {

    private final NonReentrantLock lock = new NonReentrantLock();
    private int sharedResource = 0;

    public static void main(String[] args) {
        NonReentrantLockExample example = new NonReentrantLockExample();
        Runnable task = () -> {
            try {
                example.increment();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Thread t1 = new Thread(task, "Thread-1");
        t1.start();
    }

    public void increment() throws InterruptedException {
        lock.lock();
        try {
            System.out.printf("%s가 increment() 메서드를 수행중입니다.%n", Thread.currentThread().getName(), sharedResource);

            sharedResource++;
            System.out.printf("%s가 sharedResource를 %d로 변경하였습니다.%n", Thread.currentThread().getName(), sharedResource);

            anotherMethod();
        } finally {
            lock.unlock();
        }
    }

    public void anotherMethod() throws InterruptedException {
        lock.lock();
        try {
            System.out.printf("%s가 anotherMethod() 메서드를 수행중입니다.%n", Thread.currentThread().getName(), sharedResource);
        } finally {
            lock.unlock();
        }
    }


    private static class NonReentrantLock {
        private boolean isLocked = false;

        public synchronized void lock() throws InterruptedException {
            while (isLocked) {
                wait();
            }
            isLocked = true;
        }

        public synchronized void unlock() {
            isLocked = false;
            notify();
        }
    }
}
```

![alt text](<image/ReentrantLock/재진입 안되는 예시 출력.png>)

"Thread-1가 anotherMethod() 메소드를 수행중입니다." 까지 출력하여 총 3줄이 출력될 것이라고 예상하지만 데드락으로 인해 2줄까지만 출력하고 `anotherMethod()` 메서드 안에서 대기중인 상태입니다.

### ReentrantLock을 이용한 재진입 예시

```java
public class ReentrantLockExample {

    private final ReentrantLock lock = new ReentrantLock();
    private int sharedResource = 0;

    public static void main(String[] args) {
        ReentrantLockExample example = new ReentrantLockExample();
        Runnable task = () -> {
            try {
                example.increment();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Thread t1 = new Thread(task, "Thread-1");
        t1.start();
    }

    public void increment() throws InterruptedException {
        lock.lock();
        try {
            System.out.printf("%s가 increment() 메서드를 수행중입니다.%n", Thread.currentThread().getName());

            sharedResource++;
            System.out.printf("%s가 sharedResource를 %d로 변경하였습니다.%n", Thread.currentThread().getName(), sharedResource);

            // anotherMethod를 호출하여 재진입 시도
            anotherMethod();
        } finally {
            lock.unlock();
        }
    }

    public void anotherMethod() throws InterruptedException {
        lock.lock();
        try {
            System.out.printf("%s가 anotherMethod() 메서드를 수행중입니다.%n", Thread.currentThread().getName());
        } finally {
            lock.unlock();
        }
    }
}
```

![alt text](<image/ReentrantLock/ReentrantLock 재진입 예시 출력.png>)

`ReentrantLock`을 이용하면 재진입을 할 수 있습니다.

## 공정성(Fairness)

`ReentrantLock`을 이용하면 공정성 락(Fair Lock)을 설정할 수 있습니다. FIFO 순서대로 스레드가 잠금을 획득합니다.   

### 공정성이 보장되지 않는 예제

기본적인 락은 공정성을 보장해주지 않고, 스레드의 수가 많아질수록 공정성이 보장되지 않을 확률이 높아집니다.(예: synchronized)
공정성이 보장되지 않는다면 기아 현상(starvation)이 일어날 수 있습니다.

```java
public class UnfairReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock(false); // 비공정한 락 설정
    private int sharedResource = 0;

    public static void main(String[] args) {
        UnfairReentrantLockExample example = new UnfairReentrantLockExample();
        Runnable task = example::increment;

        Thread[] threads = new Thread[100];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(task, "Thread-" + (i + 1));
            threads[i].start();
        }
    }

    public void increment() {
        lock.lock();
        try {
            sharedResource++;
            System.out.println(Thread.currentThread().getName() + " incremented to: " + sharedResource);
        } finally {
            lock.unlock();
        }
    }
}
```

![alt text](<image/ReentrantLock/공정성이 보장되는 않는 예시 출력.png>)

특정 구간에서 스레드의 순서가 뒤바뀌는 걸 볼 수 있습니다.

### 공정성 보장 예제

```java
public class FairReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock(true); // 공정한 락 설정
    private int sharedResource = 0;

    public static void main(String[] args) {
        FairReentrantLockExample example = new FairReentrantLockExample();
        Runnable task = example::increment;

        Thread[] threads = new Thread[100];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(task, "Thread-" + (i + 1));
            threads[i].start();
        }
    }

    public void increment() {
        lock.lock();
        try {
            sharedResource++;
            System.out.println(Thread.currentThread().getName() + " incremented to: " + sharedResource);
        } finally {
            lock.unlock();
        }
    }
}
```

![alt text](<image/ReentrantLock/공정성이 보장되는 예시 출력.png>)

모든 구간에서 스레드들이 대체로 순차적으로 진입하는 것을 관찰할 수 있습니다. 간혹 순서가 뒤바뀌는 지점이 있지만, 이는 후순위 스레드가 먼저 `lock`을 요청하는 경우로 보입니다. 그럼에도 전반적인 공정성은 유지되는 것으로 판단됩니다.

## 인터럽트

데드락 상태가 발생했을 때, 대기 중인 스레드를 강제로 풀어줘야 할 수 있습니다. 이 때 인터럽트 가능한 락을 사용하면 유용합니다.

### 인터럽트 불가능 락

```java
public class NonInterruptibleLockExample {

    private final Object lock = new Object();

    public static void main(String[] args) {
        NonInterruptibleLockExample example = new NonInterruptibleLockExample();
        Runnable task = example::increment;

        Thread t1 = new Thread(task, "Thread-1");
        Thread t2 = new Thread(task, "Thread-2");

        t1.start();
        t2.start();

        // t2가 대기 중일 때 인터럽트 발생 시뮬레이션
        try {
            Thread.sleep(100); // 잠시 대기하여 t1이 락을 잡도록 함
            t2.interrupt(); // t2를 인터럽트
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public void increment() {
        synchronized (lock) {
            try {
                System.out.println(Thread.currentThread().getName() + ": 락 진입");

                // 잠금 유지 시간 시뮬레이션
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + ": 인터럽트 발생");
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

![alt text](<image/ReentrantLock/인터럽트 불가능 락 예제 출력.png>)

위 예제는 데드락 상황은 아니지만, 대기 중인 Thread-2에 인터럽트를 발생시켜 대기 큐에서 나오는 것을 예상하는 예제입니다.   
하지만 Thread-2가 락을 획득하여 코드를 진행하기 전까지 인터럽트를 캐치하지 않습니다.

### 인터럽트 가능 락

```java
public class InterruptibleLockExample {

    private final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        InterruptibleLockExample example = new InterruptibleLockExample();
        Runnable task = example::increment;

        Thread t1 = new Thread(task, "Thread-1");
        Thread t2 = new Thread(task, "Thread-2");

        t1.start();
        t2.start();

        // t2가 대기 중일 때 인터럽트 발생 시뮬레이션
        try {
            Thread.sleep(100); // 잠시 대기하여 t1이 락을 잡도록 함
            t2.interrupt(); // t2를 인터럽트
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public void increment() {
        try {
            lock.lockInterruptibly();
            try {
                System.out.println(Thread.currentThread().getName() + ": 락 진입");

                // 잠금 유지 시간 시뮬레이션
                Thread.sleep(5000);
            } finally {
                lock.unlock();
            }
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + ": 인터럽트 발생");
            Thread.currentThread().interrupt();
        }
    }
}
```

![alt text](<image/ReentrantLock/인터럽트 발생 예제 출력.png>)

`ReentrantLock` 은 락을 획득하지 않아도 인터럽트를 캐치할 수 있습니다.

## tryLock

tryLock 기능은 락을 얻기 위해 일정 시간 동안 대기를 하다가 일정 시간이 지나면 다른 작업을 수행합니다. 쉽게 타임아웃 기능이라고 볼 수 있습니다.

```java
public class TryLockExample {
    private final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        TryLockExample example = new TryLockExample();
        Runnable task = example::incrementWithTryLock;

        Thread t1 = new Thread(task, "Thread-1");
        Thread t2 = new Thread(task, "Thread-2");

        t1.start();
        t2.start();
    }

    public void incrementWithTryLock() {
        boolean isLockAcquired = false;
        try {
            // 2초 동안 잠금을 시도합니다.
            isLockAcquired = lock.tryLock(2, TimeUnit.SECONDS);
            if (isLockAcquired) {
                System.out.println(Thread.currentThread().getName() + ": 락을 획득했습니다.");
                // 잠금 유지 시간 시뮬레이션
                Thread.sleep(3000);
            } else {
                System.out.println(Thread.currentThread().getName() + ": 락을 얻지 못했습니다.");
            }
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + ": 인터럽트가 발생했습니다.");
        } finally {
            if (isLockAcquired) {
                lock.unlock();
            }
        }
    }
}
```

![alt text](<image/ReentrantLock/tryLock 예제 출력.png>)

## 이외 기능

- getHoldCount()
- getQueueLength()
- hasQueuedThread(Thread)
- hasQueuedThreads()
- isFair()
- isHeldByCurrentThread()
- isLocked()

## 결론

- `ReentrantLock` 의 Reentrant란 이름이 `sychronized` 와 비교하여 재진입이 가능한 줄 알았지만, 그것은 아니었다. 일반 락과 비교하여 재진입이 가능한 것
- `ReentrantLock` 은 고급 락으로 다양한 기능(공정성, 인터럽트, 타임아웃 등)이 있다.
- `ReentrantLock` 설명 주석에 공정성과 같은 기능을 사용하면 일반 락에 비해 성능이 저하될 수 있다고 한다.
- `ReentrantLock` 는 직접 락을 획득하고 반환하는 코드가 들어가므로, try-finally 구조로 코딩하는 것을 추천한다. 하지만 이 구조는 코드의 가독성을 저하시킨다. 다양한 기능이 필요없다면 `sychronized`이 좋다. 
- 다양한 기능이 필요한 락을 사용할 때는 `ReentrantLock`을 사용하고, put, get과 같은 단순한 구조에서는 `sychronized`를 하자

## 참조

[Java Lock - Jakob Jenkov](https://www.youtube.com/watch?v=MWlqrLiscjQ&t=1273s)
