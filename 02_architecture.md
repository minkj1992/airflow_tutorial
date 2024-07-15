# Architecture Overview
> https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html

Airflow is a platform that lets you build and run workflows.

![](https://airflow.apache.org/docs/apache-airflow/stable/_images/edge_label_example.png)

- workflow == DAG
- pieces of work == Task
- predefined Task == Operator
- special type of `Operator` == `Sensors`
    - waitting for external event to occur
    - subclass of Operators

> `Sensor` is a special type of operator, waits for something to occur. `poke` mode is default, it takes a worker slot for its entire runtime (= block). `reschedule` mode takes up a worker slot only when it is checking, and sleeps for a duration btw checks.

> Latency에 대해서 trade-off가 있으니, 초 단위로 체크해야한다면, poke mode, 분단위로 체크 해야한다면 should be in reschedule mode.



## Airflow components

### Required

- `scheduler`: trigger workflow and submit tasks to the `executor`
- `executor`: 
    - task를 실행하는 방식을 결정하는 scheduler의 property
    - SequentialExecutor부터 KubernetesExecutor까지 다양한 옵션이 있어 단일 머신에서의 순차 실행부터 대규모 분산 클러스터에서의 병렬 처리까지 광범위한 실행 환경을 지원
- `webserver`
- `A folder of DAG files`: read by the `scheduler` to figure out **what tasks and when to run**
- `metadata db`: store state of workflows (DAG) and Tasks


### Optional

- `worker`: executes `tasks` given by `scheduler`
    - **default: part of the scheduler**
    - as a POD in the `KubernetesExecutor`
- `triggerer`: executes deferred tasks in an asyncio envet loop
    - default: deferred tasks are not used
- `DAG processor`: `parses` DAG files and `serializes` them into the `metadata DB`
    - default: scheduler parses and serializes
- `folder of plugins`: 3rd packages


#### c.f More aboout trigger


<script src="https://gist.github.com/minkj1992/35a095bdf92ca170cba686aaa5b85233.js"></script>



## Deploying

### Basic Airflow deployment

![](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_basic_airflow_architecture.png)


### Distributed Airflow architecture

1. Airflow의 분산 아키텍처:
   - 여러 기계에 Airflow 컴포넌트들이 분산됨
   - 세 가지 주요 사용자 역할 소개: *Deployment Manager*, **DAG author**, **Operations User**

2. 보안 고려사항:
   - Webserver는 DAG 파일에 직접 접근하지 않고 대신, UI의 Code 탭 내용은 메타데이터 데이터베이스에서 읽어옴
   - Webserver는 DAG 작성자가 제출한 코드를 실행할 수 없음
   - Webserver는 Deployment Manager가 설치한 패키지나 플러그인의 코드만 실행 가능
   - `Operations User`는 UI 접근만 가능하며, DAG와 태스크 트리거만 가능 (DAG 작성 불가)

3. DAG 파일 동기화:
   - **Scheduler, Triggerer, Worker 간 DAG 파일 동기화 필요**

이 아키텍처는 Airflow의 분산 환경에서의 보안과 역할 분리를 강조하며, 각 컴포넌트와 사용자 역할 간의 상호작용을 명확히 정의하고 있습니다.

![](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_distributed_airflow_architecture.png)

### Separate DAG processing architecture
> Standalone DAG Processor 컴포넌트


이 컴포넌트의 주요 기능은 스케줄러가 DAG 파일에 직접 접근하는 것을 분리하는 것입니다. 이는 파싱된 태스크 간의 격리에 중점을 둔 배포에 적합합니다.
Airflow는 아직 완전한 멀티테넌트 기능을 지원하지 않지만, 이 컴포넌트를 통해 일정 수준의 격리를 제공합니다.

**주요 이점은 DAG 작성자가 제공한 코드가 스케줄러 컨텍스트에서 실행되는 것을 방지할 수 있다는 점입니다.**

![](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_dag_processor_airflow_architecture.png)


## Workloads

Task의 3가지 types

1. `Operators`: 특정 작업을 수행하는 단일 태스크 단위, 예를 들어 데이터 전송이나 SQL 쿼리 실행
2. `Sensors`: 특정 조건이나 이벤트를 기다리는 특수한 형태의 operator, 예를 들어 파일 도착 감지나 외부 프로세스 완료 대기
3. `Taskflow`: `@task` 데코레이터를 사용하여 Python 함수를 직접 DAG 태스크로 변환, 코드를 더 간결하고 읽기 쉽게 만듦


## Control flow

DAGs are designed to be run many times, and multiple runs of them can happen in parallel.

Tasks have dependencies declared on each other.

```
first_task >> [second_task, third_task]
fourth_task << third_task
```

```
first_task.set_downstream([second_task, third_task])
fourth_task.set_upstream(third_task)
```


**To pass data between tasks you have three options:**

1. `XComs`( Cross-communications): a system where you can have tasks push and pull small bits of metadata.
2. Uploading and downloading large files from a storage service (cloud)
3. `Implicit XComs`: TaskFlow API automatically passes data between tasks via implicit XComs