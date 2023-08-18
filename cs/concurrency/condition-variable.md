# Condition Variable

동시성 프로그램에서, 다음과 같이 어떤 조건이 만족되기 기다린 다음 작업을 수행하는 패턴은 흔하게 나타난다. 코드로 나타내면 다음과 같다.

```python
while not condition:
    wait()
do_something()
```

위에서 나온 코드를 그대로 구현하면 busy-waiting을 하는 것과 다를 바가 없기 때문에, CPU cycle을 낭비하게 된다. Condition variable은 이런 상황에서 사용되는 synchronization primitive이다.

## Definition

Condition variable은 어떤 조건이 만족되기를 기다리는 스레드를 저장하는 큐를 가지고, 두 가지 operation이 가능하다.

- `wait`: 큐에 조건을 만족하기를 기다리는 스레드를 추가한다
- `signal`: 조건이 만족되어 큐에 있는 스레드 중 하나를 깨운다.

## Using locks

POSIX threads, PintOS 등에서 `wait`, `signal`은 lock(혹은 mutex)을 필요로 하고, operation을 하는 스레드가 lock을 가지고 있음을 가정한다. `wait` operation 중에 이 lock은 해제되고, lock을 원래 갖고 있던 스레드는 sleep 상태가 되었다가, 다른 스레드가 `signal` operation을 수행했을 때 깨어난 다음 다시 lock을 가져간다.

Lock을 사용할 필요 없이 스레드를 sleep시키는 것으로 충분할 것 같지만, 그렇게 할 경우 race condition이 발생할 가능성이 있다. 다음 코드를 생각해 보자.

```python
cond = CondVar()

def child():
    do_work()
    print("child: Work done.")
    cond.signal()

print("parent: Started.")
spawn_thread(child)
cond.wait()
print("parent: Finished.")
```

앞서 얘기한 것처럼 lock 없이 condition variable을 구현했다고 해 보자. 두 가지 경우를 생각해 볼 수 있을 것이다.

### `wait` :arrow_right: `signal`

직관적으로도 말이 되고, 별 문제 없이 동작하는 상황이다. `child`에서 `signal`을 호출하는 시점에서, condition variable의 큐에는 parent thread가 잠자면서 줄을 서 있을 것이다. `signal`에서는 큐에서 잠자고 있던 parent thread를 발견하고, pop 한 다음 깨우고 종료한다.

### `signal` :arrow_right: `wait`

직관적으로는 말이 되지 않지만 충분히 일어날 수 있는 상황이다. `child`에서 `signal`을 호출하는 시점에서, condition variable의 큐에는 아무것도 없을 것이다. `signal`은 아무것도 하지 않고 종료하고, parent thread는 `wait`에 걸려서 영원한 숙면을 취한다.

앞서 설명한 POSIX threads, PintOS에서와 같이 condition variable에서 lock을 사용한다면 다음과 같은 코드를 작성할 것이다.

```python
cond = CondVar()
lock = Lock()

def child():
    do_work()
    print("child: Work done.")
    lock.acquire()
    cond.signal(lock)

print("parent: Started.")
lock.acquire()
spawn_thread(child)
cond.wait(lock)
print("parent: Finished.")
```

위와 같은 코드에서는 parent thread가 `wait`을 실행하기 전까지 `child`는 `lock`을 가져가지 못하기 때문에, `signal`은 언제나 `wait` 다음 실행되고, 앞서 설명한 race condition이 일어나지 않는다.