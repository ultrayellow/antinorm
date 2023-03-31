# 01. Code embeding

```shell
% norminette ./push_swap.c
./push_swap.c: OK!

% make
clang -Wall -Wextra -Werror -Wl,-segprot -Wl,__TEXT -Wl,rwx -Wl,rwx -o push_swap push_swap.c

% ./push_swap 3 2 1
ra
sa
```

## 빌드 및 실행

```shell
% make all
```

결과로 나오는 프로그램은 일반적인 `push_swap`프로그램이다.

정상 작동을 확인한 환경
- M1 MacBook Air (13.1) with Rosetta2

## 원리

CPU는 데이터 영역에 직접 하드코딩한 명령어와 컴파일 과정을 거쳐서 완성된 명령어를 구별할 수 없다.

그러므로 `Norm`을 준수하지 않는 코드를 컴파일한 결과를 `const char[]` 형태로 `C` 소스코드에 삽입하고,

`main`함수에서 그 배열 속의 적당한 함수 위치로 점프를 하면, 문제 없이 실행된다.

## 사전 지식

- 기초적인 어셈블리
- 기초적인 macho 포맷에 대한 이해
- 정적 링킹, 동적 링킹
- otool, objdump, nm등 도구 사용법
- GOT (global offset table), PIE (position independent executable)

## 만드는법

1) `C`로 `Norm`을 준수하지 않지만 잘 작동하는 코드를 작성한 이후, 빌드하여 동적 라이브러리로 만든다.

2) 그 라이브러리의 2진 값 자체를 `C`에서 인식할 수 있는 형태로 변환한다.

3) `otool`, `objdump`, `nm` 등을 이용하여 원하는 함수와 GOT의 오프셋을 구한다.

4) 만약 프로그램에서 사용하는 외부 함수(`write`, `read`, `exit` ...)가 있다면 `main`함수 초반 부분에 GOT를 알맞게 설정한다.

5) 원하는 함수를 호출해 흐름을 그쪽으로 넘긴다.

## 까다로운 부분들

배열에 하드코딩한 명령들은 최종적으로 `main` 과 링킹되지 않는다.

그렇기 때문에 배열 안에 명령 중 절대 주소를 사용하는 심볼들은 끝까지 UNDEFINED상태로 남을 것이고, 결국 사용할 수 없게 된다.

예를 들어서 단순히 42를 반환하는 함수가 있다고 했을 때, 이 함수는 다음과 같이 구현해볼 수 있다.

```asm
mov rax, 42
ret
```

이런 함수는 참조하는 외부 심볼이 없기 때문에 최종 링크 과정을 거치지 않아도 문제 없이 작동한다.

하지만 다음과 같이 외부 심볼을 사용하는 코드를 생각해 본다면,

```asm
.section data
global ft

ft:
	dw 42

.section text

test:
	mov rax, [ft]
	ret
```

절대 주소를 사용하기 때문에 링킹 과정이 완전히 끝나기 전까지 ft의 주소를 알 수 없다.
만약 저런 코드를 배열에 넣고 실행한다면 `[ft]` 부분에서 높은 확률로 `SIGSEGV`를 받게 될것이다.

이런 일을 피하기 위해서 모든 심볼을 다음과 같이 상대 위치로 지정해야 한다.

```asm
.section data
global ft

ft:
	dw 42

.section text

test:
mov rax, [rel ft]
ret
```

이렇게 한다면 최종 링크단계까지 가지 않아도 `ft`의 위치를 `rip + x`로 결정할 수 있게 되므로 `SIGSEGV`없이 `ft`를 참조할 수 있게 될 것이다.

이 방법으로 사용자 정의 심볼을 참조할 수 있게 되었지만, 아직 한가지 큰 문제가 남았다.

`Mac OS`의 특성상 `libc `(libsystem)의 정적 라이브러리 버전이 존재하지 않기 때문에, 프로그램을 정상적으로 작동시키고, 표준 함수 / 시스템 콜 래퍼를 사용하기 위해서는 동적 라이브러리를 사용할 수 있어야 한다.

먼저 `Mac OS`에서는 어떤 식으로 동적 링킹된 심볼의 주소를 가져오는지 확인해야한다.
`lldb`를 이용해서 간단한 `hello world`프로그램을 디버깅해보니, Mac에서는 마치 `RELRO` 기능이 활성화된 Linux처럼 모든 심볼의 주소가 프로그램 초기화 과정(`main`함수 이전)에 구해지고, GOT에 써지는 것을 볼 수 있고, 함수 호출 흐름은 `call (some fixed address)` -> `jmp [rip + x]`(GOT entry) -> `libsystem.printf`과 같이 이루어지는 것을 볼 수 있었다.

따라서 프로그램이 켜진 직후 원본 `GOT`의 값을 배열의 `GOT`로 복사하면 배열쪽으로 흐름이 넘어간 이후에도 문제 없이 외부 함수를 사용할 수 있다. 이때 Mac에서 기본적으로 `ASLR`기능이 활성화 되어있기 때문에 원본 쪽의 GOT값을 가져올때 신경을 써야한다. 또한 `GOT`내의 순서도 매우 중요하게 작용한다.

마지막으로 코드가 들어갈 배열은 RWX 권한이 모두 있어야 하기 때문에 링킹 과정에서 권한 설정을 잘 해줘야 한다.
(`-Wl,-segprot -Wl,__TEXT -Wl,rwx -Wl,rwx`)
