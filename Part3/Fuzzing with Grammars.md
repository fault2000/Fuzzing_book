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

터미널이 아닌 기호는 