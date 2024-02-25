# Fuzzing: Breaking Things with Random Input

이 챕터에선, 우리는 가장 간단한 테스트 생성 기법 중 하나로 시작할 것이다. *fuzzing*이라고도 불리는 random text 생성의 주요 아이디어는 실패를 찾아내기 위한 희망을 가지고 무작위 문자열을 프로그램에 넣어보는 것이다.

https://youtu.be/YjO1pIx7wS4

## 전제 조건

- [Introduction to Software Testing](/Part1/Introduction%20to%20Software%20Testing.md) 챕터의 내용같은 소프트웨어 테스팅의 기본을 알아야함.
- [Python tutorial](https://docs.python.org/3/tutorial/)의 내용과 같은 파이썬에 대한 괜찮은 이해를 필요로 함.

우리는 이러한 전제 조건을 명시적으로 만들 수 있다. 먼저, 우리는 notebook에서 작동하기 위해 필요한 일반적인 패키지들을 import할 것이다.

> import [bookutils.setup](https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils)

> from [typing](https://docs.python.org/3/library/typing.html) import Dict, Tuple, Union, List, Any

이제, 필요하기도 한 이전 챕터들을 명시적으로 import 한다.

> import [Intro_Testing](https://www.fuzzingbook.org/html/Intro_Testing.html)

# 개요

이 챕터에서 제공된 코드를 사용하기 위해서, 다음을 쓴다;

```jsx
>>> from [fuzzingbook.Fuzzer](https://www.fuzzingbook.org/html/Fuzzer.html) import <identifier>
```

그러면 그 다음과 같은 기능을 사용할 수 있다.

이 챕터는 [A Fuzzing Architecture](https://www.fuzzingbook.org/html/Fuzzer.html#A-Fuzzing-Architecture)에서 소개된 두 중요한 classes를 제공한다. 

- Fuzzer의 기본 클래스가 되는 *Fuzzer*
- 테스트에 놓인 프로그램의 기본 클래스가 되는 *Runner*

## Fuzzers

*Fuzzer*는 간단한 인스턴트화된 RandomFuzzer를 가진 fuzzer다. Fuzzer 객체의 fuzz() method는 생성된 입력인 문자열을 return한다.

```python
>>> random_fuzzer = RandomFuzzer()
>>> random_fuzzer.fuzz()
'%$<1&<%+=!"83?+)9:++9138 42/ "7;0-,)06 "1(2;6>?99$%7!!*#96=>2&-/(5*)=$;0$$+;<12"?30&'
```

RandomFuzzer() 생성자는 여러 키워드 인자를 허용한다.

```python
>>> print(RandomFuzzer.__init__.__doc__)
Produce strings of `min_length` to `max_length` characters
           in the range [`char_start`, `char_start` + `char_range`)

>>> random_fuzzer = RandomFuzzer(min_length=10, max_length=20, char_start=65, char_range=26)
>>> random_fuzzer.fuzz()
'XGZVDDPZOOW'
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/49d5a777-35f2-40db-9a89-86307bbfd66f/c5068d3d-cbf0-4a2b-9567-2b517a62c4ad/Untitled.png)

## Runners

Fuzzer는 fuzzed된 문자열을 입력으로 받을 수 있는 Runner와 짝을 지을 수 있다. 이 결과는 class-specific status와 outcome(PASS, FAIL, or UNRESOLVED)이다. A PrintRunner는 단순히 주어진 입력을 출력하고 PASS 결과를 return한다.

```python
>>> print_runner = PrintRunner()
>>> random_fuzzer.run(print_runner)
EQYGAXPTVPJGTYHXFJ

('EQYGAXPTVPJGTYHXFJ', 'UNRESOLVED')
```

ProgramRunner는 외부 프로그램에게 생성된 입력을 준다. 이는 프로그램 상태의 짝(a CompletedProcess instance)들과 outcome(PASS, FAIL, or UNRESOLVED)을 결과로 낸다.

```python
>>> cat = ProgramRunner('cat')
>>> random_fuzzer.run(cat)
(CompletedProcess(args='cat', returncode=0, stdout='BZOQTXFBTEOVYX', stderr=''),
 'PASS')
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/49d5a777-35f2-40db-9a89-86307bbfd66f/6ca1863a-26da-42e3-9e1a-cbaaa5cb86d4/Untitled.png)

# A Testing Assignment

대충 역사이므로 중략

# A Simple Fuzzer

위 과제를 해결하고 fuzz 생성기를 만들어 보자. 아이디어는 무작위 문자를 생산하고, 버퍼 문자열 변수(out)에 그들을 넣은 뒤, 마지막으로 문자열을 return한다.

구현은 다음의 python 기능과 함수를 사용한다.

- `random.randrange(start, end)` – 무작위 수 return  [`start`, `end`)
- `range(start, end)` – [`start`, `end)` 범위 의 정수를 포함한 (리스트로 사용될 수 있는)반복자 생성.
- `for elem in list: body` – list로부터의 각 값을 elem이 받는 동안 body가 반복적으로 실행됨
- `for i in range(start, end): body` –
- `chr(n)` – return a character with ASCII code `n`

무