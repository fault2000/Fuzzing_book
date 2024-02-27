# Fuzzing: Breaking Things with Random Input

이 챕터에선, 우리는 가장 간단한 테스트 생성 기법 중 하나로 시작할 것이다. *fuzzing*이라고도 불리는 random text 생성의 주요 아이디어는 실패를 찾아내기 위한 희망을 가지고 무작위 문자열을 프로그램에 넣어보는 것이다.

https://youtu.be/YjO1pIx7wS4

### 전제 조건

- [Introduction to Software Testing](/Part1/Introduction%20to%20Software%20Testing.md) 챕터의 내용같은 소프트웨어 테스팅의 기본을 알아야함.
- [Python tutorial](https://docs.python.org/3/tutorial/)의 내용과 같은 파이썬에 대한 괜찮은 이해를 필요로 함.

우리는 이러한 전제 조건을 명시적으로 만들 수 있다. 먼저, 우리는 notebook에서 작동하기 위해 필요한 일반적인 패키지들을 import할 것이다.

<code class="language-python">
    import <a href="https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils">bookutils.setup</a>
</code><br>

<code>
    from <a href="https://github.com/uds-se/fuzzingbook//tree/master/notebooks/shared/bookutils">typing</a> import Dict, Tuple, Union, List, Any
</code>

이제, 필요하기도 한 이전 챕터들을 명시적으로 import 한다.

<code>
    import <a href="https://www.fuzzingbook.org/html/Intro_Testing.html">Intro_Testing</a>
</code>

> ## 개요
> 
> 이 챕터에서 제공된 코드를 사용하기 위해서, 다음을 쓴다;
> 
> <code>
>     from <a href="https://www.fuzzingbook.org/html/Fuzzer.html">fuzzingbook.Fuzzer</a> import &lt;identifier&gt;"
> </code>
> 
> 그러면 아래의 기능들을 사용할 수 있다.
> 
> 이 챕터는 [A Fuzzing Architecture](https://www.fuzzingbook.org/html/Fuzzer.html#A-Fuzzing-Architecture)에서 소개된 두 중요한 classes를 제공한다. 
> 
> - Fuzzer의 기본 클래스가 되는 *Fuzzer*
> - 테스트에 놓인 프로그램의 기본 클래스가 되는 *Runner*
> 
> ### Fuzzers
> 
> *Fuzzer*는 간단한 인스턴트화된 RandomFuzzer를 가진 fuzzer다. Fuzzer 객체의 fuzz() method는 생성된 입력인 문자열을 return한다.
> 
> ```python
> >>> random_fuzzer = RandomFuzzer()
> >>> random_fuzzer.fuzz()
> '%$<1&<%+=!"83?+)9:++9138 42/ "7;0-,)06 "1(2;6>?> 99$%7!!*#96=>2&-/(5*)=$;0$$+;<12"?30&'
> ```
> 
> RandomFuzzer() 생성자는 여러 키워드 인자를 허용한다.
> 
> ```python
> >>> print(RandomFuzzer.__init__.__doc__)
> Produce strings of `min_length` to `max_length` characters
>            in the range [`char_start`, `char_start` + > `char_range`)
> 
> >>> random_fuzzer = RandomFuzzer(min_length=10, > max_length=20, char_start=65, char_range=26)
> >>> random_fuzzer.fuzz()
> 'XGZVDDPZOOW'
> ```
> 
> ![image](https://github.com/fault2000/Fuzzing_book/assets/73513005/096bd647-033e-4d50-9bdb-6748f646f59e)
> 
> ### Runners
> 
> Fuzzer는 fuzzed된 문자열을 입력으로 받을 수 있는 Runner와 짝을 지을 수 있다. 이 결과는 class-specific status와 outcome(PASS, FAIL, or UNRESOLVED)이다. A PrintRunner는 단순히 주어진 입력을 출력하고 PASS 결과를 return한다.
> 
> ```python
> >>> print_runner = PrintRunner()
> >>> random_fuzzer.run(print_runner)
> EQYGAXPTVPJGTYHXFJ
> 
> ('EQYGAXPTVPJGTYHXFJ', 'UNRESOLVED')
> ```
> 
> ProgramRunner는 외부 프로그램에게 생성된 입력을 준다. 이는 프로그램 상태의 짝(a CompletedProcess instance)들과 outcome(PASS, FAIL, or UNRESOLVED)을 결과로 낸다.
> 
> ```python
> >>> cat = ProgramRunner('cat')
> >>> random_fuzzer.run(cat)
> (CompletedProcess(args='cat', returncode=0, stdout='BZOQTXFBTEOVYX', stderr=''),
>  'PASS')
> ```
> 
> ![image](https://github.com/fault2000/Fuzzing_book/assets/73513005/11273346-2b7c-42f5-b6a3-a74a3e0506ce)

# A Testing Assignment

대충 역사이므로 생략

# A Simple Fuzzer

위 과제를 해결하고 fuzz 생성기를 만들어 보자. 아이디어는 무작위 문자를 생산하고, 버퍼 문자열 변수(out)에 그들을 넣은 뒤, 마지막으로 문자열을 return한다.

구현은 다음의 python 기능과 함수를 사용한다.

- `random.randrange(start, end)` – 무작위 수 return  [`start`, `end`)
- `range(start, end)` – [`start`, `end)` 범위 의 정수를 포함한 (리스트로 사용될 수 있는)반복자 생성.
- `for elem in list: body` – list로부터의 각 값을 elem이 받는 동안 body가 반복적으로 실행됨
- `for i in range(start, end): body` –
- `chr(n)` – return a character with ASCII code `n`

무작위 숫자를 사용하기 위해, 우리는 각각의 모듈을 import해야 한다.

```python
import random
```

여기 실제 fuzzer() 함수가 나온다.

```python
def fuzzer(max_length: int = 100, char_start: int = 32, char_range: int = 32) -> str:
    """A string of up to `max_length` characters
       in the range [`char_start`, `char_start` + `char_range`)"""
    string_length = random.randrange(0, max_length + 1)
    out = ""
    for i in range(0, string_length):
        out += chr(random.randrange(char_start, char_start + char_range))
    return out
```

기본 인자와 함께, fuzzer() 함수는 무작위 문자를 가진 문자열을 return한다.

```python
fuzzer()
```

> '!7#%"*#0=)\$;%6*;>638:*>80"=</>(/*:-(2<4 !:5*6856&?""11<7+%<%7,4.8,*+&,,\$,."'

Bart Miller는 이러한 무작위하고, 구조가 없는 데이터를 "fuzz"라고 명명했다. 이제 이 "fuzz" 문자열이 특정 입력 포맷, 다시 말해, 쉼표로 구분된 값의 목록, 혹은 이메일 주소를 예상하는 프로그램의 입력이라고 가정하자. 프로그램은 이러한 입력은 아무 문제 없이 처리할 수 있을 것인가?

만약 위의 fuzzing 입력이 이미 흥미롭다면, fuzzing이 쉽게 다른 종류의 입력을 생산하도록 세팅될 수 있음을 고려하자. 예를 들어, 우리는 fuzzer()가 소문자만을 생산하도록 할 수 있다. 우리는 ord(c)를 사용해 문자 c의 ASCII 코드를 return한다.

```python
fuzzer(1000, ord('a'), 26)
```

> 'zskscocrxllosagkvaszlngpysurezehvcqcghygphnhonehczraznkibltfmocxddoxcmrvatcleysksodzlwmzdndoxrjfqigjhqjxkblyrtoaydlwwisrvxtxsejhfbnforvlfisojqaktcxpmjqsfsycisoexjctydzxzzutukdztxvdpqbjuqmsectwjvylvbixzfmqiabdnihqagsvlyxwxxconminadcaqjdzcnzfjlwccyudmdfceiepwvyggepjxoeqaqbjzvmjdlebxqvehkmlevoofjlilegieeihmetjappbisqgrjhglzgffqrdqcwfmmwqecxlqfpvgtvcddvmwkplmwadgiyckrfjddxnegvmxravaunzwhpfpyzuyyavwwtgykwfszasvlbwojetvcygectelwkputfczgsfsbclnkzzcjfywitooygjwqujseflqyvqgyzpvknddzemkegrjjrshbouqxcmixnqhgsgdwgzwzmgzfajymbcfezqxndbmzwnxjeevgtpjtcwgbzptozflrwvuopohbvpmpaifnyyfvbzzdsdlznusarkmmtazptbjbqdkrsnrpgdffemnpehoapiiudokczwrvpsonybfpaeyorrgjdmgvkvupdtkrequicexqkoikygepawmwsdcrhivoegynnhodfhryeqbebtbqnwhogdfrsrksntqjbocvislhgrgchkhpaiugpbdygwkhrtyniufabdnqhtnwreiascfvmuhettfpbowbjadfxnbtzhobnxsnf'

프로그램이 입력으로 식별자를 예상하고 있다고 가정하자. 이렇게 긴 식별자를 기대할까?

## Fuzzing External Programs

우리가 fuzz된 입력으로 외부 프로그램을 실제로 실행하면 무슨 일이 일어날지 보자. 이를 위해, 두 단계로 진행한다. 먼저, fuzz된 검사 데이터로 입력 파일을 만든다; 그 다음 선택된 프로그램에 입력 파일을 준다.

### Creating Input Files

파일 시스템을 어지럽히지 않도록 임시 파일 이름을 얻겠다.

```python
import os
import tempfile
```

```python
basename = "input.txt"
tempdir = tempfile.mkdtemp()
FILE = os.path.join(tempdir, basename)
print(FILE)
```

> /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt

이제 이 파일을 쓰기를 위해 열 수 있다. 파이썬 open() 함수는 임의의 내용을 쓸 수 있도록 열어 준다. 이는 흔히 파일이 필요없어지자마자 파일을 닫는 것을 보장하는 with문과 같이 사용된다.

```python
data = fuzzer()
with open(FILE, "w") as f:
    f.write(data)
```

파일이 실제로 만들어진 것은 파일의 내용을 읽음으로써 검증될 수 있다.

```python
contents = open(FILE).read()
print(contents)
assert(contents == data)
```

> \<?6&" !3'7-5\>18%55*,5

### Invoking External Programs

이제 입력 파일이 있으니, 프로그램을 그것을 가지고 호출할 수 있다. 재미 삼아, 산술식을 받고 평가하는 bc 계산기 프로그램을 검사한다.

bc를 호출하기 위해, Python의 subprocess 모듈을 사용한다. 다음은 이것이 어떻게 작동하는지다.

```python
import os
import subprocess
```

```python
program = "bc"
with open(FILE, "w") as f:
    f.write("2 + 2\n")
result = subprocess.run([program, FILE],
                        stdin=subprocess.DEVNULL,
                        stdout=subprocess.PIPE,
                        stderr=subprocess.PIPE,
                        universal_newlines=True) # Will be "text" in Python 3.7
```

결과로부터, 프로그램의 출력을 검사할 수 있다. bc의 경우, 산술식 표현을 평가한 것의 결과이다.

```python
result.stdout
```

> '4\n'

여기서 또한 상태를 검사할 수 있다. 0의 값은 프로그램이 정확히 종료되었음을 가리킨다.

```python
result.returncode
```

> 0

모든 에러 메세지는 results.stderr에서 사용가능하다.

```python
result.stderr
```

> ''

bc 대신, 여러분이 좋아하는 모든 프로그램을 실제로 넣어볼 수 있다. 그러나 주의할 점은, 여러분의 프로그램이 여러분의 시스템을 바꿀 수 있거나 심지어 해를 끼칠 수 있을 경우 fuzz된 입력에 이를 정확히 수행할 수 있는 데이터 혹은 명령이 포함됐을 가능성이 상당히 있다.

예를 들어, rm 명령을 fuzzing 한다고 했을 때, fuzzer()가 시스템 내의 모든 파일을 지울 인자를 생성할 확률은 대략 1/1000 정도의 확률이다. 주어진 fuzz 검사는 보통 몇백만 번 실행되므로, 이러한 확률은 꽤나 위협적이다.

### Long-Running Fuzzing

