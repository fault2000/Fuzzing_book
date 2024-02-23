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

이러한 장의 대부분은 가장 중요한 개념을 어떻게 사용하는지 설명하는 "개요" 부분으로 시작한다. 여러분이 "사용"의 관점을 원할지(그러면 그저 개요를 읽기만 하면 된다) 혹은 "이해"의 관점을 원하는지(그러면 계속 읽으면 된다)를 선택할 수 있다.

## The Undergraduate Tour

여러분이 컴퓨터공학의 학생이거나 소프트웨어 엔지니어의 경우, 테스팅의 기본과 관련된 영역을 알고 싶음. 기술을 그저 사용하는 것에서 그치지 않고 알고리즘와 구현에 대해 깊이 파고 싶을 경우, 다음을 여러분께 추천한다.

1. 기본 개념을 잡기 위해 [Introduction to Testing](/Part1/Introduction%20to%20Software%20Testing.md)과 [coverage](/Part2/Code%20Coverage.md)로 시작한다(이들 중 몇몇은 이미 알고 있을 수 있지만, 당신은 학생이지 않은가?).
2. [Fuzzer에 대한 장](/Part2/Fuzzing:%20Breaking%20Things%20with%20Random%20Inputs.md)으로부터 간단한 fuzzer가 어떻게 작동하는지 배운다. 이는 이미 90년대 UNIX utilities의 30%를 차지했던 도구를 제공한다. 만약 여러분이 이전에 fuzz된 적이 없는 도구를 테스트하면 무슨 일이 일어날까?
3. [Mutation-based fuzzing](/Part2/Mutation-Based%20Fuzzing.md)는 오늘날에 fuzzing의 기준이다: seed 한 세트를 가지고 버그를 찾을 때까지 변형시킨다.
4. 어떻게 [grammars](/Part3/Fuzzing%20with%20Grammars.md)를 사용하여 구문적으로 정확한 입력을 생성하는지를 배운다. 이는 테스트 생성을 훨씬 더 효율적으로 만들지만, 여러분은 처음부터 문법을 써야(혹은 [내것을 사용](/Part4/Mining%20Input%20Grammars.md))해야 한다.
5. [fuzz APIs](/Part5/Fuzzing%20APIs.md)와 [graphical user interfaces](/Part5/Testing%20Graphical%20User%20Interfaces.md)를 하는 법을 배운다. 둘 다 소프트웨어 테스트 생성에 중요한 도메인이다.
6. 자동적으로 [reduce failure-inducing inputs](/Part3/Reducing%20Failure-Inducing%20Input.md)를 최소한으로 만드는 법을 배운다. 디버깅을 위한 좋은 time saver고 특히 자동 테스팅의 합에서 그렇다.

이 모든 장에서, 그들의 개념을 이해하기 위해 구현을 실험한다. 여러분이 원하는 만큼 자유롭게 실험해보자.  
만약 당신이 교사라면, 위 장들은 프로그래밍 그리고/혹은 소프트웨어 공학 코스에서 유용할 수 있다. 슬라이드와/혹은 live 프로그래밍을 사용하여 학생들이 실험하도록 할 수 있다.  

## The Graduate Tour

"대학생" 투어 맨 위에서, 당신은 더 요구하는 기술을 포함한 테스트 생성 기술에 더 깊이 알고 싶어할 수 있다.  

1. [Search-based testing](/Part2/Search-Based%20Fuzzing.md)은 특정 목표, 예를 들어 code coverage, Robust 그리고 효율을 향한 테스트 생성을 인도할 수 있게 한다.
2. [configuration testing](/Part5/Testing%20Configurations.md)에 대한 소개를 받는다. 여러 구성 옵션을 제공하는 시스템을 테스트하고 다루는 방법은 무엇인가?
3. [Mutation analysis](/Part2/Mutation%20Analysis.md)는 테스트가 버그를 찾을 수 있는지 검사하기 위해 인조 결함(mutation)을 심는다. 만약 테스트가 mutation을 찾지 못하면, 그들은 실제 버그도 찾아내지 못할 것이다.
4. 문법을 사용해 입력을 [parse](/Part3/Parsing%20Inputs.md)하는 법을 가르친다. 만약 현존하는 입력을 분석하고, 분해하고, 변형하고 싶다면 그걸 위해 parser가 필요할 것이다.
5. [Concolic](/Part4/Concolic%20Fuzzing.md)과 [symbolic](/Part4/Symbolic%20Fuzzing.md) fuzzing은 프로그램 경로 사이의 제약 조건을 해결하여 테스트하기 어려운 코드에 도달한다. 신뢰성이 가장 중요한 곳이라면 어디서나 사용할 수 있으며 또한 뜨거운 연구 주제이기도 하다.
6. [fuzzing을 언제쯤 멈춰야 할지](/Part6/When%20To%20Stop%20Fuzzing.md) 예상하는 법을 배운다. 언젠가는 fuzzing을 멈춰야 할 것이다, 그렇지?

이 모든 장에 대해 코드를 실험해 보자. 여러분만의 변형과 확장을 자유롭게 만들자. 이것이 우리가 연구하는 방식이다!
만약 당신이 교사라면, 위 장들은 프로그래밍 그리고/혹은 소프트웨어 공학 코스에서 유용할 수 있다. 슬라이드와/혹은 live 프로그래밍을 사용하여 학생들이 실험하도록 할 수 있다.  

나머지는 귀찮으니까 영어 원문으로 대체한다. 아몰랑~

## The Black-Box Tour

This tour focuses on black-box fuzzing – that is, techniques that work without feedback from the program under test. Have a look at  

1. Basic fuzzing. This already gives you tools that took down 30% of UNIX utilities in the 90s. What happens if you test some tool that has never been fuzzed before?
2. Syntactical fuzzing focuses on grammars as a means to specify the syntax of inputs. The resulting test generators produce syntactically correct inputs, making tests much faster, and provide lots of control mechanisms for the tester.
3. Semantic fuzzing attaches constraints to grammars, making inputs not only syntactically valid, but also semantically valid - and empowering you to shape test inputs just like you want them,
4. Domain-specific fuzzing showing a number of applications of these techniques, from configurations to graphical user interfaces.
5. If you want to deploy a large set of fuzzers, learn how to manage a large set of fuzzers.

## The White-Box Tour

This tour focuses on white-box fuzzing – that is, techniques that leverage feedback from the program under test. Have a look at  

1. Coverage to get the basic concepts of coverage and how to measure it for Python.
2. Mutation-based fuzzing is pretty much the standard in fuzzing today: Take a set of seeds, and mutate them until we find a bug.
3. Greybox fuzzing with algorithms from the popular American Fuzzy Lop (AFL) fuzzer.
4. Information Flow and Concolic Fuzzing showing how to capture information flow in Python programs and how to leverage it to produce more intelligent test cases.
5. Symbolic Fuzzing, reasoning about the behavior of a program without executing it.

## The Researcher Tour

On top of the "Graduate" tour, you are looking for techniques that are somewhere between lab stage and widespread usage – in particular, techniques where there is still room for lots of improvement. If you look for research ideas, go for these topics.

1. Mining function specifications is a hot topic in research: Given a function, how can we infer an abstract model that describes its behavior? The conjunction with test generation offers several opportunities here, in particular for dynamic specification mining.
2. Mining input grammars promises to join the robustness and ease of use of lexical fuzzing with the efficiency and speed of syntactical fuzzing. The idea is to mine an input grammar from a program automatically, which then serves as base for syntactical fuzzing. Still in an early stage, but lots of potential.
3. Probabilistic grammar fuzzing gives programmers lots of control over which elements should be generated. Plenty of research possibilities at the intersection of probabilistic fuzzing and mining data from given tests, as sketched in this chapter.
4. Fuzzing with generators and Fuzzing with constraints gives programmers the ultimate control over input generation, namely by allowing them to define their own generator functions or to define their own input constraints. The big challenge is: How can one best exploit the power of syntactic descriptions with a minimum of contextual constraints?
5. Carving unit tests brings the promise of speeding up test execution (and generation) dramatically, by extracting unit tests from program executions that replay only individual function calls (possibly with new, generated arguments). In Python, carving is simple to realize; here's plenty of potential to toy with.
6. Testing web servers and GUIs is a hot research field, fueled by the need of practitioners to test and secure their interfaces (and the need of other practitioners to break through these interfaces). Again, there's still lots of unexplored potential here.
7. Greybox fuzzing and greybox fuzzing with grammars bring in statistical estimators to guide test generation towards inputs and input properties that are most likely to discover new bugs. The intersection of testing, program analysis, and statistics offers lots of possibilities for future research.

For all these topics, having Python source available that implements and demonstrates the concepts is a major asset. You can easily extend the implementations with your own ideas and run evaluations right in a notebook. Once your approach is stable, consider porting it to a language with a wider range of available subjects (such as C, for example).

## The Author Tour
This is the ultimate tour – you have learned everything there is and want to contribute to the book. Then, you should read two more chapters:

1. The guide for authors gives an introduction on how to contribute to this book (coding styles, writing styles, conventions, and more).
2. The template chapter serves as a blueprint for your chapter.

If you want to contribute, feel free to contact us – preferably before writing, but after writing is fine just as well. We will be happy to incorporate your material.

## Lessons Learned

- You can go through the book from beginning to end...
- ...but it may be preferable to follow a specific tour, based on your needs and resources.
- Now go and explore generating software tests!