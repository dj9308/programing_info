# Micro Service Architecture

## 정의

- 쉽게 말해 캡슐화된 비즈니스 로직이다.
- Monolithic 구조에서 하나의 프로젝트에 종속되어 있던 비즈니스를 도메인 별로 나누어 별개의 서비스로 운영을 하는 것을 의미한다.
- Monolithic 구조
  - ![Monolitic](https://ssipflow.github.io/assets/images/static/180919/Monolithic.png)
- MSA 구조
  - ![MSA](https://ssipflow.github.io/assets/images/static/180919/MicroServices.png)

## 장점

1. 빌드 및 테스트 시간이 단축된다.
2. 서비스간 영향을 받지 않는다.
3. 각 서비스는 자신만의 고유한 아키텍쳐와 기술을 적용할 수 있다.
4. 탄력적이고 선택적인 확장이 가능하다.

## 단점

- Monolithic에 비해 속도가 느리다.
- 하나의 어플리케이션을 여러 개의 서비스로 나누기 때문에 모니터링, 로깅, 인스턴스 관리에 어려움이 있다.