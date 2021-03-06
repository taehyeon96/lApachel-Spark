# 01. Spark Overview

---
## 1.1 Spark Basic

- Spark
  - 하둡 기반의 맵리듀스 작업이 가진 단점을 보완하기 위한 것
  - 메모리를 이용한 데이터 저장 방식을 제공 => 머신러닝 등 반복적인 데이터 처리가 필요할 시 높은 성능
  - 하둡이나 스파크는 **클러스터 환경에서 동작**하는 프레임워크
  
- 클러스터
  - 여러 대의 컴퓨터(=서버)가 하나의 그룹을 형성해 마치 하나의 컴퓨터인 것처럼 동작하는 것
    - 예) 16GB 메모리가 장착된 서버 10대로 클러스터를 구성하여, 마치 160GB 메모리가 장착된 서버처럼 간주하고 프로그램을 작성
  - 따라서, **여러 서버가 한 대의 서버처럼 동작해야 하기 때문에 CPU나 메모리, 디스크 등의 자원 관리가 어려움**
  - **개발 및 테스트를 위해 1대의 단독 서버 또는 개인 PC에서 개발 및 실행하고, 실 서비스는 클러스터에서 실행**

 
- Spark에서 데이터를 표현하고 처리하기 위한 프로그래밍 모델
  - "RDD", "데이터셋", "데이터프레임"


- RDD : resilient distributed dataset
  - 단순히 "값"으로 표현되는 데이터만 가리키는 것이 아니고,
  - 데이터를 다루는 방법까지 포함하는 일종의 클래스와 같은 개념
  - "RDD란 스파크가 사용하는 핵심 데이터 모델로서 다수의 서버에 걸쳐 분산 방식으로 저장된 데이터 요소들의 집합을 의미하며,
    병렬처리가 가능하고 장애가 발생할 경우에도 **스스로 복구될 수 있는 내성(tolerance)**를 가지고 있다."
  - "Spark revolves around the concept of a resilient distributed dataset (RDD), which is a fault tolerant collection of elements that can be operated on in parallel."

  - 즉, RDD는 스파크에서 정의한 분산 데이터 모델로서 병렬처리가 가능한 요소로 구성되며,
    데이터를 처리하는 과정에서 문제가 발생하더라도 스스로 에러를 복구할 수 있는 능력을 가진 모델이다.
    => 첫 글자가 resilient 인 이유
    
    
- RDD를 통해 분산 병렬처리를 하는 방법
  - RDD에 속한 요소들은 "파티션"이라는 단위로 나눠질 수 있다.
  - 따라서 스파크는 작업을 수행할 때 하나의 RDD를 여러 개의 파티션으로 나눠서 병렬처리를 수행
  - 이때 각 파티션을 다수의 서버에 보내어 처리


- 리니지(Lineage)
  - 리니지란, **스파크에서 RDD 생성 작업을 기록해두는 것**
  - 다수의 서버에서 작업하다 보니 일부 파티션 처리에 장애가 발생할 수 있는데
  - 이때 스파크는 손상된 RDD를 복원하기 위해 RDD의 "생성 과정을 기록"해둠
  - 즉, RDD에 포함된 데이터를 일일히 저장해뒀다가 복원하는게 아니고, 
    RDD를 생성했던 기록을 저장해서 문제가 발생했던 "직전 시점의 RDD를 만들 때 작업을 똑같이 실행해 데이터를 복구"하는 방식임 
    
    
- RDD 생성 방법 3가지
  - Collection 사용
    - sc.parallelize(리스트 자료형)
  - 파일로부터 생성
    - sc.textFile("파일경로")
  - 기존 RDD로부터 새로운 RDD 생성
    - rdd_B = rdd_A.map(변환할 내용)


- RDD 연산
  - RDD 생성 후 수행하는 두 가지 필수 작업
  - Transformation (트랜스포메이션)
    - 어떤 RDD에 변형을 가해 "새로운 RDD를 생성하는 연산"
    - 이때, 기존 RDD는 바뀌지 않고, 변형된 값을 가진 새로운 RDD가 생성되는 연산임 => resilient를 위해
    - <중요> 해당 연산은 바로 수행되지 않음!! 정보만 가지고 있음!!
  - Action (액션)
    - 연산의 결과가 RDD가 아닌 "int, long 등의 값을 반환하거나 / 아얘 반환하지 않거나 / 결과물 출력하는 등의 연산"
    - 트랜스포메이션이 정보를 누적해서 갖고 있다가 "액션"에 해당하는 연산이 호출될 때 한꺼번에 실행
    - 이를 통해, 본격적인 작업 실행 전에 데이터를 어떤 방법과 절차에 따라 변형할지 정할 수 있음
    - => 특히 쓸데없는 작업 수를 줄여 데이터 처리 시 비용을 줄이고, 셔플(shuffle) 횟수도 줄일 수 있다.
  - **우지는 XML에 데이터 처리 시나리오를 설계해서 작성하는게 상당히 복잡하고 어려운데**
  - **트랜스포메이션과 액션은 이를 매우 쉽게 만들어준다!**

- 우지(oozie)와 DAG 
  - 오픈소스 워크플로우 시스템 중 하나로, 하둡 맵리듀스 잡과 피그 등을 스케줄링하고, 연동할 수 있게 지원하는 도구
  - 데이터 처리 시 각 단계마다최적화된 작업을 수행하기 위해 사용하는 라이브러리가 다 다른데,
  - 우지를 사용해 일련의 작업 흐름을 XML을 사용해 명시적으로 선언해 쓸 수 있다.
  - 이때 작업 흐름을 나타내는 워크플로우는 DAG(directed acyclic graph)를 구성해서 위의 라이브러리들을 연동하면서 데이터를 처리함
  

- DAG
  - 그래프 이론에서 사용되는 용어로, 여러 개의 node와 그 사이를 이어주는 directed edges로 구성되며
  - 어떤 노드에서 출발하더라도 다시 원래 노드로 돌아오지 않도록 구성된 graph model임
  - **빅데이터에서 DAG는 복잡한 작업 흐름을 나타내는 용도로 많이 사용**

- DAG 스케줄러
  - 스파크에서 DAG 처리를 담당
  1. 스파크는 전체 작업을 스테이지(Stage)라는 단위로 나누고, 
  2. 각 스테이지를 다시 여러 개의 테스크(Task)로 나눠 실행
  3. 이때 최초로 메인 함수를 실행해 RDD를 생성하고, 각종 연산을 호출하는 프로그램을 드라이버(Driver)라고 함 => Main함수에 일반적인 코드 작성
  4. 드라이버 메인 함수에서 => 스파크 애플리케이션, 스파크 클러스터 연동을 담당하는 Spark Context 또는 Spark Session이란 객체를 생성
  5. Spark Context 또는 Session을 이용해 잡을 실행하고 종료
  6. 드라이버는 Spark Context를 통해 RDD 연산 정보를 DAG 스케줄러에게 전달
  7. DAG 스케줄러는이 정보를 기반으로 "실행 계획 수립" 후 클러스터 매니저에게 전달
  8. 최종적으로 클러스터는 전체 데이터 처리 흐름을 분석해서 네트워크를 통한 데이터의 이동이 최소화 되도록 스테이지를 구성(=셔플 줄이고, 비용 줄이고)


- 넓은 의존성과 좁은 의존성 (wide dependency and narrow dependency)
  - 하나의 RDD가 새로운 RDD로 변환될 때, 기존 RDD = 부모 RDD, 새로운 RDD = 자식 RDD 라고 가정
  - 넓은 의존성
    - 부모 RDD를 구성하는 파티션이 여러 개의 자식 RDD 파티션과 관계를 맺는 경우
    - <다른 파티션을 참조해야 할 경우> RDD의 하늘색 각 파티션이 분산되어 다른 서버에 저장되어 있는데, SUM()하기 위해 참조하고 싶은 경우 등 사용
  - 좁은 의존성
    - 그 반대
    - <일반적인 연산> 만약 파티션 내 데이터에 숫자 1을 더하는 연산을 할 경우, 부모 RDD 파티션 -> 자식 RDD 파티션에 각각 +1 수행됨
    - 즉, 같은 RDD 내에 다른 파티션에 영향을 받지않고 각각 +1해줌
  - 다음 그림에서 큰 박스가 RDD, 하늘색 작은 박스가 하나의 파티션임
  
    <img width="40%" src="https://user-images.githubusercontent.com/61690289/150372489-d0168f1c-57f4-41b3-aa8d-acfa2b48caf6.PNG"/>


- 람다 아키텍처
  - 빅데이터 처리를 위한 시스템을 구성하는 방법 중 하나(= 아키텍처 모델)
  - 대량의 데이터 처리, 실시간 로그 데이터 처리 등 두 요구사항을 잘 충족시키기 위해 어떤 아키텍처가 좋을지 정리한 것
        <img width="40%" src="https://user-images.githubusercontent.com/61690289/150492401-e2cbcc65-ca38-427a-be55-0e0856df4c12.PNG"/>

  - Batch Layer
    - 데이터를 처리하는 시스템들을 일괄 처리하는 영역
    - **머신러닝과 같은 반복 처리가 필요한 업무에 적용**
  - Speed Layer
    - 데이터를 처리하는 시스템들을 실시간 처리하는 영역
    - **서브 모듈인 스파크 스트리밍을 통해 머신러닝 업무 적용**
    
  - 람다 아키텍처 작동 프로세스
    1. 새로운 데이터는 두 레이어에 모두 전달됨
    2. Batch Layer는 원본 데이터를 저장한 후, 일정 주기마다 한 번씩 일괄적으로 가공하여 Batch View를 생성
      - (이때, View는 외부에 보여지는 데이터로, 즉 결과 데이터라고 이해해도 됨)
    3. Speed Layer는 들어오는 데이터를 즉시(또는 매우 짧은 주기로) 처리해 Real-time View를 생성
    4. Serving Layer는 앞에 Batch View와 Real-tiem View의 결과를 적절히 조합해 User에게 데이터를 전달
      - (이때, Serving Layer를 거치지 않고 두 View를 직접 조회할 수도 있음)
  - **람다 아키텍처 핵심**
    - Batch Layer에서 일괄 처리 작업을 수행하지만, 배치 처리가 수행되지 않은 부분을 실시간 처리를 통해 보완함
    - 이때, 실시간 처리 결과는 다소 정확하지 않을 수 있음
    - 따라서 추후에 일괄 처리 작업을 수행하여 보정하는 형태로 운영
    - 즉, 두 개의 Real-time과 Batch View를 적절히 조합해야 함


---
## 1.2 Spark Install

- 사용 컴퓨터 성능
  - OS : Linux (Ubuntu 18.04.6 LST)
  - processor : Intel i5-11500
  - RAM : 32.0GB

- Virtualbox 및 ubuntu 설치 (여분의 컴퓨터 필요 시) 
  - https://spidyweb.tistory.com/212?category=842040
  - https://sseni.tistory.com/95
  - OS : Linux (Ubuntu 18.04.6 LST)
  - RAM : 4096MB
  - HDD : 24GB
 
- jdk 설치
  - sudo apt install openjdk-8-jdk -y
  - java -version; javac -version
  - 필요 시 폴더 이동

- Spark 설치
  - 컴퓨터 Ver : spark-3.1.2
  - Virtualbox Ver : spark-3.2.0
  - wget https://downloads.apache.org/spark/spark-3.1.2/spark-3.1.2-bin-hadoop2.7.tgz
  - 압축해제
    - sudo tar xvf spark-*
  - 필요 시 폴더 이동
    - sudo mv spark-3.0.2-bin-hadoop2.7 /apps

- Spark가 잘 설치되었는지 확인 - README 파일의 포함된 단어의 개수 세어보기 예제
  - cd apps/spark-3.1.2-bin-hadoop2.7
  - ./bin/run-example JavaWordCount README.md
  - (설명) run-example : 셸 프로그램, JavaWordCount : 실행할 프로그램 이름(main함수를 갖는 클래스명)
    - [실행결과]
    - ~로그 출력 후~
    - package : 1
    - For : 3
    - Programs : 1
    - ...
  - "For"라는 단어가 포함된 모든 라인 출력
  - grep -w -n --color=always "For" README.md
    - [실행결과]
    - 32: 문장
    - 57: 문장
    - 68: 문장

- **Spark Shell**
  - 코드 작성 후 빅드한 뒤 스크립트를 이용해 실행하지 않고, 
  - Linux Shell, Python IDEL 3.0처럼 인터랙티브 방식으로 프로그램을 작성할 수 있는 것
  - cd apps/spark-3.1.2-bin-hadoop2.7
  - **Scala**
    - ./bin/spark-shell
        <img width="40%" src="https://user-images.githubusercontent.com/61690289/151196895-784242b9-60b4-42c4-80d2-02dc88ed490d.PNG"/>
  - **PySpark** (Python 설치 필요)
    - ./bin/pyspark
  - 셸 종료 방법
    - Ctrl + D


- 로그 설정
  - cd conf/
  - cp ./log4j.properties.template log4j.properties
  - vim log4j.properties
    - 'i'로 수정 => "log4j.logger.org.apache.spark.repl.Main" -> "log4j.logger.org.apache.spark.repl.Main=INFO" 
  - cd ..
  - ./bin/spark-shell
    - 설정하기 전보다 더 자세히 로그가 출력되어 있음
  - sc라는 키워드를 통해 스파크 컨텍스트(Spark Context)와 스파크 세션(Spark Session)이라는 인스턴스를 사용 가능
    - scala> sc
    - [결과] res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@2a794780
    - => sc가 org.apache.spark.Sparkcontext 타입의 인스턴스(=클래스의 객체)를 가리킴을 알 수 있음!(=임의의 변수 res0를 할당!!)
    
