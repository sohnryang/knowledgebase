# 어셈블리어에서 반복문 사용하기

다음과 같은 C언어 코드를 생각해 보자.

```c
for (int i = 0; i < 10; ++i) {
  // do something
}
// code to be run after loop
```

위 코드의 `for` 루프는 `while` 루프와 `if`문의 조합으로 바꿀 수 있다.

```c
int i = 0;
while (1) {
  if (i >= 10) break;
  // do something
}
// code to be run after loop
```

어셈블리어에서는 `if`문 대신 `cmp` 명령어와 Jcc 계열 명령어로 조건 분기를 하게 된다. 조건이 참이면 내부를 실행하는 `if`문과는 달리, `cmp` 명령어가 CPU의 플래그를 세팅하고, Jcc 계열 명령어는 플래그에 따라 점프 여부를 결정한다.

| 명령어 | 확인하는 플래그       | C언어 번역    |
| ------ | --------------------- | ------------- |
| `jmp`  | 없음 (무조건 점프)    | `if (true)`   |
| `je`   | `ZF == 1`             | `if (a == b)` |
| `jne`  | `ZF == 0`             | `if (a != b)` |
| `jl`   | `SF != OF`            | `if (a < b)`  |
| `jle`  | `ZF == 1 || SF != OF` | `if (a <= b)` |
| `jg`   | `ZF == 0 && SF == OF` | `if (a > b)`  |
| `jge`  | `SF == OF`            | `if (a >= b)` |

이들을 이용하여 위 코드를 어셈블리어로 나타내면 다음과 같다.

```assembly
1:
	cmpl %eax, $10
	jge 2f
	# do something
	jmp 1b
2:
	# code to be run after loop
```

여기서 `1:`, `2:` 는 로컬 레이블이고, 심볼 대신에 임시로 이름을 사용하게 해 준다. `2f`와 같이 쓰면 `2f`보다 뒤에 있는 것 중 가장 앞에 있는 `2:`의 주소를 의미하고, `1b`와 같이 쓰면 `1b` 앞에 있는 것 중 가장 뒤에 있는 `1:`의 주소를 의미한다.