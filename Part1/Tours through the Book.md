# Tours through the Book

이 책은 방대하다. 무려 20000줄의 코드와 15만개의 텍스트 이상으로 이루어져 있으며 출판된 버전은 1200 페이지 이상이다. 명백히도, 우리는 모두가 모든 것을 읽기 원한다고 가정하지 않는다.  
이 책의 장은 차례로 읽을 수 있기 때문에, 이 책을 읽는 경로는 다양하다. 이 그래프에서, 화살표 A -> B는 A장은 B장을 위한 준비물이라는 뜻이다. 당신이 가장 관심있는 주제에 대해 읽기 위해 이 그래프의 임의의 경로를 선택할 수 있다.  

<img width="701" alt="image" src="https://github.com/fault2000/Fuzzing_book/assets/73513005/b8434c0b-3317-4aba-ab2a-5334e5ebdefa">

하지만 이 맵도 압도적으로 느낄 수 있기 때문에, 여기 당신이 시작하기에 알맞은 몇몇 투어들이 있다. 이 각 투어들은 프로그래머, 학생 혹은 연구원 같은 특정 시점에 집중할 수 있도록 해준다.  

## The Pragmatic Programmer Tour

여러분에겐 테스트해봐야 할 프로그램이 있다. 이 프로그램에 대한 가능한 한 빠르고 철저한 테스트를 생성하기를 원한다. 여러분은 어떤 것이 어떻게 구현되는지에 대해 그렇게 신경 쓰지 않지만, 그것은 그 일을 해낼 것이다. 여러분은 어떻게 사용하는지를 배우고 싶어한다.  

1. 기본 개념을 잡기 위해 [Introduction to Testing](/Part1/Introduction%20to%20Software%20Testing.md)부터 시작하자.(아마 이미 대부분의 내용을 알고 있을 테지만, 아무튼 잠깐 상기하는 것은 나쁘지 않다)
2. 당신의 프로그램을 첫 임의의 입력으로 테스트해보기 위해 [Fuzzer에 대한 장](/Part2/Fuzzing:%20Breaking%20Things%20with%20Random%20Inputs.md)에서의 간단한 fuzzer를 사용해보자.
3. 프로그램으로부터 [coverage](/Part2/Code%20Coverage.md)를 얻고, 이 coverage 정보를 사용해 [테스트 생성기를 code coverage로 안내한다](/Part2/Greybox%20Fuzzing.md).
4. [입력 문법](/Part3/Fuzzing%20with%20Grammars.md)를 프로그램에 정의하고 이 문법을 사용해 구문적으로 정확한 입력으로 철저하게 당신의 프로그램을 fuzz할 수 있다. fuzzer로서, 입력 요소의 범위를 보장하기 위해 [문법 coverage fuzzer](/Part3/Grammar%20Coverage.md)를 추천한다.
5. 만약 당신이 생성된 입력에 대한 더 많은 통제를 원한다면, [probabilistic fuzzing](/Part3/Probabilistic%20Grammar%20Fuzzing.md)과 [fuzzing with generator functions](/Part3/Fuzzing%20with%20Generators.md)를 고려하라.
6. 만약 fuzzer의 많은 집합을 적용하고 싶다면, 어떻게 [manage a large set of fuzzers](/Part6/Fuzzing%20in%20the%20Large.md)를 하는지 배워라.

이러한 각 장에서, "개요" 부분부터 시작하자; 이들은 어떻게 사용하는지에 대한 빠른 소개를 줄 것이고, 또한 관련된 사용 예시를 보여줄 것이다. 이 정도면 충분하다. 일하러 돌아가서 즐겨라!

## The Page-by-Page Tours

이 투어는 어떻게 이 책이 구성되는지이다. 테스팅에 대한 기본 개념을 소개받았다면, 다음 부분을 통해 방법을 읽을 수 있다.

1. [lexical tour](/Part2/Lexical%20Fuzzing.md)는 어휘적 테스트 생성 기법, 즉 입력을 문자 단위와 바이트 단위로 구성하는 기술에 집중한다. 최소한의 bias로 매우 빠르고 튼튼한 기술이다.
2. [syntactical tour](/Part3/Syntactic%20Fuzzing.md)는 입력 구문을 지정하는 수단으로 문법에 중점을 둔다. 테스트 생성기의 결과는 구문적으로 정확한 입력을 생성하여 테스트를 훨씬 빠르게 만들고, 테스터기를 위한 많은 제어 메커니즘을 제공한다.
3. [semantic tour](/Part4/Semantic%20Fuzzing.md)는 code semantic을 활용하여 테스트 생성의 모양을 잡고 안내한다. 입력 문법을 추출하거나 함수 세부사항을 캐거나, symbolic 제약 조건 해결 증이 포함되어 가능한 한 많은 코드 경로를 cover하려고 한다.
4. [application tour](/Part5/Domain-Specific%20Fuzzing.md)는 이전 부분에서 정의한 기술을 웹 서버, 유저 인터페이스, APIs 혹은 configurations같은 도메인에 적용한다.
5. [management tour](/Part6/Managing%20Fuzzing.md)는 마침내 테스트 생성기의 큰 집합을 어떻게 다루고 정리하는지, fuzzing을 언제 멈춰야 하는지에 대해 집중한다.

이러한 장의 대부분은 가장 중요한 개념을 어떻게 사용하는지 설명하는 "개념" 부분으로 시작한다. 