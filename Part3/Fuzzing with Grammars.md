# Fuzzing with Grammars

["Mutation-Based Fuzzing"](/Part2/Mutation-Based%20Fuzzing.md) 장에서는 샘플 입력 파일과 같은 추가 힌트를 사용하여 테스트 생성 속도를 높이는 방법에 대해 알아보았다. 이 장에선, 프로그램에 합법적 입력의 세부사항을 제공함으로써 이 아이디어를 가지고 한 걸음 더 나아갈 것이다. 문법을 통해 입력을 정의하는 것은 특히 복잡한 입력 포맷의 경우에 매우 체계적이고 효율적인 테스트 생성을 할 수 있게 한다. 문법은 fuzzing, API fuzzing, GUI fuzzing, 그리고 더 많은 경우의 기본처럼 다뤄질 수 있다.

```python
from bookutils import YouTubeVideo
YouTubeVideo('Jc8Whz0W41o')
```

[![Fuzzing with Grammars](https://img.youtube.com/vi/Jc8Whz0W41o/0.jpg)](https://www.youtube.com/watch?v=Jc8Whz0W41o)

**준비물**

- 기본적인 fuzzing 작동방식을 알아야 한다, 예를 들어, [fuzzing을 소개하는 챕터](/Part2/Fuzzing,%20Breaking%20Things%20with%20Random%20Inputs.md)로부터.
- [mutation-based fuzzing](/Part2/Mutation-Based%20Fuzzing.md)과 [coverage](/Part2/Code%20Coverage.md)에 대한 지식은 아직 필요하진 않지만, 여전히 추천한다.

```python
import bookutils.setup
```

```python
from typing import List, Dict, Union, Any, Tuple, Optional
```

```python
import Fuzzer
```

## Synopsis

이 장에서 제공하는 코드를 사용하기 위해, 다음을 쓰자

```python
from fuzzingbook.Grammars import <identifier>
```

그다음 다음 기능들을 쓸 수 있다.

이 장에선 문법을 입력 언어를 명시하는 간단한 의미로 소개하고, 구문적으로 유효한 입력으로 프로그램을 테스팅하는 경우에 그들을 쓴다. 문법은 다음 예제와 같이 터미널이 아닌 기호를 대체 확장 목록에 매핑하는 것으로 정의된다.

```python
US_PHONE_GRAMMAR: Grammar = {
    "<start>": ["<phone-number>"],
    "<phone-number>": ["(<area>)<exchange>-<line>"],
    "<area>": ["<lead-digit><digit><digit>"],
    "<exchange>": ["<lead-digit><digit><digit>"],
    "<line>": ["<digit><digit><digit><digit>"],
    "<lead-digit>": ["2", "3", "4", "5", "6", "7", "8", "9"],
    "<digit>": ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
}

assert is_valid_grammar(US_PHONE_GRAMMAR)
```

터미널이 아닌 기호는 각 괄호(예로, \<digit\>)에 둘러싸여 있다. 문법으로부터 입력 문자열을 생성하기 위해, 공급자는 시작 symbol인 (\<start\>)로 시작해서 이 symbol에 대해 임의 확장을 무작위로 선택한다. 이는 계속해서 처리되어 마침내 모든 nonterminal symbol이 확장될 때까지 지속된다. simple_grammar_fuzzer() 함수는 그 일을 수행한다:

```python
>>> [simple_grammar_fuzzer(US_PHONE_GRAMMAR) for i in range(5)]
['(692)449-5179',
 '(519)230-7422',
 '(613)761-0853',
 '(979)881-3858',
 '(810)914-5475']
```

그렇지만 실질적으로 simple_grammar_fuzzer() 대신, 여러분은 GrammarFuzzer class를 사용하거나 혹은 [coverage-based](/Part3/Grammar%20Coverage.md), [probabilistic-based](/Part3/Probabilistic%20Grammar%20Fuzzing.md), 혹은 [generator-based](/Part3/Fuzzing%20with%20Generators.md) 파생 중 하나를 써야만 한다; 이들이 훨씬 더 효율적이고, 무한 성장을 막으며, 여러 추가적인 기능을 제공한다.

이 장에서는 문법을 쓰는데 도움이 되는 여러 함수, 예를 들어 문자 종류나 반복에 바로 가지 표시법을 사용하거나 문법을 확장하는 것 같은 함수를 포함한 [grammar toolbox](#a-grammar-toolbox)를 소개한다.

## Input Languages

프로그램이 사용가능한 모든 행동은 그것의 입력으로부터 발생한다. 여기서 "입력"은 사용가능한 다양한 원인이 될 수 있다: 여기서 말하는 것은 파일, 환경, 네트워크 너머로부터 유저에 의해 입력되거나 다른 자원과 상호작용하여 얻어진 데이터에 대해 얘기하는 것이다. 이러한 모든 입력의 집합은 그것의 실패를 포함해 프로그램이 어떻게 행동하는지를 결정한다. 따라서 테스트할 때 가능한 입력 소스, 제어 가능한 방법 및 체계적인 테스트 방법을 생각해 보는 것이 매우 도움이 된다.

간단성을 위해서, 프로그램이 입력 소스가 오직 하나밖에 없다고 가정할 것이다; 이것은 이전 장에서 우리가 사용한 가정과 동일하다. 프로그램에 유효한 입력의 집합은 *언어*라고 불린다. 언어는 간단한 것부터 복잡한 것까지 다양하다: CSV 언어는 유효한 쉼표로 구분된 입력을 가리키고, 반면 파이썬 언어는 유효한 파이썬 프로그램의 집합을 나타낸다. 우리는 흔히 데이터 언어와 프로그래밍 언어를 분리하지만, 그럼에도 불구하고 모든 프로그램은 입력 데이터(예를 들어, 컴파일러)로 취급될 수 있다. [파일 형식에 대한 Wikipedia 페이지](https://en.wikipedia.org/wiki/List_of_file_formats)는 1,000개 이상의 서로다른 파일 형식의 리스트를 작성해놓았고 각 형식은 고유의 언어를 가진다.

공식으로 언어를 설명하기 위해, 공식 언어의 영역은 언어를 설명하는 많은 언어 설명서를 고안했다. 정규식은 문자열 집합을 나타내는 이러한 언어의 가장 단순한 종류를 나타낸다. 예를 들어, 정규식 [a-z]\*은 소문자의 sequence를 나타낸다. *Automata theory*는 이러한 입력을 허락하는 오토마타에 이러한 언어를 연결한다; 예를 들어, *유한 상태 기계(finite state machine)*은 정규 식의 언어를 명시하는데 사용할 수 있다.

정규식은 너무 복잡하지 않은 입력 형식의 경우 좋으며, 연관된 유한 상태 기계는 그들을 추론하기 좋게 만들어 주는 많은 속성을 가지고 있다. 그렇지만 더 복잡한 입력을 명시하기 위해, 그들은 빠르게 한계에 직면한다. 언어 스펙트럼의 다른 끝에는 튜링 기계가 수용하는 언어를 나타내는 범용 문법이 있다. 튜링 기계는 계산될 수 있는 모든 것들을 계산할 수 있으며, 파이썬이 Turing-complete하다면 파이썬 프로그램 p를 사용해 합법적 입력을 정의하거나 열거할 수도 있음을 뜻한다. 하지만 컴퓨터 공학 이론은 이러한 검사 프로그램이 검사될 프로그램에 대해 특별히 작성되어야 함을 말해주며, 이는 우리가 원하는 자동화 수준이 아니다.

## Grammars

정규식과 Turing machine 사이의 중간 부분은 문법으로 덮여 있다. 문법은 가장 유명한(그리고 제일 이해하기 쉬운) 형식주의 중 하나로 입력 언어를 공식으로 정의한다. 문법을 사용하면 입력 언어의 넓은 범위의 속성을 나타낼 수 있다. 문법은 입력의 구문적인 구조를 표현하는데 특히 좋으며 인접하거나 재귀적 입력을 나타내기 위해 선택하는 형식주의이다. 우리가 사용하는 문법은 소위 context-free grammars라고 불리며 가장 쉽고 가장 유명한 문법 형식주의 중 하나이다.

### Rules and Expansions

문법은 start symbol과 이 start symbol(그리고 다른 symbol)이 어떻게 확장될지 나타내는 expansion rules(혹은 simply rules)의 집합으로 이루어진다. 예시로 2가지 숫자의 sequence를 의미하는 다음 문법을 보자.

> \<start\> ::= \<digit\>\<digit\>
> \<digit\> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

이러한 문법을 읽기 위해선 start symbol(\<start\>부터 시작한다. expansion rule \<A\> ::= \<B\>는 좌측 symbol (\<A\>)가 우측 문자열 (\<B\>)로 대체될 수 있음을 의미한다. 위 문법에서, \<start\>는 \<digit\>\<digit\>으로 대체될 수 있다.

다시 이 문자열 내에서, \<digit\>은 \<digit\> rule 우측에 위치한 문자열로 대체될 수 있다. 특수한 연산자인 | 은 확장을 위해 어떤 숫자든 선택될 수 있음을 의미하는 expansion alternatives(혹은 simply alternatives)를 의미한다. 따라서 각 \<digit\>은 주어진 숫자 중 하나로 확장될 수 있으며, 결국 00과 99 사이의 문자열을 산출할 것이다. 0에서 9까지는 더 이상에 expansion이 없기 때문에, 모든 준비가 끝났다.

문법과 관련된 흥미로운 점은 그들이 재귀적일 수 있다는 것이다. 이 말은 expansion은 이전에 확장된 symbol을 활용할 수 있으며 그러면 다시 한 번 확장될 수 있다. 예를 들어, 정수를 설명하는 다음 문법을 보자.

> \<start\> ::= \<integer\>
> \<integer\> ::= \<digit\> | \<digit\>\<integer\>
> \<digit\> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

여기서 \<integer\>은 하나의 숫자일수도 있고, 또다른 정수에 뒤이은 숫자일 수도 있다. 따라서 수 1234는 하나의 숫자 1과 그 뒤를 잇는 정수 234로 나타낼 수 있으며 이 정수 234 또한 숫자 2와 뒤이은 정수 34로 차례차례 나타낼 수 있다.

만약 정수 앞의 기호(+ or -)가 있을 수 있다는 것을 표현하고 싶다면 문법을 다음과 같이 쓸 것이다:

> \<start\> ::= \<number\>
> \<number\> ::= \<integer\> | +\<integer\> | -\<integer\>
> \<integer\> ::= \<digit\> | \<digit\>\<integer\>
> \<digit\> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

이 규칙은 언어를 정규적으로 정의한다: start symbol로부터 파생되는 모든 것들은 언어의 일부분이다; 파생될 수 없는 것들은 언어의 일부분이 아니다.

### Arithmetic Expressions

문법을 확장하여 전체 산술 표현인 문법에 대한 poster child 예제를 다루도록 해보자. (\<expr\>)식은 합 혹은 빼기 혹은 term이고; term는 곱셈 혹은 나눗셈 혹은 factor이고; factor은 숫자이거나 괄호로 묶은 식이다. 거의 모든 규칙이 재귀될 수 있으며, 따라서 임의의 복잡한 식, 예를 들어 $(1 + 2) * (3.4 / 5.6 - 789)$같은 것들도 허용한다.

> \<start\> ::= \<expr\>
> \<expr\> ::= \<term\> + \<expr\> | \<term\> - \<expr\> | \<term\>
> \<term\> ::= \<term\> * \<factor\> | \<term\> - \<factor\> | \<factor\>
> \<factor\> ::= +\<factor\> | -\<factor\> | (\<expr\>) | \<integer\> | \<integer\>.\<integer\>
> \<integer\> ::= \<digit\>\<integer\> | \<digit\>
> \<digit\> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

이러한 문법에선, 만약 우리가 \<start\>로부터 시작해 한 symbol을 차례로 무작위로 선택된 대체물로 확장하면, 유효한 산술식을 차례로 빠르게 생성할 수 있다. 이러한 문법 fuzzing은 복잡한 입력을 생산하는데 매우 효과적이며 이것이 우리가 이번 장에서 구현할 것들이다.

## Representing Grammars in Python

문법 fuzzer를 만들기 위한 첫 걸음은 문법에 적합한 형식을 찾는 것이다. 문법을 작성하는 것을 최대한 간단하게 만들기 위해, 문자열과 리스트에 기반을 둔 형식을 사용한다. 파이썬에서 우리의 문법은 symbol 이름과 expansion 간의 mapping의 형식을 가진다. 여기서 expansion은 대안의 리스트이다. 숫자를 위한 단일 규칙 문법은 따라서 이 형식을 가진다

```python
DIGIT_GRAMMAR = {
    "<start>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
}
```

<details>
<summary>A Grammar Type</summary>
<div markdown="1">

문법 유형(Grammar type)을 정적으로 확인할 수 있도록 문법의 유형을 정의해보자.

문법 유형의 첫 시도는 각 symbol이 expansion(문자열)의 리스트에 매핑된 것이다:

```python
SimpleGrammar = Dict[str, List[str]]
```



</div>
</details>

## A Grammar Toolbox