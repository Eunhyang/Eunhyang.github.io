layout: post
    title: ""
    description: "TIL:django"
    date: 2018-
    tags: [TIL]
    comments: true
    share: true목

## Two scoops of django 5장 - settings와 requirements파일

- 최선의 장고 설정 방법
    - 버전 컨트롤 시스템으로 모든 설정 파일을 관리해야 한다
    - 반복됟는 설정들을 없애야 한다
    - 암호나 비밀 키 등은 안전하게 보관해야 한다
- 비밀 정보 보호하기!
    - SECRET_KEY 세팅은 장고의 암호화 인증 기능에 이용되고 이 세팅값은 다른 프로젝트와는 다른 유일무이한 값이 되어야 하며 버전 컨트롤 시스템에서 제외해야 한다.
- 여러 개의 settings 파일 이용하기
    - 하나의 settings파일을 만들고 각자 local_settings를 만들면 이러한 **문제점**이 있음
        - 모든 머신에 버전 컨트롤에 기록되지 않는 코드가 존재함
        - 운영환경과 로컬 환경 setting이 다르므로 setting에서 에러날 확률이 꽤 있는데 이 부분 찾느라 시간 허비함
        - 마찬가지로 버그 발견해서 푸시하고 났더니 해당 local_settings에서만 발견되는 에러임
        - 여러 팀원들이 local_settings를 복붙함
    - requirements + settings: 각 세팅 모듈은 그에 해당하는 독립적인 requirements.파일을 필요로 함.
    - settings/local, staging, test, production이런식으로 분리
- 다중 개발 환경 세팅
    - 개발자마다 자기만의 환경이 필요한 경우
        - ⇒ 버전 컨트롤 시스템에서 관리되는 파일들을 구성하여 이용
        - ex) settings/dev_eunhyang.py
        - 개발 환경 또한 버전관리를 하면 더 좋고 팀원 간 서로의 개발 세팅 파일을 참고할 수 있음
- 코드에서 설정 분리하기
    - 환경 변수 패턴 (환경 변수를 비밀 키를 위해 이용)
        - 장점
            - 환경 변수를 이용해 비밀 키를 보관함으로써 걱정 없이 세팅 파일을 버전 컨트롤 시스템에 추가할 수 있음
            - 파이썬 코드 수정 없이 시스템 관리자들이 프로젝트 코드를 쉽게 배치
            - 대부분의 Paas가 설정을 환경 변수를 통해 이용하기를 추천하고 있고 이를 위한 기능을 내장햠
- 여러 개의 requrirements파일 이용하기
    - requirements/base.txt, local.txt, staging.txt, production.txt
- settings에서 파일 경로 처리하기
    - ⇒ 장고 세팅 파일에 하드 코딩된 파일 경로를 넣지 말자!!
    - BASE_DIR 같은 경로 세팅을 하는 것이 가장 깔금
    - `tip` diffsettings 명령어: 장고의 기본 설정과 얼마나 다르게 변했는지 보여줌
