# Executor

- Query Processing의 Optimizer가 생성한 실행 계획에 따라 데이터 가공에 필요한 알고리즘 실행  
- 실행 계획 저장 Plan Tree에서, 각 Plan Node는 실행 계획 각 단계에서 실행할 알고리즘 함수 포인터 포함  
- 실행 계획을 순회 및 사용자 요구 결과물로 데이터 가공

## Executor Framework

쿼리를 처리하는 프레임워크는 pquery.c 파일 내부 ProcessQuery 함수에서 실행  

1. CreateQueryDesc  
    QueryDesc 객체 초기화 및 실행 계획 수행에 필요한 초기 정보 메타데이터 저장  
    **QueryDesc**  
    Optimizer가 생성한 Plan Tree와 쿼리가 참조할 스냅샷 정보 및 바인드 변수 정보  저장 객체  

2. ExecutorStart  
    실행 계획 수행에 사용할 리소스 및 표현식 정보 초기화  
    QueryDesc 생성 후 Plan Tree에 있는 Plan Node 순회하며 PlanState 구조체로 이루어진 **State Tree** 이진트리 생성  

    - **PlanState**: 각 Plan Node의 Operation 알고리즘을 수행하며 기록해야 하는 상태 정보 저장  

    - **EState**: 쿼리 전반에 걸쳐 기록해야 하는 상태 정보 저장  

    State tree를 만들면서 각 Plan Node의 Operation에서 필요한 초기화 작업 수행  

    **State Tree**  
    ExecutorStart 시작 시 Plan Tree의 루트 노드부터 시작하여 재귀적으로 자식 노드(Plan Node)에 대한 초기화 함수 호출하며 만들어지는 이진트리  
    Plan Tree의 루트 노드를 입력으로 ExecInitNode 함수 실행하여, Plan Node의 Operation 별로 사용할 리소스를 초기화 진행 후 재귀적으로 자식 노드의 ExecInitNode 호출하며 State Tree 생성

3. 실행 계획 수행 (ExecutorRun)  
    ExecutorStart에 만든 State Tree의 루트에 접근하여, 노드 내부 ExecProcNode 함수 포인터 실행  
    **ExecProcNode**
    - Operation에서 수행해야 하는 알고리즘을 담은 함수  
    - PlanState 초기화 시 Operation 마다 정의된 함수로 설정  
    - 각 Operation의 Input 데이터를 받기 위한 OuterPlanState의 재귀 호출 로직 포함

4. 실행 계획 수행 완료 후처리 작업 수행 (ExecutorFinish)  
    

5. 실행 계획 수행에 사용한 리소스 해제 및 상태 정보 초기화 (ExecutorEnd)
6. 쿼리 메타데이터 해제 (FreeQueryDesc)

