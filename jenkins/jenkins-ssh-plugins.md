# Jenkins SSH plugins 문제점

![](https://velog.velcdn.com/images/cyeongy/post/45f08121-f04b-4f11-aa99-c623792ad11f/image.png)

## 문제점

### JavaPath를 찾지 못한다.

별도로 설정하지 않으면 SSH plugin은 노드 관리 상단에 위치한 $Remote root directory/jdk/java 에서 자바를 찾는다. 당연히 jenkins agent를 실행하려면 master에서 다운로드한 remoting.jar를 실행시켜야하는데 java를 인식하지 못하니 찾을 수 있을리 만무하다.

#### 대처

우선 상기 이미지처럼 Launch agents via SSH 방식에 제공되는 JavaPath에 Java 경로를 넣어서 실행시켰다.

***

#### 원인

이 후 여러가지 테스트를 진행해보며 문제점을 찾을 수 있었다. -JVM Options에 -Djava.awt.headless=true를 넣으면 SSH가 실행되면서 환경변수 등을 출력해주어 편하게 확인할 수 있었다.

### zprofile, .zshrc 파일을 읽지 않는다.

리눅스 환경은 별도로 실험을 해보지 않았고 맥의 기본 쉘이 zsh이니 리눅스에서 chsh -s /bin/zsh 혹은 맥에서 bash로 바꾸면 서로 비교할 수 있을 지도 모르나 별도로 테스트하진 않았다. 맥북은 기본적으로 zsh을 사용하고, bash나 sh이라도 .profile .bashrc 문제는 동일할 것으로 보인다.

.zprofile과 .zshrc에 JAVA\_HOME, PATH, PYTHONPATH 등을 export 하더라도 SSH 연결에서 /bin:/usr/bin:/usr/sbin 만 작동했다.

당연히 JDK가 어디있는지, JAVA는 어디 있는지 확인할 수 있을리 만무하다

#### 해결책

아래의 쉘스크립트를 Prefix Start Agent Command에 넣어서 해결하였다.

```bash

# .zprofile과 .zshrc에 환경변수 설정이 분리된 경우
# ex) 기본 PYTHONPATH를 그대로 사용해서 PYTHONPATH가 .zprofile에 있을 떄
# "PATH=/Library/Frameworks/Python.framework/Versions/3.7/bin:$PATH" >> .zprofile

source $HOME/.zprofile && source $HOME/.zshrc &&

# .zshrc에 다 있을 때, 혹은 .zprofile을 생성하지 않았을 때 (삭제하거나)
source $HOME/.zshrc &&

```

마지막에 &&를 넣어서 명령어가 이어지게 해야한다. 콘솔 로그를 보면 아래와 같은 한 줄의 명령어로 젠킨스 agent를 실행하게된다. &&를 잘 넣어서 에러가 나지 않게 조심하자

```bash
${Prefix Start Agent Command} cd "${Remote root directory}" && java ${JVM Options} -workdir ~~~~
```

## 정리

아무래도 젠킨스 마스터 (Window)와 에이전트(OSX)를 연결해서 맥북에서 빌드를 진행했고, 이런 복잡한 환경이 위와 같은 문제를 발생시킨 것일 수도 있다. 다음 포스트는 Jenkins의 python plugins에서 발생한 문제점을 작성해야겠다.
