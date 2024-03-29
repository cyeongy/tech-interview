# Spec Repo

## offline 설치하기

코코아팟 라이브러리를 구축하려면 두 종류의 리포지토리가 필요하다. 하나는 Podspec을 저장하는 Spec 리포지토리이고, 다른 하나는 실제 소스코드가 저장된, Podspec이 가리키고 있는 Library 리포지토리이다.

실제 개발을 하면서 pod lib create를 통해 만들고 적용할 수도 있지만, 오프라인, 망분리, 사내망, 폐쇄망 환경에서 cocoapods.org에 위치한 Public Repo를 카피하는 것이 목표이니 빌드를 위한 미러 서버를 구축한다 생각하면 좋을 것 같습니다.

## Spec Repo

Spec 리포지토리는 cocoapods에서 사용할 수 있는 소스 코드 Libarary들의 스펙과 url을 알려주는 저장소이다. cocoapods의 [Public Spec Repo](https://github.com/CocoaPods/Specs) 은 압축된 아카이빙 파일로 500MB가 넘는 데이터를 가지고 있기 때문에 지난 포스트에서 언급한 [CDN](https://github.com/CocoaPods/cdn.cocoapods.org) 에서 Spec을 제공합니다. 실제 캐시파일을 보면 podspec.json 형식으로 저장되는데 podspec.json 형식도 가능하다면, 개인적으로 podspec 작성이 조금 더 쉬워질 것 같기도 합니다.

### Spec 디렉토리 구조

Spec 디렉토리의 경우 아래의 포맷과 예제를 보고 작성하시면됩니다.

```
└── [SPEC REPO 이름, git slug name]/
    ├── [REPO NAME]/
    │   ├── [버전]/
    │   │   └── [REPO NAME].podspec
    │   └── [버전]/
    │       └── [REPO NAME].podspec
    └── [REPO NAME]/
        ├── [버전]/
        │   └── [REPO NAME].podspec
        └── [버전]/
            └── [REPO NAME].podspec
```

```
└── my-spec-repo/
    ├── Toast-Swift/
    │   ├── 4.0.1/
    │   │   └── Toast-Swift.podspec
    │   └── 5.0.1/
    │       └── Toast-Swift.podspec
    └── RxSwift/
        ├── 5.1.3/
        │   └── RxSwift.podspec
        └── 6.5.0/
            └── RxSwift.podspec
```

### Git 사용하기

형상 관리 도구(SCM)이 git만 있는 것은 아니지만, 2014년 [Issue #1747](https://github.com/CocoaPods/CocoaPods/issues/1747)에서 Subversion의 사용 가능성을 물었고 실제 [Cocoapods-Downloader/subversion.rb](https://github.com/CocoaPods/cocoapods-downloader/blob/master/lib/cocoapods-downloader/subversion.rb) 를 통해 svn의 사용은 가능하나 지원 중단을 밝혔습니다. 실제로 22년에도 지속적인 코드 개선이 이루어지는 git.rb와 달리 subversion.rb의 경우 16년을 마지막으로 추가적인 개선이 이루어지지 않았고 플러그인 개발자도 없는 상태입니다.

> svn 플러그인 개발 리포지토리의 마지막 커밋 2016년 11월 https://github.com/dustywusty/cocoapods-repo-svn

해당 포스팅에서는 Git만을 사용하되, Git을 설치한 상태에서 파일시스템에서 꼼수로 관리하는 법을 알려드리겠습니다.

## Spec Repo 추가하기

```bash
pod repo add [SPEC REPO 이름] [SPEC REPO URL]

# 예제 코드
pod repo add my-repo 'https://github.com/cyeongy/my-specs.git'


pod repo list
# TYPE: 'my-repo' (git)
# URL: 'https://github.com/cyeongy/my-specs.git'
# PATH: '~/.cocoapods/specs/my-repo'
```

해당 명령어를 사용하여 spec을 지정할 수 있습니다. 이전 포스팅의 podfile 작성 시 (1) 항목의 source 경우 URL을 작성하는데 SPEC REPO의 이름으로 대체할 수 있습니다. pod repo add를 통해 git 주소를 등록했다면, 아래의 두 코드는 같은 동작을 합니다.

```rb
source 'my-repo'
source 'https://github.com/cyeongy/my-specs.git'
```

### Spec Repo 추가하기 (파일 꼼수)

위에서 작성한 SPEC REPO URL의 경우 굳이 git remote 저장소에 업로드 하지 않아도 절대 경로로 디렉토리를 지정해주면 그대로 사용할 수 있습니다.

```
# 예제 코드
cd /Users/cyeongy/Desktop/test/local-my-spec

# 로컬 git 저장 폴더로 만듭니다.
git init
git add .
git commit -m "로컬 스펙 저장소"

pod repo add my-repo '/Users/cyeongy/Desktop/test/local-my-spec'

pod repo list
# TYPE: 'my-repo' (git)
# URL: '/Users/cyeongy/Desktop/test/local-my-spec'
# PATH: '~/.cocoapods/specs/my-repo'
```

```rb
source '/Users/cyeongy/Desktop/test/local-my-spec'
```

다만 spec을 수정하기 위해 \~/.cocoapods/specs 폴더 내부의 파일을 직접 수정하는 경우 pod repo update 혹은 pod install --repo-update를 사용하면 git 저장소의 내용이 덮어쓰기 될 수 있습니다. git init으로 생성한 로컬 저장소에 git add & commit으로 저장 후 pod repo update를 하는 것이 좋습니다.
