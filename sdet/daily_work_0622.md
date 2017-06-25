# 교육 안내

# Assignment 1
팀 프로젝트 Quality Plan + Code Review

- [ ] 과제 수행 일정 수립 with 팀원

# 1일차 / 6/22(목)
## SDET Driven Development by 엄위상w
- 서비스 인터페이스를 정의한다. (서비스 뿐만 아니라 업무)
- GOOS - TDD and 테스트를 위한 소프트웨어 설계 추천 도서
- Interface를 정의하고 설계하는 것을 Test가 drive함
- Lead development by your code
- 코드는 생각보다 힘이 세다. code + data

## LLVM

개발자 테스트 토론
- 제어연에서 SDET이 커버리지 기준을 제시해서 unittest 작성하도록 가이드하고(branch coverage 75%), Jenkins에 연동
- MC에서 TDD로 개발
- QA에서 함
- SIC : 프로세스에 일단 개발자 테스트가 있음. fake/stubbing해서 테스트함. 설계 초기에 의존성 제거하고 
- 각 모듈별로 sanity test하고, code scroller(unit test tool) , 정적 분석 툴(coverage)

테스트 서버: 156.147.178.117 / user3/lge123

[Clang AST](https://clang.llvm.org/docs/IntroductionToTheClangAST.html)
[ASTNode](https://clang.llvm.org/doxygen/classclang_1_1ASTNodeImporter.html)

[ASTMatcher](http://clang.llvm.org/docs/LibASTMatchersReference.html)

[mod.lge.com](http://mod.lge.com/code/projects/SDETCOM/repos/apitestassistant/browse/src)

[frog program](www.frog-test.com)

### Concolic Testing
Test Oracle
