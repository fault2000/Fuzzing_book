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

정규식은 너무 복잡하지 않은 입력 형식의 경우 좋으며, 연관된 유한 상태 기계는 추론하기 좋은 

## A Grammar Toolbox