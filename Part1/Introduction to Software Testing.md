# Introduction to Software Testing

우리가 이 책의 중심 부분으로 가기 전에, 소프트웨어 테스팅에 대한 필수적인 개념에 대해 소개하려고 한다. 왜 소프트웨어를 시험하는 것이 필수적인 걸까? 어떻게 소프트웨어를 시험할까? 어떻게 시험이 성공적이라고 말할 수 있을까? 어떻게 시험이 충분했다고 알 수 있을까? 이 장에서, 가장 중요한 개념을 상기하면서 동시에 Python 및 대화형 노트북에 대해 알아보겠다.  

```python
from bookutils import YouTubeVideo
YouTubeVideo('C8_pjdl7pK0')
```

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/C8_pjdl7pK0/0.jpg)](https://www.youtube.com/watch?v=C8_pjdl7pK0)

이 장(그리고 이 책)은 테스트에 관한 교과서를 대체하도록 설정되지 않았다. 권장 읽기는 마지막에 있는 [background](#background)를 참조하라.

## Simple Testing

간단한 예시로 시작해보자. 당신의 동료에게 제곱근 함수 √x를 구현하기를 요청받았다(잠깐만 당신이 그것을 이미 가지고 있지 않은 환경에 있다고 가정하자). [Newton-Raphson method](https://en.wikipedia.org/wiki/Newton%27s_method)을 공부한 후에, 그녀는 다음과 같은 파이썬 코드를 고안해 내는데, 이 함수는 실제로 제곱근을 계산한다고 주장한다.

```python
def my_sqrt(x):
    """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx
```

이제 여러분의 일은 이 기능이 실제로 자신이 주장하는 것을 수행하는지 여부를 확인하는 것이 당신의 일이다.

### Understanding Python Programs

만약 여러분이 파이썬에 처음이라면, 아마 위의 코드가 무슨 일을 하는지 정확히 이해해야만 할 것이다. 여기선 파이썬이 어떻게 작동하는지 알기 위해 [파이썬 튜토리얼](https://docs.python.org/3/tutorial/)을 추천한다. 이 코드를 이해하기 위한 가장 중요한 것 3가지는 다음과 같다:

1. 파이썬은 들여 쓰기를 통해 프로그램을 구성하므로, 함수와 while 구문은 들여씀으로써 정의된다.
2. 파이썬은 동적 타입으로, x, approx 혹은 guess같은 변수들의 타입이 런타임에 결정됨을 뜻한다.
3. 파이썬의 구문적 기능들의 대부분은 다른 흔한 언어들에서 영감을 받았다, 예를 들어, 제어 구조(while, if), 할당(=), or 비교(==, !=, <).

이를 가지고, 위 코드가 뭘 하는지 이미 알 수 있을 것이다. x/2를 가진 guess로 시작해서, approx의 값이 더 이상 바뀌지 않을 때까지 approx에 점점 더 나은 근사치를 산출해낸다. 이 값이 마지막으로 return되는 값이다.

### Running a Function

my_sqrt()가 정확히 작동하는지 알아내기 위해, 몇가지 값을 통해 테스트할 수 있다. 예를 들어, x = 4일 경우 이는 정확한 값을 생산해낸다.

```python
my_sqrt(4)
```

> 2.0

위 my_sqrt(4)(소위, cell) 부분은 기본으로 입력을 평가하는 파이썬 인터프리터에 주는 입력이다. 아래 (2.0) 부분은 출력이다. 우리는 my_sqrt(4)가 정확한 값을 생산함을 볼 수 있다.

x = 2.0일 경우에도 마찬가지다.

```python
my_sqrt(2)
```

> 1.414213562373095

### interacting with Notebooks

대충 노트북으로 상호작용할 경우 직접 해볼 수 있다는 뜻

### Debugging a Function

my_sqrt()가 작동하는 방식을 보기 위한 간단한 전략은 중요한 장소에 print() 문을 삽입하는 것이다. 예를 들어, 여러분은 approx가 각 루프 반복마다 실제 값에 얼마나 가까워지는지 보기 위해 approx의 값을 남길 수 있다.

```python
def my_sqrt_with_log(x):
    """Computes the square root of x, using the Newton–Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        print("approx =", approx)  # <-- New
        approx = guess
        guess = (approx + x / approx) / 2
    return approx
```

```python
my_sqrt_with_log(9)
```

> approx = None
> approx = 4.5
> approx = 3.25
> approx = 3.0096153846153846
> approx = 3.000015360039322
> approx = 3.0000000000393214
> 3.0

### Checking a Function

테스팅으로 돌아가보자. 우리는 코드를 읽고 쓸 수 있지만, my_sqrt(2)의 위 값은 정말로 정확한 것일까? 이는 √x의 제곱이 x가 됨을 활용하여 쉽게 검증할 수 있다:

```python
my_sqrt(2) * my_sqrt(2)
```

> 1.9999999999999996

반올림 오류를 얻긴 했지만, 그 외에는 괜찮아 보인다.

우리가 지금 한 것은 위 프로그램을 검사한 것이다: 우리는 주어진 입력에 대해 프로그램을 실행했고 그것의 결과가 정확한지 아닌지 검사했다. 이러한 검사는 프로그램이 생산에 들어가기 전에 거의 최소한의 질적 보장이다.

## Automating Test Execution

지금까지, 우리는 위 프로그램을 수동으로 검사했다, 이는 검사의 매우 유연한 방법이지만, 오래갈수록 이는 효율적이지 않다.

1. 수동으로는 매우 제한된 수의 실행과 그 결과를 검사할 수 있다.
2. 프로그램의 어떤 변화가 생긴 후 검사 프로세스를 다시 수행해야 한다.

이는 왜 자동 검사가 매우 유용한지 보여준다. 이렇게 하는 간단한 방법 중 하나는 컴퓨터가 먼저 계산을 수행하고 그 결과를 검사하도록 하는 것이다.

예를 들어, 이 코드 조각은 √4 = 2임을 자동적으로 검사한다:

```python
result = my_sqrt(4)
expected_result = 2.0
if result == expected_result:
    print("Test passed")
else:
    print("Test failed")
```

> Test passed

이 검사의 좋은 점은 이걸 반복해서 실행할 수 있다는 것이고, 따라서 최소한 4의 제곱근이 정확히 계산됨을 보장한다. 하지만 여기서도 여전히 몇 가지 문제가 존재하는데:

1. 하나의 검사를 수행하기 위해 5줄이 필요하다.
2. 반올림 오류를 다루지 않는다.
3. 오직 하나의 입력과 하나의 결과만을 검사한다.

이 문제들을 하나씩 짚어보자. 먼저, 검사를 좀 더 작게 만들어 보자. 거의 모든 프로그래밍 언어는 가진 조건를 자동적으로 검사하고 만약 조건이 다르면 실행을 중지하는 방법을 가지고 있다. 이는 assertion이라고 불리며, 이는 검사에서 대단히 유용하다.

파이썬에서, assert 문은 조건을 받고, 만약 조건이 참이라면, 아무 일도 일어나지 않는다(만약 모든 것이 그래야되는 것처럼 잘 돌아가면, 여러분은 방해받지 않을 것이다). 하지만 만약 조건이 거짓으로 평가되면, assert는 예외를 발생시키고, 이는 검사가 실패했음을 나타낸다.

여기의 예시에서, 여러분은 assert를 사용하여 my_sqrt()가 위처럼 예상된 결과를 낼지 쉽게 검사할 수 있다.

```python
assert my_sqrt(4) == 2
```

이 코드 줄을 실행하면, 아무 일도 일어나지 않는다: 우리는 방금 우리의 구현이 정말 √4 = 2를 계산함을 보여주었다(혹은 주장했다).  
  
이에 반해 기억할 점은, 소수점 계산이 반올림 오류를 유발할 수 있다는 것이다. 그래서 우리는 단순히 두 소수점 값을 동일하게 비교할 수 없다; 그것보다, 우리는 그들의 사이의 절대적인 차이가 보통 ϵ 혹은 epsilon로 표시되는 특정 임계값 이하로 유지됨을 보장한다. 이게 어떻게 우리가 그것을 할 수 있는지다:

```python
EPSILON = 1e-8
```

```python
assert abs(my_sqrt(4) - 2) < EPSILON
```

이 목적을 위해 특별한 함수를 만들 수 있고 이제 더 견고한 값을 위한 더 많은 검사를 진행할 수 있다.

```python
def assertEquals(x, y, epsilon=1e-8):
    assert abs(x - y) < epsilon
```

```python
assertEquals(my_sqrt(4), 2)
assertEquals(my_sqrt(9), 3)
assertEquals(my_sqrt(100), 10)
```

작동하는 것처럼 보인다, 그렇지 않은가? 만약 계산의 예상된 결과를 안다면, 우리 프로그램이 정확히 작동함을 보장하기 위해 이러한 assertion을 반복해서 사용할 수 있다.

(Hint: 진정한 파이썬 프로그래머는 math.isclose() 함수를 대신 사용할 것이다.)

## Generating Tests

√x × √x = x 가 보편적으로 성립함을 기억하는가? 몇 가지 값으로 이는 명시적으로 검사해볼 수 있다.

```python
assertEquals(my_sqrt(2) * my_sqrt(2), 2)
assertEquals(my_sqrt(3) * my_sqrt(3), 3)
assertEquals(my_sqrt(42.11) * my_sqrt(42.11), 42.11)
```

여전히 작동하는 것처럼 보인다, 그렇지 않은가? 그렇지만 가장 중요하게는, √x × √x = x는 몇 천 개의 값의 경우에 매우 쉽게 검사할 수 있는 것이다.

```python
for n in range(1, 1000):
    assertEquals(my_sqrt(n) * my_sqrt(n), n)
```

my_sqrt()를 100개의 값으로 검사하는데 얼마나 걸리는가? 한 번 보자.

우리는 우리만의 [Timer module](/Appendices/Timer.md)을 사용해 경과 시간을 특정한다. Timer를 사용할 수 있게 하기 위해, 먼저 다른 노트북을 import할 수 있게 하는 우리의 다용도 모듈을 import한다.

```python
import bookutils.setup
```

```python
from Timer import Timer
```

```python
with Timer() as t:
    for n in range(1, 10000):
        assertEquals(my_sqrt(n) * my_sqrt(n), n)
print(t.elapsed_time())
```

> 0.015223124995827675

10,000개의 값은 100분의 1초가 걸리며 my_sqrt()가 한 번 실행되면 1/1000000 초 혹은 1 마이크로초가 걸린다.

이제 이걸 무작위로 고른 10,000개의 값으로 반복해보자. 파이썬의 random.random() 함수는 0.0과 1.0사이의 무작위 값을 돌려준다:

```python
import random
```

```python
with Timer() as t:
    for i in range(10000):
        x = 1 + random.random() * 1000000
        assertEquals(my_sqrt(x) * my_sqrt(x), x)
print(t.elapsed_time())
```

> 0.017563624991453253

몇 초 내에, 이번엔 10,000개의 무작위 값을 검사했다. 그리고 각 시간마다, 제곱근은 정확히 계산된다. my_sqrt의 매 변화마다 검사를 반복하여 my_sqrt가 원하는 대로 작동한다는 우리의 신뢰를 강화할 수 있다. 그럼에도 주의할 점은 무작위 값을 생산하는 무작위 함수가 편향되지 않음에도 불구하고, 프로그램의 행동을 극적으로 바꿀 특별한 값을 생성해낼 것 같지는 않다. 이는 추후 아래에서 논의된다.

## Run-Time Verification

my_sqrt를 위한 검사를 쓰고 실행하는 대신, 구현 바로 옆에 검사를 통합하고 갈 수 있다. 이 방법으로는, my_sqrt()가 불러올 때마다 자동적으로 검사된다.

이러한 자동적 런타임 검사는 매우 쉽게 구현할 수 있다.

```python
def my_sqrt_checked(x):
    root = my_sqrt(x)
    assertEquals(root * root, x)
    return root
```

이제, my_sqrt_checked()로 루트를 계산할 때마다...

```python
my_sqrt_checked(2.0)
```

> 1.414213562373095

...우리는 이미 결과가 정확함을 알고 있으며, 모든 새로운 계산도 그럴 것임을 안다.

자동 런타임 검사는 두 가지를 가정한다:

- 이러한 런타임 검사를 공식화할 수 있어야 한다. 확인할 수 있는 구체적인 값을 갖는 것은 항상 가능해야 하지만, 추상적인 방식으로 원하는 속성을 공식화하는 것은 매우 복잡할 수 있다. 실제로, 여러분은 어떤 속성이 가장 중요한지 결정해야하고, 그들을 위한 적절한 검사를 설계해야한다. 추가로, 런타임 검사는 local 속성에만이 아닌 모두 인식되어져야 하는 프로그램 상태의 여러 속성들에 의존한다.
- 이러한 런타임 검사를 제공할 수 있어야 한다. my_sqrt()의 경우, 검사는 별로 비싸지 않다; 하지만 만약 큰 데이터 구조를 가진 간단한 명령을 검사해야 한다면, 검사의 비용은 엄두도 못 낼 정도로 클 것이다. 실제로, 제조 과정에서는 보통 효율성을 위해 안정성을 대가로 런타임 검사를 비활성화한다. 반면, 포괄적인 런타임 검사 제품군은 에러를 찾고 그들을 빠르게 디버그하는데 훌륭한 방법이다; 여러분은 생산 과정에서 얼마나 많은 그러한 기능을 원하는지 결정해야 한다.

런타임 검사의 중요한 제한은 검사할 결과가 있어야지만 정확성을 보장할 수 있다는 것이다 - 이는, 그들은 항상 하나가 있음을 보장하지 않는다. 이는 훨씬 더 높은(종종 수동) 노력으로 똑같이 결과가 있음을 보장하는 symbolic verification 기술과 프로그램 proofs와 비교해서 중요한 제한이다.

## System Input vs Function Input

이 시점에서, 우리는 my_sqrt()를 다른 프로그래머들에게 사용가능하도록 만들어 그들의 코드에 my_sqrt()를 임베드했을 수도 있다. 어느 시점에서, my_sqrt()는 서드 파티, 예를 들어 개발자에 의해 통제될 수 없는 곳에서 온 입력을 처리해야할 수 있다.

이제 서드파티의 통제 하에 있는 문자열을 입력으로 받는 sqrt_program()이라는 프로그램을 가정해서 시스템 입력을 시뮬레이션해보자:

```python
def sqrt_program(arg: str) -> None:
    x = int(arg)
    print('The root of', x, 'is', my_sqrt(x))
```

우리는 sqrt_program이라는 명령 줄로부터 시스템 입력을 허용하는 프로그램을 가정하자.

```bash
sqrt_program 4
2
```

sqrt_program()은 여러 시스템 입력으로 쉽게 부를 수 있다.

```python
sqrt_program("4")
```

> The root of 4 is 2.0

문제가 뭘까? 글쎄, 문제는 외부 입력에 대해 유효성 검사흫 하지 않는다는 것이다. 예를 들어, sqrt_program(-1)을 실행해보자, 무슨 일이 일어나는가?

실제로, 만약 my_sqrt()를 음수로 실행하면, 무한 루프에 빠지게 된다. 기술적인 이유로, 이 장에서는 무한 루프를 가질 수 없다(우리가 영원히 실행되는 코드를 원하지 않는 이상); 그래서 특별한 ExpectTimeOut(1) 구조를 사용해 실행이 1초 이상 실행되는 것을 방해한다.

```python
from ExpectError import ExpectTimeout
```

```python
with ExpectTimeout(1):
    sqrt_program("-1")
```

> Traceback (most recent call last):
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/1288144681.py", line 2, in <cell line: 1>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sqrt_program("-1")
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/449782637.py", line 3, in sqrt_program
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print('The root of', x, 'is', my_sqrt(x))
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/2661069967.py", line 5, in my_sqrt
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while approx != guess:
> &nbsp;&nbsp;&nbsp;&nbsp;File "/Users/zeller/Projects/fuzzingbook/notebooks/Timeout.ipynb", line 43, in timeout_handler
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;raise TimeoutError()
> TimeoutError (expected)

위 메세지는 에러 메세지로, 무언가 잘못되었음을 가리킨다. 이는 함수의 call stack과 에러 당시에 활성화된 줄의 리스트를 보여준다. 제일 아래의 줄이 마지막으로 실행된 줄이다; 위로 갈수록 함수 호출을 나타내며 - 우리의 경우 my_sqrt(x)까지 간다.

여기서 예외로 인해 우리의 코드가 종료되기를 원치는 않는다. 결과적으로, 외부 입력을 허락할 때마다, 적절히 검증되었는지를 보장해야 한다. 예를 들어 다음과 같이 쓸 수 있다:

```python
def sqrt_program(arg: str) -> None:
    x = int(arg)
    if x < 0:
        print("Illegal Input")
    else:
        print('The root of', x, 'is', my_sqrt(x))
```

이러면 my_sqrt()가 그것의 요구조건에 따라 실행됨을 보장할 수 있다.

```python
sprt_program("-1")
```

> Illegal Input

그렇지만 기다려봐라! 만약 sqrt_program()이 숫자로 실행되지 않는다면 무슨 일이 일어날까?

한번 해보자! 숫자가 아닌 문자열을 변환하려고 시도하면, 런타임 에러를 결과로 얻을 것이다.

```python
from ExpectError import ExpectError
```

```python
with ExpectError():
    sqrt_program("xyzzy")
```

> Traceback (most recent call last):
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/1336991207.py", line 2, in <cell line: 1>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sqrt_program("xyzzy")
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/3211514011.py", line 2, in sqrt_program
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x = int(arg)
> ValueError: invalid literal for int() with base 10: 'xyzzy' (expected)

여기 잘못된 입력을 검사하는 또다른 버전이 있다.

```python
def sqrt_program(arg: str) -> None:
    try:
        x = float(arg)
    except ValueError:
        print("Illegal Input")
    else:
        if x < 0:
            print("Illegal Number")
        else:
            print('The root of', x, 'is', my_sqrt(x))
```

```python
sqrt_program("4")
```

> The root of 4.0 is 2.0

```python
sqrt_program("-1")
```

> Illegal Number

```python
sqrt_program("xyzzy")
```

> Illegal Input

이제 우리는 프로그램이 제어되지 않는 상태에 절대 빠지지 않고 적절하게 모든 종류의 입력을 다룰 수 있어야 함을 system 단계에서 볼 수 있다. 물론 이는 그들의 프로그램을 모든 상황에서 튼튼하게 만들어야 하기 위해 노력하는 개발자들의 몫이다. 그렇지만 이 짐은 소프트웨어 검사를 생성할 때 이점이 된다: 만약 프로그램이 모든 종류의 입력을 다룰 수 있다면(아마 잘 정의된 에러 메세지와 함께), 우리 또한 모든 종류의 입력을 보낼 수 있다. 그렇지만 생성된 값으로 함수를 호출할 때, 함수의 정확한 전제 조건을 알아야만 한다.

## The Limits of Testing

우리의 최선의 노력을 검사에 부었음에도 불구하고, 여러분은 언제나 입력의 유한한 집합에 대해 기능을 검사하고 있음을 명심해야 한다. 따라서, 여기엔 언제나 함수가 여전히 실패할 수 있는 검사되지 않은 입력이 있을 수 있다.

예를 들어, my_sqrt()의 경우, √0을 계산하는 것은 0으로 나누기를 결과로 낸다.

```python
with ExpectError():
    root = my_sqrt(0)
```

> Traceback (most recent call last):
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/820411145.py", line 2, in <cell line: 1>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root = my_sqrt(0)
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/2661069967.py", line 7, in my_sqrt
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;guess = (approx + x / approx) / 2
> ZeroDivisionError: float division by zero (expected)

지금까지의 우리의 검사에선, 이 상태를 검사해오진 않으며, 이는 √0 = 0으로 빌드된 프로그램이 놀랍게도 실패함을 의미한다. 하지만 우리의 무작위 생성기를 1-1000000보다 0-1000000의 범위로 입력을 생산하도록 설정하였다하더라도, 우연히 0의 값을 생산할 확률은 여전히 100만 분의 1이다. 만약 함수의 행동이 몇몇 소수의 개별적인 값에서 근본적으로 다르다면, 보통 무작위 검사는 이러한 값을 생산해낼 확률이 매우 적다.

물론 x에 대해 허용되는 값을 문서화하고 특수한 경우인 x = 0을 다루는 것으로 개별적으로 함수를 수정할 수 있다:

```python
def my_sqrt_fixed(x):
    assert 0 <= x
    if x == 0:
        return 0
    return my_sqrt(x)
```

이걸로, 정확히 √0 = 0을 계산할 수 있다:

```python
assert my_sqrt_fixed(0) == 0
```

잘못된 값은 이제 예외를 결과로 가져온다:

```python
with ExpectError():
    root = my_sqrt_fixed(-1)
```

> Traceback (most recent call last):
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/305965227.py", line 2, in <cell line: 1>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root = my_sqrt_fixed(-1)
> &nbsp;&nbsp;&nbsp;&nbsp;File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_71717/3001478627.py", line 2, in my_sqrt_fixed
> &nbsp;&nbsp;&nbsp;&nbsp;assert 0 <= x
> AssertionError (expected)

여전히, 광범위한 범위의 검사가 프로그램의 정확성에 높은 자신감을 우리에게 줄 수 있지만, 이는 모든 미래의 실행이 정확한지에 대한 보장을 제공하지는 못함을 기억해야만 한다. 모든 결과를 검사하는 런타임 검증조차, 결과를 도출해야만 결과가 정확함을 보장할 수 있을 뿐이다; 하지만 미래의 실행이 실패한 검사로 이어지지 않을 거라는 보장은 없다. 이 글을 쓰면서, 나는 my_sqrt_fixed(x)가 모든 유한한 수 x에 대해 √x의 정확한 구현임을 믿지만, 확신할 수는 없다.

Newton-Raphson 방법을 통해, 구현이 정확함을 실제로 증명할 수 있는 좋은 기회를 얻을 수 있다: 구현은 간단하고, 수학은 잘 이해할 수 있다. 아아, 이는 적은 도메인의 경우 중 하나일 뿐이다. 우리가 본격적인 정확성 증명에 들어가고 싶지 않다면, 검사로 할 수 있는 우리의 최선의 선택은

1. 여러, 잘 선택된 입력으로 프로그램을 검사한다; 그리고
2. 결과를 광범위하고 자동적으로 검사한다.

프로그램을 철저히 검사할 수 있도록 도와주고 여기에 더해 정확성을 위한 그들의 상태를 검사하는 것을 돕는 기술을 고안하는 것들이 나머지 코스에서 남은 것들이다. 즐거운 시간 되기를!

## Lessons Leraned

- 검사의 목적은 버그를 찾을 수 있도록 프로그램을 실행하는 것이다.
- 검사 실행, 검사 생성, 그리고 검사 결과를 검사하는 것은 자동화되어질 수 있다.
- 검사는 불완전하다; 코드가 에러로부터 자유롭다고 100% 보장을 제공할 순 없다.

## Next Steps

여기서부터, 여러분은 다음으로 넘어갈 수 있다.

- [무작위 입력으로 프로그램을 검사하기 위해 fuzzing을 사용하기(use fuzzing to test programs with random inputs)](/Part2/Fuzzing,%20Breaking%20Things%20with%20Random%20Inputs.md)

즐거운 읽기 되기를!

## Background

소프트웨어 검사와 분석에는 많은 연구가 있다.

- 완전 새로운 현대의, 포괄적인, 그리고 검사의 온라인 교과서는  "[Effective Software Testing: A Developer's Guide](https://www.effective-software-testing.com/)" [[Maurício Aniche, 2022](https://www.effective-software-testing.com/)]이다. 매우 추천한다!
- 이 책의 경우, 영역에 대한 소개 때문에 "Software Testing and Analysis"[[Pezzè et al, 2008](http://ix.cs.uoregon.edu/~michal/book/)]를 기쁘게 추천한다. 이 책의 강한 기술적 집중은 우리의 방법론에 잘 들어맞는다.
- 심리학 및 조직을 포함한 소프트웨어 검사에 대한 포괄적인 접근을 위한 또다른 중요한 필독책으로는 "Software Testing Techniques"[[Beizer et al, 1990](https://dl.acm.org/citation.cfm?id=79060)]과 "The Art of Software Testing" [[Myers et al, 2004](https://dl.acm.org/citation.cfm?id=983238)]이 있다.

## Exercises

이 부분은 공통적으로 생략했다. 궁금하면 직접 해보자.