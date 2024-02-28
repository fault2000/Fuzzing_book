# Fuzzing_book
[Fuzzing book](https://www.fuzzingbook.org)의 내용을 번역하고, 실습해본 리포지토리

## Before we begin

안타깝게도 원문에는 코드 내의 import가 있는 경우 클릭하여 가져오는 코드의 원문이나 만든 코드의 github 페이지로 연결하는 기능이 존재하나, 현재 작성하는 마크다운의 특성상 이 기능을 구현하는 방법을 찾지 못했다.  
html 기능을 활용하면 링크를 활성화할 순 있으나, 코드가 하이라이팅되지 않는 문제가 발생하고 이 문제가 가독성에 더 영향을 끼친다고 생각하여 안타깝게도 링크 기능은 추가하지 않았다. 만약 코드 내의 링크 기능을 사용하고 싶다면 원문을 활용하는 것을 권한다.

## Sitemap

이 책의 장들은 차례로 읽을 수 있지만, 책을 읽을 수 있는 순서 또한 다양하다. 이 그래프에서, 화살표 A -> B는 챕터 A가 챕터 B를 읽기 위한 준비물임을 나타낸다. 당신이 가장 흥미있는 곳을 위해 당신은 이 그래프의 임의의 경로를 골라 읽을 수 있다.

![image](https://github.com/fault2000/Fuzzing_book/assets/73513005/f4e87e3e-774c-49ca-996d-360e83618685)

## 목차

[Part 1: 식욕 돋구기](./Part1/Whetting%20Your%20Appetite.md)
- [책을 통한 여행](./Part1/Tours%20through%20the%20Book.md)
- [소프트웨어 테스팅 소개](./Part1/Introduction%20to%20Software%20Testing.md)

[Part 2: Lexical(어휘) Fuzzing]()
- [Fuzzing: 임의 입력으로 고장내기]()
- [코드 커버리지]()
- [변형 기반 fuzzing]()
- [Greybox fuzzing]()
- [변형 분석]()

[Part 3: Syntactic(구문) Fuzzing]()
- [문법으로 fuzzing]()
- [효율적 문법 fuzzing]()
- [문법 커버리지]()
- [입력 구문 분석]()
- [확률적 문법 fuzzing]()
- [생성기로 fuzzing]()
- [문법과 함께 Greybox Fuzzing]()
- [고장을 유발하는 입력 줄이기]()

[Part 4: Semantic(의미) Fuzzing]()
- [제약 조건을 활용한 Fuzzing]()
- [입력 문법 캐기(Mining Input Grammars)]()
- [정보 흐름 추적]()
- [concolic Fuzzing]()
- [Symbolic Fuzzing]()
- [함수 세부설명 캐기(Mining Function Specification)]()

[Part 5: Domain-Speciic Fuzzing]()
- [테스팅 구성]()
- [Fuzzing APIs]()
- [Carving Unit Tests]()
- [Testing Compilers]()
- [Testing Web Applications]()
- [Testing Graphical User Interfaces]()

[Part 6: Managing Fuzzing]()
- [Fuzzing in the Large]()
- [When To Stop Fuzzing]()

[Appendices]()
- [Acadimic Prototyping]()
- [Prototyping with Python]()
- [Error Handling]()
- [Timer]()
- [Timeout]()
- [Class Diagrams]()
- [Railroad Diagrams]()
- [Control Flow Graph]()
