# Code Coverage

이전 장에선 검사할 프로그램에 무작위 입력을 생성하는 기본 fuzzing을 소개했다. 이러한 검사의 효율성을 어떻게 특정할까? 한가지 방법은 찾아낸 버그의 수(와 심각성)으로 검사하는 것이지만; 만약 버그가 적다면, 버그를 발견하기 위한 테스트 가능성에 대한 대체물이 필요하다. 이 장에선, 검사 실행동안 실제로 실행된 프로그램의 부분을 측정하는 code coverage의 개념을 소개한다.  이러한 coverage를 측정하는 것은 가능한 한 많은 코드를 커버하려고 시도하는 테스트 생성기에도 중요하다.

```python
from bookutils import YouTubeVideo
YouTubeVideo('8HxW8j9287A')
```

[![8HxW8j9287A](https://img.youtube.com/vi/8HxW8j9287A/0.jpg)](https://www.youtube.com/watch?v=8HxW8j9287A)

#### 준비물

- 어떻게 프로그램이 실행되는지에 대한 이해가 필요함
- [이전 장](/Part2/Fuzzing,%20Breaking%20Things%20with%20Random%20Inputs.md)에서 기본 fuzzing에 대해 배웠어야 한다.

## 개요

이전 장의 코드를 사용하기 위해, 쓰자

```python
>>> from fuzzingbook.Coverage import <identifier>
```

그런 다음 다음 기능을 사용할 수 있다.

이 장에선 파이썬 프로그램을 위해 커버리지를 측정할 수 있도록 해주는 Coverage 클래스를 소개한다. 이 책의 문맥 내에서, 발견되지 않은 영역으로 fuzzing을 인도하기 위해 커버리지 정보를 사용한다.

Coverage 클래스의 일반적인 사용법은 with 클래스와 결합하는 것이다.

```python
>>> with Coverage() as cov:
>>>    cgi_decode("a+b")
```

커버리지 객체를 출력하면 커버된 함수를 보여주며 커버되지 않은 줄은 접두사 #이 붙는다.

```python
>>> print(cov)
#  1  def cgi_decode(s: str) -> str:
#  2      """Decode the CGI-encoded string `s`:
#  3         * replace '+' by ' '
#  4         * replace "%xx" by the character with hex number xx.
#  5         Return the decoded string.  Raise `ValueError` for invalid inputs."""
#  6  
#  7      # Mapping of hex digits to their integer values
   8      hex_values = {
   9          '0': 0, '1': 1, '2': 2, '3': 3, '4': 4,
  10          '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
  11          'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15,
  12          'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15,
# 13      }
# 14  
  15      t = ""
  16      i = 0
  17      while i < len(s):
  18          c = s[i]
  19          if c == '+':
  20              t += ' '
  21          elif c == '%':
# 22              digit_high, digit_low = s[i + 1], s[i + 2]
# 23              i += 2
# 24              if digit_high in hex_values and digit_low in hex_values:
# 25                  v = hex_values[digit_high] * 16 + hex_values[digit_low]
# 26                  t += chr(v)
# 27              else:
# 28                  raise ValueError("Invalid encoding")
# 29          else:
  30              t += c
  31          i += 1
  32      return t
```

trace() 메소드는 trace를 return한다. 이 trace는 순서대로 실행된 영역의 리스트이다. 각 영역은 짝(fuction name, line)이 지어져있다.

```python
>>> cov.trace()
[('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 8),
 ('cgi_decode', 10),
 ('cgi_decode', 8),
 ('cgi_decode', 10),
 ('cgi_decode', 8),
 ('cgi_decode', 10),
 ('cgi_decode', 8),
 ('cgi_decode', 10),
 ('cgi_decode', 8),
 ('cgi_decode', 10),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 11),
 ('cgi_decode', 8),
 ('cgi_decode', 12),
 ('cgi_decode', 8),
 ('cgi_decode', 12),
 ('cgi_decode', 8),
 ('cgi_decode', 15),
 ('cgi_decode', 16),
 ('cgi_decode', 17),
 ('cgi_decode', 18),
 ('cgi_decode', 19),
 ('cgi_decode', 21),
 ('cgi_decode', 30),
 ('cgi_decode', 31),
 ('cgi_decode', 17),
 ('cgi_decode', 18),
 ('cgi_decode', 19),
 ('cgi_decode', 20),
 ('cgi_decode', 31),
 ('cgi_decode', 17),
 ('cgi_decode', 18),
 ('cgi_decode', 19),
 ('cgi_decode', 21),
 ('cgi_decode', 30),
 ('cgi_decode', 31),
 ('cgi_decode', 17),
 ('cgi_decode', 32)]
 ```

 coverage() 메소드는 coverage를 return한다. 이 coverage는 trace내에서 최소한 한 번 이상 실행된 영역의 집합이다:

 ```python
 >>> cov.coverage()
{('cgi_decode', 8),
 ('cgi_decode', 9),
 ('cgi_decode', 10),
 ('cgi_decode', 11),
 ('cgi_decode', 12),
 ('cgi_decode', 15),
 ('cgi_decode', 16),
 ('cgi_decode', 17),
 ('cgi_decode', 18),
 ('cgi_decode', 19),
 ('cgi_decode', 20),
 ('cgi_decode', 21),
 ('cgi_decode', 30),
 ('cgi_decode', 31),
 ('cgi_decode', 32)}
 ```

 Coverage 집합은 intersection(교차점)(여러 실행에 의해 커버되는 영역)과 차이(실행 a에선 커버되지만 실행 b에서는 커버되지 않는 영역)와 같은 집합 작업의 대상이 될 수 있다.

 이 장에서는 또한 C 프로그램으로부터 커버리지를 어떻게 얻는지 논의한다.

 ![image](https://github.com/fault2000/Fuzzing_book/assets/73513005/8aa32c85-b42b-4904-9570-7d5b4ed630c8)

```python
import bookutils.setup
```

## A CFI Decoder

CGI-encoded 문자열을 디코드하는 간단한 파이썬 함수를 소개함으로써 시작한다. CGI 인코딩은 URLs(예를 들어, 웹 주소)에서 사용되어 공백이나 특정 구두점같은 URL에서 유효하지 않은 문자를 인코드한다:

- 공백은 + 로 대체된다
- 다른 유효하지 않은 문자는 '%xx'로 대체되며 xx는 2자리의 16진수 동치이다.

CGI 인코딩에서, 문자열 "Hello, world!"는 이에 따라 "Hello%2c+world%21"가 되며 2c와 21은 16진수 동치어로 각각 ','와 '!'이다.

cgi_decode() 함수는 이러한 인코드된 문자열을 받아 원래 형태로 디코드해준다. 우리의 구현은 [[Pezzè et al, 2008](http://ix.cs.uoregon.edu/~michal/book/)]의 코드를 모방했다(이것은 자체 버그마저 포함하지만 - 지금 이 시점에선 밝히지 않을 것이다).

```python
def cgi_decode(s: str) -> str:
    """Decode the CGI-encoded string `s`:
       * replace '+' by ' '
       * replace "%xx" by the character with hex number xx.
       Return the decoded string.  Raise `ValueError` for invalid inputs."""

    # Mapping of hex digits to their integer values
    hex_values = {
        '0': 0, '1': 1, '2': 2, '3': 3, '4': 4,
        '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
        'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15,
        'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15,
    }

    t = ""
    i = 0
    while i < len(s):
        c = s[i]
        if c == '+':
            t += ' '
        elif c == '%':
            digit_high, digit_low = s[i + 1], s[i + 2]
            i += 2
            if digit_high in hex_values and digit_low in hex_values:
                v = hex_values[digit_high] * 16 + hex_values[digit_low]
                t += chr(v)
            else:
                raise ValueError("Invalid encoding")
        else:
            t += c
        i += 1
    return t
```

여기 cgi_decode가 어떻게 작동하는지에 대한 예시가 있다:

```
cgi_decode("Hello+world")
```

> 'Hello world'

cgi_decode를 체계적으로 검사하고 싶다면, 어떻게 진행해야 할까?

검사 보고서는 검사를 도출하는 2가지 방법으로 구별된다: Black-box 검사와 White-box 검사다.

## Black-Box Testing

블랙 박스 검사의 아이디어는 세부사항에서 테스트를 도출하는 것이다. 따라서 위의 경우 다음을 포함하여 지정되고 문서화된 기능으로 cgi_decode()를 검사해야 한다.'

- '+'의 정확한 교체 검사
- "%xx"의 정확한 교체 검사
- 다른 문자 비교체 검사
- 유효하지 않은 입력 인식 검사

이 4가지 기능을 커버하는 4개의 assertions(tests)가 있다. 우리는 그것들이 전부 통과되는 걸 볼 수 있다:

```python
assert cgi_decode('+') == ' '
assert cgi_decode('%20') == ' '
assert cgi_decode('abc') == 'abc'

try:
    cgi_decode('%?a')
    assert False
except ValueError:
    pass
```

블랙 박스 검사의 장점은 지정된 행동에서의 오류를 찾을 수 있다는 것이다. 이는 주어진 구현에 대해 독립적이고, 따라서 구현 전부터 검사를 만들어낼 수 있다. 단점은 보통 구현된 행동이 지정된 행동보다 더 많은 영역을 커버한다는 것이고, 따라서 사양서만을 기반으로 한 검사는 모든 구현된 세부사항을 커버하지 못한다.

## White-Box Testing

블랙 박스 검사와 대조적으로, white-box 검사는 구현, 특히 내부 구조에서 검사를 도출한다. 화이트 박스 검사는 코드의 구조적 기능을 커버하는 개념과 밀접하게 연관되어 있다. 예를 들어 만약 코드의 구문이 검사 도중 실행되지 않는다면, 이는 그 구문의 오류가 발생되지 못함을 의미한다. 따라서 화이트 박스 검사는 검사가 충분하다고 말할 수 있기 전에 충족해야 하는 여러 커버리지 기준을 소개한다. 가장 자주 사용되는 커버리지 기준은

- 구문 커버리지 - 코드 내의 각 구문은 적어도 하나의 테스트 입력에 의해 실행되어져야 한다.
- 분기 커버리지 - 코드 내의 각 분기는 적어도 하나의 테스트 입력에 의해 실행되어져야 한다.(이는 if와 while 결정이 한 번은 참이고, 한 번은 거짓일 경우로 해석된다)

이 외에도, 분기 실행의 연속 실행, 루프 반복 실행(zero, one, many), 변수 정의와 사용 간 데이터 흐름 등등 더 많은 커버리지 기준들이 있다. [[Pezzè et al, 2008](https://ix.cs.uoregon.edu/~michal/book/)]이 대단한 개요를 가지고 있다.

위의 cgi_decode를 코드 내의 각 구문이 적어도 한 번 실행되도록 해야 하는 이유를 생각해보자. 우리는 다음을 커버해야 한다

- if c == '+' 다음 블럭
- if c == '%' 다음 2 블럭(유효한 입력의 경우 하나, 유효하지 않은 입력 하나)
- 모든 다른 문자에 대한 마지막 else의 경우

이는 위 블랙 박스 검사로 한 같은 조건에서 결과를 낸다. 위 assertion은 실제로 코드 내의 모든 구문을 커버한다. 개발자가 서로 다른 코드 영역에 서로 다른 행동을 구현하는 경향이 있기 때문에 이러한 영역을 커버하는 것은 서로 다른(지정된) 행동을 커버하는 테스트 케이스로 이어지고 때문에 실제로는 매우 일반적이다.

화이트 박스 검사의 장점은 구현된 행동에서 오류를 찾는다는 것이다. 이는 설명서가 충분한 세부사항을 제공하지 못하는 겨웅에도 작동할 수 있다; 사실, 이는 설명서 내의 corner 케이스를 식별(및 지정)하는데 도움이 된다. 단점은 구현되지 않은 행동을 놓칠 수 있다는 것이다: 만약 몇몇 지정된 기능들이 없다면, 화이트 박스 검사는 그것을 찾아내지 못할 것이다.

(이 줄은 본문에 없다)간단히 말하자면, 소스 코드를 가지고 검사를 하는 것이다. 소스 코드의 특정 구문 기준을 선택해 그 기준이 충족될 때까지 반복하는 것이라고 생각하면 된다.

## Tracing Executions

화이트 박스 검사의 한 가지 좋은 기능은 어떤 프로그램의 기능이 실제로 커버되었는지 자동적으로 평가할 수 있는 기능이다. 이를 위해, 프로그램의 실행을 instrument하여 실행 중에 특수 기능이 어떤 코드가 실행되었는지를 추적한다. 검사 후, 이 정보는 아직 커버되지 않은 코드를 커버하기 위한 검사를 작성하는데 집중하는 개발자에게 넘겨질 수 있다.

대부분의 프로그래밍 언어에서는 실행을 추적할 수 있도록 프로그램을 설정하는 것이 오히려 어렵다. 그렇지만 파이썬에선 아니다. sys.settrace(f) 함수는 모든 줄이 실행될 때마다 호출되는 tracing function f()를 정의할 수 있게 한다. 더 좋은 점은, 현재 함수와 함수 이름, 현재 변수 내용, 그리고 더 많은 것들에 접근할 수 있도록 해준다는 것이다. 따라서 이는 실제 실행 동안 무엇이 발생하는지 분석하는 동적 분석을 위한 이상적인 도구이다.

이것을 어떻게 할 수 있는지 나타내기 위해, 다시 cgi_decode()의 특정 실행을 보자.

```python
cgi_decode("a+b")
```

> 'a b'

실행이 어떻게 cgi_decode()를 통해 이루어졌는지 추적하기 위해, sys.settrace를 활용한다. 먼저, 각 줄마다 호출될 tracing function을 정의한다. 이것은 3가지 매개 변수를 갖는다:

- frame 매개변수는 현재 위치와 변수에 접근할 수 있도록 하는 현재 frame을 준다:
  - frame.f_code는 현재 실행되는 코드이고 frame.f_code.co_name은 함수 이름이다;
  - frame.f_lineno는 현재 줄 번호를 가지고 있다; 그리고
  - frame.f_locals는 현재 지역 변수와 인자들을 가지고 있다.
- event 매개변수는 "줄"(새 줄에 도달함) 또는 "call"(함수가 호출됨)을 포함한 값을 가진 문자열이다.
- arg 매개변수는 몇몇 이벤트에 대한 추가적인 인자로, 예를 들어 "return" 이벤트의 경우 arg는 return된 값을 가진다.

