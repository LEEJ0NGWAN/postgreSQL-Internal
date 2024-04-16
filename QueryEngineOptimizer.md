[돌아가기](https://github.com/LEEJ0NGWAN/postgreSQL-Internal)

# Optimizer

- 클라이언트의 쿼리 수행을 최적화하기 위해, 모든 수행 방법 조합을 비교하고 최선의 실행 경로를 결정  
- PostgreSQL의 Optimizer는 쿼리 수행 최적화를 위해 Cost-based Optimization 방법 사용  

**Cost?**  
쿼리 수행에 필요한 자원을 수치화한 것;  

- 로우를 읽고 값 비교를 하는 데 발생하는 CPU 비용  
- 임의의 페이지를 읽을 때 발생하는 I/O 비용  
- ...  

쿼리 수행 가능한 모든 실행 경로에 대해, 비용 계산 및 최저 비용 경로를 실행 계획(Execution Plan) 형태로 수립  

## 실행 계획 수립

### Query Tree 간소화 작업  

실행 경로 생성 전, Query tree 내부에 미리 불필요한 표현식에 대해 간소화 진행  
- 실행 경로의 변수 개수를 줄여줌
- Query 최적화 작업 시간 감소
- 메모리, CPU 사용량 절약

e.g.,  
1. select 절에 포함된 표현식 간소화  
SQL 컴파일 시 계산할 수 있는 정적 표현식 간소화 진행  
(ex) `1+1` --> `2`  

2. boolean 연산자 간소화  
이중 부정 형식 불리언 간소화 진행  
(ex) `NOT(NOT CONDITION)` -->  `CONDITION`  

3. AND/OR 표현식 평탄화  

### 최저 비용의 실행 경로 찾기

간소화를 끝낸 Query Tree로 실행 경로를 생성한다  

쿼리가 요구하는 표현식에 따라 다양한 종류가 생성됨  

**실행경로 종류**  
- Access Path (데이터 스캔 방식)  
    - Sequential Scan: 디스크 저장 순서대로 데이터 로드  
    - Index Scan: 인덱스 활용하여 특정 로우 데이터 로드  

- Join Path (테이블 조인 방식)  
    - Hash Join: Hash Table 활용  
    - Index Join: Index 활용  
    - Nested Loop Join: 모든 로우의 조합(카테시안 곱)을 검사  

- Grouping Path (그룹 함수 집계 방식)  
    - Hash Aggregate: Hash Table을 사용하는 집계  
    - Group Aggregate: 그룹키로 정렬된 데이터 집계  

### 실행 경로를 실행 계획으로 변환  
실행 경로를 바탕으로 Plan Tree(실행 계획을 저장하는 트리 자료구조)를 만들어 실행 엔진에 전달  

**Plan Tree**  
실행 엔진이 실행 계획(plan)을 수행하는 논리적 단위 Plan Node를 이진 트리 형태로 구성  
이때, **Plan Node는 실행되어야 하는 순서의 역순으로 구성되어 있다**  
(가장 먼저 수행되는 노드가 자식노드에 위치)  
