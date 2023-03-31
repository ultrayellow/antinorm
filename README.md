# Antinorm

Norm을 우회하는 42가지 방법.

## 0. [코드 임베딩](01-code-embeding/README.md)

> 코드 안에 바이너리를 직접 삽입한다면?

### 평가 (높을수록 좋음)

| 항목 | 점수 |
| - | - |
| 이식성               | :last_quarter_moon: :new_moon: :new_moon: :new_moon: :new_moon:   |
| 개발 난이도           | :last_quarter_moon: :new_moon: :new_moon: :new_moon: :new_moon:   |
| 사용 난이도           | :last_quarter_moon: :new_moon: :new_moon: :new_moon: :new_moon:   |
| 응용 가능성           | :full_moon: :full_moon: :full_moon: :full_moon: :full_moon:       |
| 인간 Norm 통과 가능성  | :full_moon: :full_moon: :last_quarter_moon: :new_moon: :new_moon: |

18/50 점.

### 총평

#### 장점

- 배열에 크기 제한이 없다는 점 때문에 무려 함수 1개 / 파일 1개의 `Minishell` 도 만들 수 있다는 무궁무진한 가능성이 있다.
- c89수준의 문법만으로도 가능하다.

#### 단점

- 이식성이 매우 떨어진다.
- 개발 난이도가 매우 높다.
- 일반적이지 않은 컴파일 플래그가 들어간다.
- `const`가 표시된 배열의 값을 바꾸는 부분이 있다.
