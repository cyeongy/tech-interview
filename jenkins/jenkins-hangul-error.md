# 01. 젠킨스 한글 에러

## 젠킨스 한글 에러



포스팅을 위해 로컬에서 재현을 시도해보았는데 한글 윈도우라 재현이 안되는 듯 하다. 요 며칠 전, Window 2016 Server 영어 버전에서 영어로 Job 생성 후 여러번의 테스트를 통해 세팅을 완료하고, 관리 편의성을 위해 한글로 Folder를 생성, 이동한 적이 있다.

당시 에러코드가 정확하게 기억이 나진 않지만 jenkins work directory를 찾을 수 없었다는 에러를 뱉어냈다. 당연히 잘 작동하던 파이프라인이 작동이 안되니 패닉에 빠졌고 이 후 JOB NAME이 아니라 DISPLAY NAME을 한글로 수정해야 문제없이 작동하는 것을 확인할 수 있었다.

어떤 빌드 머신에서는 작동하고 안하는 것을 반복하는 것을 볼 때, 준엄한 Java 프로그램 위에서는 한글 경로를 설정하는 우를 범하지 않았으면한다. 이 후, 자바로 돌아가는 nexsu3으로 리포지토리와 blob 스토어를 구축할 때는 의도적으로 한글을 배제하고 이름을 지었다.
