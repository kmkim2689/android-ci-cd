## Github Action으로 CI/CD 파이프라인 만들기

### Github Action 생성하기
* .y(a)ml 파일을 활용
  * y(a)ml : 데이터 직렬화를 위한 언어로, 스크립트 자동화에 특히 많이 사용
* `indentation`에 유의한다.

1. 프로젝트 root에 .github라는 이름의 디렉토리를 생성하고, 다시 하위에 workflows라는 패키지를 생성한다.
* 반드시 해당 위치에 action 파일들이 정의되어야 한다.

2. 내부에 ci.yml 파일 생성
* 파일에서 사용되는 구문/요소들
  * `name` : 액션의 전체적인 주제를 나타내는 이름
  * `on`
    * `언제`를 가리킴
    * 어떠한 이벤트가 발생했을 때 해당 액션을 실행시킬지 결정한다.
    * or 조건 -> 하위에 여러 개의 요소가 병기되어 있다면, 그것들 중 하나가 수행되었을 때 액션이 실행된다.
    * push와 pull_request : 각각 미리 정해진 브랜치에 push/pull_request가 발생했을 때 수행됨을 의미
      ```
      name: XX
      
      on:
        push:
          branches: [ "main" ] # main branch로 push 이벤트 발생 시, 액션 시작. "또는"
        pull_request:
          branches: [ "main" ] # main branch로 pull_request 이벤트 발생 시, 액션 시작
      ```
    * workflow_dispatch : `깃허브 프로젝트의 "Actions" 탭`에서 `수동으로 액션 스크립트를 실행`시키고자 할 때 사용
      ```
      name: XX
      
      on:
        workflow_dispatch:
      ```
      
3. `jobs` - Meats and Potatoes of Github Actions
* 1개 이상의 job으로 구성되며, `병렬적 혹은 순차적으로 수행`될 수 있다.
* 하위 계층에 원하는 이름으로 여러 개의 job을 정의 가능
* `job의 이름`의 하위 계층부터, 수행하고자 하는 작업들을 명시한다.
  * `runs-on` : job이 수행될 기기를 나타낸다. 특정 job에 대해 동작 시키고 싶은 runner를 각기 설정 가능
    * 수행 속도 향상 등의 목적으로 특정 작업에 적합한 다양한 머신 사용 가능
    * 전 세계에 있는 깃허브의 서버 컴퓨터 중 임의의 컴퓨터에서 수행할 수 있음
    * 고전적으로는 ubuntu-latest를 활용하는 편이다.
    ```
    name: XX
    
    on:
    workflow_dispatch:
    
    jobs:
    job_1: # 바로 하위 계층에 작업명 명시
    # 작업명 하위 계층부터 세부 작업을 명시한다.
    runs-on: ubuntu-latest
    ```
  * `steps`
    * 모든 job에는 일련의 step이 필요하다. `-` 문자열로 각 스텝을 명시한다.
    * `name` : 스텝의 이름을 설정
    * step에서는 일반적으로 repository의 코드 베이스를 체크하는 과정이 필요하다. 따라서 job이 코드에 엑세스할 수 있도록 해야 한다.
      * yaml 파일이 프로젝트에 존재하기는 하지만, 직접 액션 파일에서 머신(여기서는 우분투)이 코드베이스를 실행할 수 있도록 하는 과정이 필요하다.
        `- uses: actions/checkout@v4`
      * `uses` : 서드파티 라이브러리 활용을 위함 - 깃허브 마켓플레이스에서 checkout을 검색하면 나오는 checkout action을 활용
        * 안드로이드 프로젝트에서 다른 라이브러리를 import하는 것과 같은 원리
      * https://github.com/marketplace/actions/checkout
      ```
      name: CI

      on:
      workflow_dispatch:
      
      jobs:
      job_sample:
      runs-on: ubuntu-latest # type of runner that the job will run on
      
      steps:
      - uses: actions/checkout@v3
      ```
      
    * name과 run으로 구성된 코드
    ``` 
    name: CI

    on:
    workflow_dispatch:
    
    jobs:
      job_sample:
        runs-on: ubuntu-latest # type of runner that the job will run on
        
        # steps는 runs-on과 같은 계층에 있어야 하며, job명 하위에 존재해야 한다.
        steps: 
          - uses: actions/checkout@v3
      
          # runs a single command using the runners' shell
          - name: Run a one-line script
            run:
              echo Hello World
      
          # runs a set of commands using the runners' shell
          # ,로 구분하고 있음
          - name: Run a multi-line script
            run:
              echo Add other actions to build,
              echo test, and deploy your project.
    ```
    
* 최종 파일
```
name: CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  job_sample:
    runs-on: ubuntu-latest # type of runner that the job will run on

    steps:
      - name: Check out code base
        uses: actions/checkout@v4

      - name: Run a one-line script
        run:
          echo Hello World

      - name: Run a multi-line script
        run:
          echo Add other actions to build,
          echo test, and deploy your project.
```


4. Linting

* 초기 설정
```
name: CI

on:
  push:
    branches: [ "main" ] # main branch로 push 이벤트 발생 시, 액션 시작. "또는"
  pull_request:
    branches: [ "main" ] # main branch로 pull_request 이벤트 발생 시, 액션 시작

jobs:  
  lint:
    name: Linting debug variant
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
```

* 안드로이드 앱이므로, 우분투에 java 및 gradle 환경에 대한 설정이 필요
  * 안드로이드 개발 당시에는 gradle 파일에 정의되어 있기에(jvmTarget) 별도의 작업이 필요없었음
  * 하지만 CI 과정에서는 action을 실행할 ubuntu 머신에 자바와 gradle 환경을 설정해야 한다.
    * 이것을 위하여, marketplace에 있는 "Setup Java JDK", "Gradle Build Action"을 활용하면 쉽게 설정 가능
    * https://github.com/marketplace/actions/setup-java-jdk
    * https://github.com/marketplace/actions/gradle-build-action
  ```
  - uses: actions/setup-java@v3
    with:
      distribution: 'temurin' # See 'Supported distributions' for available options
      java-version: '17' # 주의 : 자바 버전은 gradle에서 설정된 것과 동일해야 한다. gradle에서 1.8로 되어있으면 8로 설정
  ```
  
  ```
  - name: Setup Gradle
    uses: gradle/gradle-build-action@v2
  
  - name: Execute Gradle Command - Lint Debug
    run: ./gradlew lintDebug # 린트 검사를 위한 gradle의 커맨드
  ```
  
  * with
    * parameters for running `uses` properly
  
  * cf) lintDebug 관련 이슈
    https://github.com/actions/starter-workflows/issues/171
    * gradle 환경 설정한 곳과 같은 계층에 다음 코드를 추가
    * 권한 설정을 변경하기 위함. 반드시 순서는 gradle setup 이후 본격적 작업 이전에 위치시켜야 함
    ```
    - name: Make gradlew executable
      run: chmod +x ./gradlew
    ```