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

<code>
my_sqrt(4)
</code>

> 2.0

위 my_sqrt(4)

## Background