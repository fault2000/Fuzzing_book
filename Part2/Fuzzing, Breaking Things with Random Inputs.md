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

## A Testing Assignment

대충 역사이므로 생략

## A Simple Fuzzer

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

이제 프로그램에 많은 수의 입력을 넣어 어디선가 충돌이 일어날지 보자. 우리는 입력 데이터와 실제 결과를 짝지어 runs이라는 변수에 담아 모든 결과를 저장한다.(이를 실행하는 것은 조금 걸릴 수도 있다.)

```python
trials = 100
program = "bc"

runs = []

for i in range(trials):
    data = fuzzer()
    with open(FILE, "w") as f:
        f.write(data)
    result = subprocess.run([program, FILE],
                            stdin=subprocess.DEVNULL,
                            stdout=subprocess.PIPE,
                            stderr=subprocess.PIPE,
                            universal_newlines=True)
    runs.append((data, result))
```

이제 몇몇 통계의 경우 runs를 쿼리할 수 있다. 예를 들어, 얼마나 많은 실행이 실제로 통과했는지 쿼리할 수 있다 -- 이는, 에러 메세지가 없는 경우들이다. 여기선 list comprehension을 사용한다: for *element* in *list* if *condition* 형태의 표현식은 *list*로부터 *element*를 각각 받아 *condition*이 참인지 평가된 표현식의 리스트를 return한다(사실, list comprehension은 list generator를 return 하지만, 우리의 목적에선 generator는 list처럼 행동한다). 여기서, 우리는 *condition*이 성립하는 모든 *element*의 경우 *expression*은 1이 된다.

```python
sum(1 for (data, result) in runs if result.stderr == " ")
```

> 9

대부분의 입력이 유효하지 않은 것처럼 보인다 - 크게 놀랄 일은 아닌 것이, 무작위 입력이 유효한 산술적 표현식을 담을 것 같지 않기 때문이다.

첫 에러 메세지를 살펴보자.

```python
errors = [(data, result) for (data, result) in runs if result.stderr != ""]
(first_data, first_result) = errors[0]

print(repr(first_data))
print(first_result.stderr)
```

> '5&8>"86,?"/7!1%5-**&-\$&)\$91;"21(\'8"(%\$4,("(&!67%89\$!.?(*(96(28\$=6029:<:\$(6 !-+2622(&4'
>
> Parse error: bad character '&'
> &nbsp;&nbsp;&nbsp;&nbsp;/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1

illegal character, parse error, 혹은 syntax error말고 다른 메세지(다시 말해, crash 혹은 you found a fatal bug같은 메세지)를 가진 실행이 있는가? 많이는 없을 것이다.

```python
[result.stderr for (data, result) in runs if
 result.stderr != ""
 and "illegal character" not in result.stderr
 and "parse error" not in result.stderr
 and "syntax error" not in result.stderr]
```

> ["\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad assignment: left side must be scale, ibase, obase, seed, last, var, or array element\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '?'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '?'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '?'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad assignment: left side must be scale, ibase, obase, seed, last, var, or array element\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '&'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character '?'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> "\nParse error: bad character ':'\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> "\nParse error: bad character '''\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n",
> '\nParse error: bad expression\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n',
> '\nParse error: bad token\n    /var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/tmpnds9g27_/input.txt:1\n\n']

bc에 의해 제기된 충돌은 단순히 충돌일 수도 있다. 운 나쁘게도, return 코드는 0이 아니다.

```python
sum(1 for (data, result) in runs if result.returncode != 0)
```

> 91

위 bc 테스트 실행을 더 실행되도록 나두면 어떻게 될까? 테스트가 실행되는 동안, 1989년에 최신 기술이 어땠는지 살펴보자.

## Bugs Fuzzers Find

Miller와 그의 학생들이 fuzzing을 본격적으로 시작해보았을 때, 1/3 이상의 UNIX utilities이 오류를 일으켰다.

이러한 UNIX 유틸리티들이 네트워크 입력을 처리하는 스크립트에서 사용되었다는 걸 고려했을 때 이는 충격적인 결과였다. 많은 개발자들이 그들만의 fuzzer를 빠르게 만들고 실행했으며, 서둘러 보고된 오류들을 고쳤으며, 외부 입력을 더이상 믿어선 안 된다는 것을 배웠다.

Miller의 fuzzing 실험에서 무슨 종류의 문제가 발견되었을까? 이는 바로 1990년대 개발자들이 저지른 실수를 오늘날에도 반복하고 있다는 것이다.

### Buffer Overflows

많은 프로그램들이 입력과 입력 요소에 대한 최대 길이가 내장되어 있다. C 같은 언어에서, 프로그램(혹은 개발자)가 알아채지 못하면 이러한 길이를 초과하는 것은 쉬우며 소위 **버퍼 오버플로우**를 발생시킨다. 예를 들어 다음 코드는 input 문자열이 8 문자를 넘어가더라도 weekday라는 문자열에 복사한다.

```c
char weekday[9]; // 8 characters + trailing '\0' terminator
strcpy(weekday, input);
```

아이러니하게도, 만약 input이 "Wednesday"(9글자)면 이미 실패한다. 모든 초과된 문자(여기선, 'y'와 다음 '\0'string terminator)는 weekday 뒤에 메모리 내의 존재하는 값과 상관없이 단순히 복제되어 임의의 행동을 일으킨다; 'n'에서 'y'로 설정될 수 있는 boolean 문자 변수일수도 있다. fuzzing에서는 임의의 긴 입력과 입력 요소를 생산하는 것은 매우 쉽다.

파이썬 함수 내에서 이러한 버퍼 오버플로우 행동을 쉽게 시뮬레이팅할 수 있다.

```python
def crash_if_too_long(s):
    buffer = "Thursday"
    if len(s) > len(buffer):
        raise ValueError
```

이건 매우 빠르게 충돌을 일으킨다.

```python
from ExpectError import ExpectError
```

```python
trials = 100
with ExpectError():
    for i in range(trials):
        s = fuzzer()
        crash_if_too_long(s)
```

> ```bash
> Traceback (most recent call last):
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/292568387.py", line 5, in <cell line: 2>
>     crash_if_too_long(s)
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/2784561514.py", line 4, in crash_if_too_long
>     raise ValueError
> ValueError (expected)
> ```

위 코드의 with ExpectError() 줄은 에러 메세지가 출력되도록 보장하지만 실행은 계속된다; 이는 "예상된" 에러와 "예상되지 않은" 에러를 다른 코드 예시에서 구분하기 위함이다.

### Missing Error Checks

많은 프로그래밍 언어가 exception을 가지고 있진 않지만, 대신 예외적인 상황에서 특별한 **에러 코드**를 return하는 함수를 가진다. 예를 들어, C 언어의 함수 getchar()는 보통 표준 입력으로부터 문자를 return한다; 만약 더 이상 사용가능한 입력이 없을 경우, 특별한 값인 EOF(end of file)을 return한다. 이제 개발자가 다음 문자에 대한 입력을 스캔하고, 공백 문자를 읽을 때까지 getchar()로 문자를 읽는다고 가정해보자.

```c
while (getchar() != ' ');
```

퍼징으로 완벽히 구현가능한 것처럼 입력이 조기에 종료되면 무슨 일이 일어날까? 글쎄, getchar()은 EOF를 return하고, 다시 불러질 때마다 EOF를 계속해서 return할 것이다; 그래서 위 코드는 단순히 무한 루프에 들어갈 것이다.

다시, 우리는 이러한 놓친 에러 검사를 시뮬레이팅할 수 있다. 여기 만약 입력에 공백 문자가 나타나지 않는다면 효율적으로 기다리는 함수가 있다.

```python
def hang_if_no_space(s):
    i = 0
    while True:
        if i < len(s):
            if s[i] == ' ':
                break
        i += 1
```

우리의 [Introduction to Testing](/Part1/Introduction%20to%20Software%20testing.md)의 타임아웃 메커니즘을 사용하여, 조금의 시간 후에 이 함수를 interrupt할 수 있다. 그리고 맞다, 많지 않은 fuzzing 입력 후에 이것은 대기 상태가 된다.

```python
from ExpectError import ExpectTimeout
```

```python
trials = 100
with ExpectTimeout(2):
    for i in range(trials)
    s = fuzzer()
    hang_if_no_space(s)
```

> ```bash
> Traceback (most recent call last):
> File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/3194687366.py", line 5, in <cell line: 2>
>   hang_if_no_space(s)
> File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/3035466707.py", line 3, in hang_if_no_space
>   while True:
> File "/Users/zeller/Projects/fuzzingbook/notebooks/Timeout.ipynb", line 43, in timeout_handler
>   raise TimeoutError()
> TimeoutError (expected)
> ```

위 코드의 with ExpectTimeout() 줄은 근접한 코드의 실행이 2초 후에 에러 메세지를 출력하며 interrupt됨을 보장한다.

### Rogue Numbers

fuzzing과 함께라면 모든 종류의 흥미로운 행동을 일으키는 **흔하지 않은 값**을 입력으로 생성하는 것은 쉽다. 다음 C 언어 코드를 보자, 이것은 먼저 입력으로부터 버퍼 크기를 읽고, 주어진 크기의 버퍼를 할당한다.

```c
char *read_input() {
    size_t size = read_buffer_size();
    char *buffer = (char *)malloc(size);
    // fill buffer
    return (buffer);
}
```

만약 size가 매우 커서 프로그램 메모리를 초과하면 무슨 일이 일어날까? size가 다음 입력되는 문자의 수보다 작으면 무슨 일이 일어날까? size가 음수라면? 무작위 수를 여기서 제공함으로써 fuzzing은 모든 종류의 손상을 생성할 수 있다.

다시, 파이썬에서 우리는 이러한 rogue number를 쉽게 시뮬레이트 해볼 수 있다. 함수 collapse_if_too_large()는 주어진 값(문자열)이 정수로 변환된 후 너무 크면 실패한다.

```python
def collapse_if_too_large(s):
    if int(s) > 1000:
        raise ValueError
```

fuzzer()를 이용해 정수의 문자열을 생성할 수 있다.

```python
long_number = fuzzer(100, ord('0'), 10)
print(long_number)
```

> 7056414967099541967374507745748918952640135045

만약 이러한 숫자들을 collapse_if_too_large()에게 주면, 이 함수는 매우 빠르게 실패할 것이다.

```python
with ExpectError():
    collapse_if_too_large(long_number)
```

> ```bash
> Traceback (most recent call last):
> File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/2775103647.py", line 2, in <cell line: 1>
>   collapse_if_too_large(long_number)
> File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/1591744602.py", line 3, in collapse_if_too_large
>   raise ValueError
> ValueError (expected)
> ```

만약 우리가 이 정도의 메모리를 시스템에 할당하고 싶다면, 위처럼 빠르게 실패하는 것은 차라리 좋은 옵션일 수도 있다. 현실에선, 메모리 부족은 시스템을 극적으로 느리게 만들어 거의 반응을 못하는 수준까지 도달할 것이다 - 그리고 재시작만이 유일한 옵션일 것이다.

이것들은 모두 나쁜 프로그래밍 혹은 나쁜 프로그래밍 언어로 인한 문제라고 주장할 수 있다. 그렇지만 매일 1000여 명의 사람들이 프로그램을 매일 시작하고 있고, 그들 모두가 반복해서 같은 실수를 하고 있다.

## Catching Errors

Miller와 그의 학생들이 첫 fuzzer를 만들었을 때, 그들은 프로그램이 crash 혹은 hang되었기에 에러를 단순히 식별할 수 있었다. - 이 두 상태는 쉽게 인식할 수 있다. 그렇지만 만약 실패가 더 사소하다면, 우리는 추가적인 검사를 찾아내야 한다.

### Generic Checkers

[위에서 논의된](#buffer-overflows) 버퍼 오버플로우는 보다 일반적인 문제의 특별한 예이다: C와 C++같은 언어들에서, 프로그램은 메모리 내의 임의의 부분에 접근할 수 있다 - 이러한 임의의 부분은 초기화되어 있지 않거나, 이미 free되었거나 단순히 접근하려고 시도하는 데이터 구조의 부분이 아닐 수도 있다. 이는 여러분이 운영 체제를 쓰기를 원한다면 필수적이며, 최대의 성능 혹은 제어를 원한다면 좋지만, 여러분이 실수를 피하고 싶다면 꽤나 나쁠 것이다. 운 좋게도, 여기엔 런타임에서 이러한 문제를 잡아내는데 도움이 되는 도구들이 있고, 그들은 fuzzing과 조합했을 때 아주 좋다.

#### Checking Memory Accesses

검사 진행 도중 문제가 될 수 있는 메모리 접근을 감지하기 위해, C 프로그램을 특별한 memory-checking 환경에서 실행할 수 있다; 런타임 동안, 이 환경은 각각의 모든 메모리 명령들이 유효한, 초기화된 메모리에 접근하는지 검사한다. 유명한 예시로는 [LLVM Address Sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)가 있으며 이는 잠재적으로 위험한 메모리 안전 위반의 전체 집합을 감지한다. 다음 예제에서는 이 도구로 다소 간편한 C 프로그램을 컴파일하고 메모리의 할당된 부분 뒤를 읽음으로써 out-of-bounds 읽기를 유발한다.

```python
with open("program.c", "w") as f:
    f.write("""
#include <stdlib.h>
#include <string.h>

int main(int argc, char** argv) {
    /* Create an array with 100 bytes, initialized with 42 */
    char *buf = malloc(100);
    memset(buf, 42, 100);

    /* Read the N-th element, with N being the first command-line argument */
    int index = atoi(argv[1]);
    char val = buf[index];

    /* Clean up memory so we don't leak */
    free(buf);
    return val;
}
    """)
```

```python
from bookutils import print_file
```

```python
print_file("program.c")
```

```c
#include <stdlib.h>
#include <string.h>

int main(int argc, char** argv) {
    /* Create an array with 100 bytes, initialized with 42 */
    char *buf = malloc(100);
    memset(buf, 42, 100);

    /* Read the N-th element, with N being the first command-line argument */
    int index = atoi(argv[1]);
    char val = buf[index];

    /* Clean up memory so we don't leak */
    free(buf);
    return val;
}
```

이 C 프로그램을 address sanitization을 활성화한 상태로 컴파일한다.

```bash
!clang -fsanitize=address -g -o program program.c
```

만약 인자 99를 가지고 프로그램을 실행하면, buf[99]가 리턴되고 이 값은 42이다.

```bash
!./program 99; echo $?
```

하지만, buf[110]에 접근하는 것은 AddressSanitizer에서 out-of-bounds 의 결과를 낸다.

```bash
!./program 110
```

> ```bash
> =================================================================
> ==72172==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x000104202a9e at pc 0x0001041abe84 bp 0x00016bc56680 sp 0x00016bc56678
> READ of size 1 at 0x000104202a9e thread T0
>     #0 0x1041abe80 in main program.c:12
>     #1 0x181c310dc  (<unknown module>)
> 
> 0x000104202a9e is located 10 bytes after 100-byte region [0x000104202a30,0x000104202a94)
> allocated by thread T0 here:
>     #0 0x104a57244 in wrap_malloc+0x94 (libclang_rt.asan_osx_dynamic.dylib:arm64e+0x53244)
>     #1 0x1041abdc8 in main program.c:7
>     #2 0x181c310dc  (<unknown module>)
> 
> SUMMARY: AddressSanitizer: heap-buffer-overflow program.c:12 in main
> Shadow bytes around the buggy address:
>   0x000104202800: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202880: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202900: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202980: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202a00: fa fa fa fa fa fa 00 00 00 00 00 00 00 00 00 00
> =>0x000104202a80: 00 00 04[fa]fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202b00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202b80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202c00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202c80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
>   0x000104202d00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
> Shadow byte legend (one shadow byte represents 8 application bytes):
>   Addressable:           00
>   Partially addressable: 01 02 03 04 05 06 07 
>   Heap left redzone:       fa
>   Freed heap region:       fd
>   Stack left redzone:      f1
>   Stack mid redzone:       f2
>   Stack right redzone:     f3
>   Stack after return:      f5
>   Stack use after scope:   f8
>   Global redzone:          f9
>   Global init order:       f6
>   Poisoned by user:        f7
>   Container overflow:      fc
>   Array cookie:            ac
>   Intra object redzone:    bb
>   ASan internal:           fe
>   Left alloca redzone:     ca
>   Right alloca redzone:    cb
> ==72172==ABORTING
> ```

C 프로그램에서 에러를 찾기를 원한다면, 퍼징을 위해 이러한 검사를 켜놓는 것은 꽤나 쉽다. 도구에 따른 특정 요인이 실행의 속도를 늦추고(AddressSanitizer의 경우 보통 2배), 또한 더 많은 메모리를 소비하지만, 이러한 버그를 찾기 위해 들어가는 인간의 노력에 비교하면 CPU 사이클은 매우 싸다.

공격자들을 위한 것이 아닌 메모리에 접근하거나 심지어 정보를 수정하게 할 수 있기 때문에 메모리에 대한 Out-of-bounds 접근은 대단한 보안적 위협이다. 유명한 예제로, [HeartBleed bug](https://en.wikipedia.org/wiki/Heartbleed)는 OpenSSL 라이브러리 내의 보안 버그로, OpenSSL 라이브러리는 컴퓨터 네트워크를 통해 통신 보안을 제공하는 암호화 프로토콜을 구현한다(만약 여러분이 브라우저에서 이 텍스트를 읽는 경우 이러한 프로토콜을 사용하여 암호화되었을 가능성이 높다).

HeartBleed 버그는 SSL heartbeat 서비스에 특별히 제작되 명령을 전송함으로써 악용된다. heartbeat 서비스는 다른 한쪽의 서버가 여전히 살아있는지 검사하는데에 사용된다. client는 다음과 같은 문자열을 서비스에 전송한다

> BIRD(4 letters)

어떤 서버가 BIRD에 대해 답하는지 확인하고, client는 서버가 살아있는지 알게 된다.

운 나쁘게도, 이 서비스는 요청된 문자들의 집합보다 더 많이 답하도록 서버에 요청함으로써 악용될 수 있다. 이는 다음 [XKCD comic](https://xkcd.com/1354/)에서 매우 잘 설명되어 있다.

![image](https://github.com/fault2000/Fuzzing_book/assets/73513005/975cda89-489a-47ef-8a2b-bc1cffcd0f2c)

OpenSSL 구현에서, 이러한 메모리 내용은 암호화 인증서, 개인 키, 그리고 더한 것들이 포함되어 있을 수 있으며 - 최악인 점은, 이 메모리가 방금 접근되어졌음을 아무도 알아차리지 못한다는 것이다. HeartBleed 버그가 발견되었을 때, 이 버그는 오랫동안 존재해왔으며, 아무도 어떤 비밀이 새어나갔는지, 새어나가긴 했는지 알지 못했다; 빠르게 설정된 [HeartBleed announcement page](http://heartbleed.com/)가 모든 것을 말해준다.

하지만 어떻게 HeartBleed가 발견되었을까? 매우 간단하다. codenomicon과 구글의 연구원들은 OpenSSL 라이브러리를 address Sanitizer로 컴파일한 다음 fuzz된 명령어로 실행했다. 그 다음 memory sanitizer는 out-of-bound 메모리 접근이 발생되었음을 알아차렸다 - 그리고 실제로, 그것은 버그를 매우 빠르게 발견했다.

메모리 검사기는 fuzzing동안 런타임 오류를 감지하기 위해 실행하는 수많은 검사기 중 하나일 뿐이다. [chapter on mining function](/part4/Mining%20Function%20Specifications.md)에서, 일반 검사기를 정의하는 법에 대해 더 배울 것이다.

program에 볼 일은 끝났으므로, 치워놓자.

```bash
!rm -fr program program.*
```

#### Information Leaks

정보 유출은 불법적인 메모리 접근을 통해서만 일어나진 않는다; 그들은 "유효한" 메모리 내부에서 일어날 수도 있다 - 만약 이 "유효한" 메모리가 유출되지 않아야 하는 중요한 정보들을 포함하고 있다면 말이다. 파이썬 프로그램에서 이 문제를 표현해보자. 시작하기 위해, 실제 데이터와 무작위 데이터로 채워진 프로그램 메모리를 생성해보자.

```python
secrets = ("<space for reply>" + fuzzer(100) +
           "<secret-certificate>" + fuzzer(100) +
           "<secret-key>" + fuzzer(100) + "<other-secrets>")
```

더 많은 "메모리" 문자들을 secrets에 추가한다. 이 문자들은 초기화되지 않은 메모리에 대한 표시로 "deadbeef"로 채워진다:

```python
uninitialized_memory_marker = "deadbeef"
while len(secrets) < 2048:
    secrets += uninitialized_memory_marker
```

되돌려 받을 답장과 답장의 길이를 작성하는 (위에서 논의한 heartbeat 서비스와 비슷한)서비스를 정의한다. 이 서비스는 메모리 내에 보낼 답장을 저장하고, 주어진 길이에 따라 답장을 되돌려 보내준다.

```python
def heartbeat(reply: str, length: int, memory: str) -> str:
    # Store reply in memory
    memory = reply + memory[len(reply):]

    # Send back heartbeat
    s = ""
    for i in range(length):
        s += memory[i]
    return s
```

이는 평범한 문자열에 완벽히 작동한다:

```python
heartbeat("potato", 6, memory=secrets)
```

> 'potato'

```python
heartbeat("bird", 4, memory=secrets)
```

> 'bird'

하지만, 만약 답장의 길이가 답장할 문자열의 길이보다 크다면, 메모리의 추가적인 내용이 유출될 것이다. 이러한 모든 것들이 기존 배열 범위 내에서 여전히 일어나서 address sanitizer가 촉발되지 않음을 기억해두자:

```python
heartbeat("hat", 500, memory=secrets)
```

> 'hatace for reply>#,,!3?30>#61)\$4--8=<7)4 )03/%,5+! "4)0?.9+?3();\<42?=?0\<secret-certificate\>7(+/+((1)#/0\\'4!\>/\<#=78%6\$!!\$<-"3"\\'-?1?85!05629%/); *)1\\'/=9%\<secret-key\>.(#.4%\<other-secrets\>deadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadb'

이러한 문제는 어떻게 감지할까? 아이디어는 주어진 비밀이나 초기화되지 않은 메모리 같은 유출되어선 안 되는 정보를 식별화는 것이다. 작은 파이썬 예제에서 이러한 검사를 시뮬레이션해볼 수 있다.

```python
from ExpectError import ExpectError
```

```
with ExpectError():
    for i in range(10):
        s = heartbeat(fuzzer(), random.randint(1, 500), memory=secrets)
        assert not s.find(uninitialized_memory_marker)
        assert not s.find("secret")
```

> ```bash
> Traceback (most recent call last):
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/4040656327.py", line 4, in <cell line: 1>
>     assert not s.find(uninitialized_memory_marker)
> AssertionError (expected)
> ```

이러한 검사를 통해, 유출된 비밀 그리고/혹은 초기화되지 않은 메모리를 찾을 수 있다. [chapter on information flow](/Part4/Tracking%20Information%20Flow.md)에서 이를 자동으로 수행하는 방법과 민감한 정보와 그로부터 파생된 값을 "tainting"하고 "tainted" 값이 새어나가지 않도록 하는 방법에 대해 논의한다.

가장 중요한 것은, 언제나 fuzzing 동안 가능한 한 많은 자동 검사기를 활성화시켜야 한다. CPU cycle은 싸고 에러는 비싸다. 만약 실제로 에러를 감지하는 것 없이 오직 프로그램을 실행만 시킨다면, 여러분은 여러 기회들을 놓칠 것이다.

### Program-Specific Checkers

특정 플랫폼 또는 특정 언어의 모든 프로그램에 적용되는 일반 검사기 외에도 프로그램 또는 하위 시스템에 적용되는 특정 검사기를 고안할 수 있다. [chapter on testing](/Part1/Introduction%20to%20Software%20Testing.md)에서, 런타임 때에 정확성을 위해 함수 결과를 검사하는 [runtime verification](/Part1/Introduction%20to%20Software%20Testing.md)의 기술에서 이미 힌트를 얻었다.

에러를 감지하는 주요 아이디어 중 하나는 assertion의 개념이다. assertion은 중요한 함수의 입력(전제조건)과 결과(후조건)을 검사하는 속성이다. 프로그램 내에 더 많은 assertion을 가질수록, 일반적인 검사기에 의해 감지되지 않은 에러를 실행 도중에 감지할 확률이 커진다. - 특히 fuzzing 중에는, 만약 여러분이 assertion이 성능에 끼칠 영향이 걱정된다면, 생산 코드 내에선 assertion을 꺼둘 수 있다는 것을 기억하자(그럼에도 불구하고 대부분의 중요한 검사는 활성화해둔채로 놔두는 것이 도움이 될 것이다).

에러를 찾기 위한 assertion의 가장 중요한 사용처 중 하나는 복잡한 데이터 구조의 무결성을 검사하는 것이다. 간단한 예제를 사용해서 이 개념을 살펴보자. 다음과 같이 공항 코드와 공항을 매핑한다고 가정해보자

```python
airport_codes: Dict[str, str] = {
    "YVR": "Vancouver",
    "JFK": "New York-JFK",
    "CDG": "Paris-Charles de Gaulle",
    "CAI": "Cairo",
    "LED": "St. Petersburg",
    "PEK": "Beijing",
    "HND": "Tokyo-Haneda",
    "AKL": "Auckland"
}  # plus many more
```

```python
airport_codes["YVR"]
```

> 'Vancouver'

```python
"AKL" in airport_codes
```

> True

이 공항 코드의 리스트는 꽤나 중요할 수 있다: 만약 공항 코드 중 하나에서 철자를 잘못 쓴다면, 이는 우리가 가지고 있는 응용 프로그램에 영향을 미칠 수 있다. 따라서 우리는 일관성을 위해 리스트를 검사하는 함수를 소개한다. 일관성 조건은 representation invariant로 불리며, 이를 검사하는 함수(혹은 메소드)는 일반적으로 "the representation is ok"는 의미로 repOK()로 명명된다.

먼저, 개별적인 공항 코드를 위한 검사기를 가져보자. 검사기는 코드가 일관적이지 않다면 실패한다.

```python
def code_repOK(code: str) -> bool:
    assert len(code) == 3, "Airport code must have three characters: " + repr(code)
    for c in code:
        assert c.isalpha(), "Non-letter in airport code: " + repr(code)
        assert c.isupper(), "Lowercase letter in airport code: " + repr(code)
    return True
```

```python
assert code_repOK("SEA")
```

이제 code_repOK()를 사용해 리스트 내의 모든 요소를 검사할 수 있다.

```python
def airport_codes_repOK():
    for code in airport_codes:
        assert code_repOK(code)
    return True
```

```python
with ExpectError():
    assert airport_codes_repOK()
```

만약 우리가 리스트에 유효하지 않은 요소를 추가하면, 우리의 검사는 실패할 것이다:

```python
airport_codes["YMML"] = "Melbourne"
```

```python
with ExpectError():
    assert airport_codes_repOK()
```

> ```
> Traceback (most recent call last):
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/2308942452.py", line 2, in <cell line: 1>
>     assert airport_codes_repOK()
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/480627665.py", line 3, in airport_codes_repOK
>     assert code_repOK(code)
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/865020192.py", line 2, in code_repOK
>     assert len(code) == 3, "Airport code must have three characters: " + repr(code)
> AssertionError: Airport code must have three characters: 'YMML' (expected)

물론, 리스트를 직접 조작하는 것보다, 요소를 추가하기 위한 특별한 함수를 가질 것이다; 이는 또한 코드가 유효한지 검사할 수 있을 것이다:

```python
def add_new_airport(code: str, city: str) -> None:
    assert code_repOK(code)
    airport_codes[code] = city
```

```python
with ExpectError():  # For BER, ExpectTimeout would be more appropriate
    add_new_airport("BER", "Berlin")
```

이는 인자 리스트 내의 오류를 찾아낼 수 있도록 한다:

```python
with ExpectError():
    add_new_airport("London-Heathrow", "LHR")
```

> ```
> Traceback (most recent call last):
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/1427835309.py", line 2, in <cell line: 1>
>     add_new_airport("London-Heathrow", "LHR")
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/2655039924.py", line 2, in add_new_airport
>     assert code_repOK(code)
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/865020192.py", line 2, in code_repOK
>     assert len(code) == 3, "Airport code must have three characters: " + repr(code)
> AssertionError: Airport code must have three characters: 'London-Heathrow' (expected)

그렇지만 최대한의 검사를 위해, add_new_airport() 함수는 공항 코드 리스트를 바꾸기 전과 바꾼 후에 정확한 표시를 보장해야 한다.

```python
def add_new_airport_2(code: str, city: str) -> None:
    assert code_repOK(code)
    assert airport_codes_repOK()
    airport_codes[code] = city
    assert airport_codes_repOK()
```

이는 미리 소개된 비일관성을 잡아낼 수 있다.

```python
with ExpectError():
    add_new_airport_2("IST", "Istanbul Yeni Havalimanı")
```

> ```
> Traceback (most recent call last):
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/4151824846.py", line 2, in <cell line: 1>
>     add_new_airport_2("IST", "Istanbul Yeni Havalimanı")
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/2099116665.py", line 3, in add_new_airport_2
>     assert airport_codes_repOK()
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/480627665.py", line 3, in airport_codes_repOK
>     assert code_repOK(code)
>   File "/var/folders/n2/xd9445p97rb3xh7m1dfx8_4h0006ts/T/ipykernel_72058/865020192.py", line 2, in code_repOK
>     assert len(code) == 3, "Airport code must have three characters: " + repr(code)
> AssertionError: Airport code must have three characters: 'YMML' (expected)
> ```

더 많은 repOK() assertion이 코드 내에 있을수록, 잡아내는 에러는 더 많아진다 - 심지어 이들은 오직 여러분의 도메인과 문제에만 특화되어 있다. 또한 이러한 assertion은 프로그래밍 동안 여러분이 한 가정들을 문서화시키고 따라서 다른 개발자들이 당신의 코드를 이해하고 에러를 방지하는 것을 돕는다.

마지막 예제로, 복잡한 데이터 구조인 [red-black tree](https://en.wikipedia.org/wiki/Red-black_tree)를 고려한다. red-black tree는 스스로 균형을 잡는 바이너리 탐색 트리이다. red-black tree를 구현하는 것은 너무 어렵지 않지만, 정확히 구현하는 것은 숙련된 개발자에게도 시간을 잡아먹는 작업일 수 있다. 하지만 repOK() 메소드는 모든 가정을 문서화하고 또한 그들을 검사한다:

```python
class RedBlackTree:
    def repOK(self):
        assert self.rootHasNoParent()
        assert self.rootIsBlack()
        assert self.rootNodesHaveOnlyBlackChildren()
        assert self.treeIsAcyclic()
        assert self.parentsAreConsistent()
        return True

    def rootIsBlack(self):
        if self.parent is None:
            assert self.color == BLACK
        return True

    def add_element(self, elem):
        assert self.repOK()
        ...  # Add the element
        assert self.repOK()

    def delete_element(self, elem):
        assert self.repOK()
        ...  # Delete the element
        assert self.repOK()
```

여기서 repOK()는 RedBlackTree 클래스의 객체에서 실행되는 메서드이다. 이는 다섯 가지 서로다른 검사를 실행하며, 그들 모두는 자신 만의 assertion을 가진다. 요소가 추가되거나 삭제될 때마다, 이 모든 일관성 검사가 자동적으로 실행된다. 만약 이 검사들 중 하나라도 오류를 가진다면, 검사기는 그들을 찾아낼 것이다 - 물론 이는 여러분이 충분히 많은 fuzz된 입력을 통해 tree를 실행했을 때의 말이다.

### Static Code Checkers

repOK() assertion을 통해 얻을 수 있는 많은 이득들은 여러분 코드의 정적 유형 검사기를 통해서도 얻을 수 있다. 예를 들어 파이썬에선, MyPy 정적 검사기는 인자의 유형이 적절히 정의되자마자 유형 오류를 찾을 수 있다:

```python
typed_airport_codes: Dict[str, str] = {
    "YVR": "Vancouver", # etc
}
```

만약 다음처럼 문자열이 아닌 유형으로 key를 추가하면

```python
typed_airport_codes[1] = "First"
```

이는 MyPy에 의해 즉시 잡힐 것이다:

```bash
$ mypy airports.py
airports.py: error: Invalid index type "int" for "Dict[str, str]"; expected type "str"
```

정확히 3개의 대문자로 이루어진 공항 코드 혹은 비순환적인 tree같은 더 고급 속성을 정적으로 검사하는 것은 정적 검사의 한계에 빠르게 도달할 것이다. 여러분의 repOK() assertions은 여전히 필요할 것이며 좋은 검사 생성기와 결합될 때 최고일 것이다.

## A Fuzzing Architecture

이 장에 일부 부분을 나중에 재사용하고 싶기 때문에, 재사용하기 쉽고, 특히 확장하기 쉬운 방향으로 일부 부분을 정의할 것이다. 이를 위해, 재사용 가능한 방향으로 위 기능을 캡슐화하는 여러 클래스들을 소개한다.

### Runner Classes

먼저 소개할 것은 Runner의 개념이다 - Runner는 주어진 입력에 따라 어떤 객체를 실행하는 것이 목적인 객체이다. runner는 보통 검사가 진행중인 프로그램 혹은 함수이지만, 우리는 더 간단한 runner를 가질 수도 있다.

runner의 기본 클래스부터 시작하자. runner는 runner에게 input(문자열)을 건네주는데 사용되는 run(input) 메소드를 필수적으로 제공한다. run()은 (result, outcome)의 pair를 return한다. 여기서, result는 실행에 대한 세부사항을 주는 runner에 특화된 값이다; outcome은 3종류로 결과를 분류하는 값이다:

- Runner.PASS - 검사가 통과됨. 실행은 정확한 결과를 생산함.
- Runner.FAIL - 검사가 실패함. 실행은 정확하지 않은 결과를 생산함.
- Runner.UNRESOLVED - 검사는 통과되지도 실패하지도 않음. 이는 만약 실행을 수행할 수 없었으면 발생함. 예를 들어, 입력이 유효하지 않았을 경우.

```python
Outcome = str
```

```python
class Runner:
    """Base class for testing inputs."""

    # Test outcomes
    PASS = "PASS"
    FAIL = "FAIL"
    UNRESOLVED = "UNRESOLVED"

    def __init__(self) -> None:
        """Initialize"""
        pass

    def run(self, inp: str) -> Any:
        """Run the runner with the given input"""
        return (inp, Runner.UNRESOLVED)
```

기본 클래스로서 Runner는 이를 기반으로 구축되는 보다 복잡한 runner를 위한 인터페이스를 제공할 뿐이다. 더 자세히는, 추가적인 메소드를 추가하거나 상속된 메소드를 오버라이드하기 위해 그들의 부모클래스로부터 메소드를 상속받는 자식클래스를 소개한다.

이러한 자식클래스의 한 예시를 보자: PrintRunner는 상속받은 run() 메소드를 오버라이드하여 받은 모든 것들을 단순히 출력한다. 이는 많은 상황에서 기본 runner이다.

```python
class PrintRunner(Runner):
    """Simple runner, printing the input."""

    def run(self, inp) -> Any:
        """Print the given input"""
        print(inp)
        return (inp, Runner.UNRESOLVED)
```

```python
p = PrintRunner()
(result, outcome) = p.run("Some input")
```

> Some input

결과는 우리가 입력으로 넘겨주었던 문자열이다.

```python
result
```

> 'Some input'

그러나 이 시점에선, 프로그램의 동작을 구별할 수 있는 방법은 없다:

```python
outcome
```

> 'UNRESOLVED'

ProgramRunner 클래스는 프로그램의 입력을 프로그램의 표준 입력으로 대신 보낸다. 프로그램은 ProgramRunner 객체를 생성할 때 지정된다.

```python
class ProgramRunner(Runner):
    """Test a program with inputs."""

    def __init__(self, program: Union[str, List[str]]) -> None:
        """Initialize.
           `program` is a program spec as passed to `subprocess.run()`"""
        self.program = program

    def run_process(self, inp: str = "") -> subprocess.CompletedProcess:
        """Run the program with `inp` as input.
           Return result of `subprocess.run()`."""
        return subprocess.run(self.program,
                              input=inp,
                              stdout=subprocess.PIPE,
                              stderr=subprocess.PIPE,
                              universal_newlines=True)

    def run(self, inp: str = "") -> Tuple[subprocess.CompletedProcess, Outcome]:
        """Run the program with `inp` as input.  
           Return test outcome based on result of `subprocess.run()`."""
        result = self.run_process(inp)

        if result.returncode == 0:
            outcome = self.PASS
        elif result.returncode < 0:
            outcome = self.FAIL
        else:
            outcome = self.UNRESOLVED

        return (result, outcome)
```

여기에 바이너리(즉, 텍스트가 아닌) 입력 및 출력을 위한 변형이 있다.

```python
class BinaryProgramRunner(ProgramRunner):
    def run_process(self, inp: str = "") -> subprocess.CompletedProcess:
        """Run the program with `inp` as input.  
           Return result of `subprocess.run()`."""
        return subprocess.run(self.program,
                              input=inp.encode(),
                              stdout=subprocess.PIPE,
                              stderr=subprocess.PIPE)
```

cat 프로그램을 통해 ProgramRunner를 시연해보자 - cat 프로그램은 입력을 출력에 복사한다. 우리는 cat의 표준 호출이 단순히 작업을 수행하고, cat의 출력은 입력과 동일함을 알 수 있다.

```python
cat = ProgramRunner(program="cat")
cat.run("hello")
```

> (CompletedProcess(args='cat', returncode=0, stdout='hello', stderr=''), 'PASS')

### Fuzzer Classes