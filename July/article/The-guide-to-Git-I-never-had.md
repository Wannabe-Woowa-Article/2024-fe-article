## 🔗 [The guide to Git I never had](https://medium.com/@jake.page91/the-guide-to-git-i-never-had-a89048d4703a)

### 🗓️ 번역 날짜: 2024.07.07

### 🧚 번역한 크루: 러기(박정우)

---

## 내가 원했던 Git 가이드

![git](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*SRMPPL8gPMrJLMqaGo0mCA.png)
🩺의사들은 청진기를, 🔧 메카닉들은 스패너를 사용한다면, 👨‍💻 우리 개발자들은 Git을 사용합니다.

Git이 코드 작업에 얼마나 필수적인지, 사람들이 CV(Curriiculum Vitea = resume)나 기술스택에 그것을 거의 포함하지 않는다는 사실을 알고계신가요? 이미 알고 있거나 최소한 다룰 줄 안다는 가정이 깔려 있습니다. 하지만 정말로 알고 있나요?

Git은 버전 관리 시스템(VCS)입니다. 우리가 다른 사람들과 함께 코드를 저장하고 변경하며 협력할 수 있게 해주는 보편적인 기술입니다.

> 🚨 글을 소개하기 앞서, Git은 방대한 주제라는 점을 지적하고 싶습니다. Git에 관한 책이 쓰여졌고, 학술 논문으로 오해될 수 있는 블로그 글도 있습니다. 제가 여기서 하려는 것은 그런 것이 아닙니다. 저는 Git 전문가가 아닙니다. 여기서의 목표는 제가 Git을 배울 때 원했던 Git 기본 글을 작성하는 것입니다.

개발자로서 우리의 일상은 코드 읽기, 쓰기, 검토를 주로 이룹니다. Git은 분명하게 우리가 사용하는 가장 중요한 도구 중 하나라고 할 수 있습니다. Git이 제공하는 기능과 기능을 숙달하는 것은 개발자로서 자신에게 할 수 있는 최고의 투자 중 하나입니다.

그럼 시작해봅시다.

![let's go git](https://miro.medium.com/v2/resize:fit:960/format:webp/0*xs6eNc-gxLusI4AJ.gif)

> 특정 명령어에 대해 제가 놓쳤거나 더 자세히 다뤄야 할 부분이 있다고 생각되면 아래 댓글로 알려주세요. 그러면 해당 포스트를 그에 맞게 업데이트하겠습니다. 🙏

## 이 주제에 대해 이야기하는 동안

Git 기술을 활용하고 Glasskube에 기여하고 싶다면, 저희는 2월에 공식 출범했으며 Kubernetes 패키지 관리의 당연하고 기본적인 솔루션이 되는 것을 목표로 하고 있습니다. 여러분의 지원으로 우리는 나아가고 있습니다. 저희를 도와주는 가장 좋은 방법은 저희 GitHub에 Star 표시를 해주시는 겁니다. ⭐

![cat](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OTGGI6NNJyE-s83Z.jpeg)

## 기초를 다져봅시다

Git이 여러분을 Peter Griffin(아래 이미지)처럼 느끼게 한 적이 있나요?

Git을 올바르게 배우지 않으면 계속해서 고개를 갸웃거리거나, 같은 문제에 막히거나, 터미널에 또 다른 병합 충돌이 나타나는 날을 후회하게 될 위험이 있습니다. 이러한 일이 발생하지 않도록 기본적인 Git 개념을 정의해 봅시다.

![peter griffin](https://miro.medium.com/v2/resize:fit:960/format:webp/0*WaBYfIYf-bPHRyog.gif)

## Branches

Git 저장소에서는 일반적으로 "main" 또는 "master"(이제는 사용되지 않음)라고 명명된 주요 개발 라인을 찾을 수 있으며, 여기서 여러 분기가 갈라집니다. 이 분기들은 동시에 여러 작업 흐름을 나타내며, 개발자들이 동일한 프로젝트 내에서 여러 기능이나 수정을 동시에 처리할 수 있도록 해줍니다.

![git branches](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*fIKhQXkHzlr_yiW0.png)

## Commits

Git 커밋은 업데이트된 코드의 묶음으로, 특정 시점에서 프로젝트 코드의 스냅샷을 캡처합니다. 각 커밋은 마지막 커밋이 기록된 이후에 이루어진 변경 사항을 기록하여, 프로젝트 개발 여정의 포괄적인 기록(history)를 구성합니다.

![git commit](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*5LF0U6qy72fBCzps.png)

커밋을 참조할 때 일반적으로 고유하게 식별되는 암호 해시값을 사용합니다.

예시:

```json
git show abc123def456789
```

위 명령어는 해당 해시의 커밋에 대한 자세한 정보를 보여줍니다.

## Tags

GIt Tags는 깃 기록(history)의 이정표 역할을 하며, 일반적으로 프로젝트 개발의 중요한 milestons, 예를들어, release, versions,눈에 띄는 commits들을 표시합니다. 이러한 태그들은 특정 시점을 표시하는데 매우 유용하며, 종종 프로젝트 여정의 시작점이나 주요 성과를 나타냅니다.

![git tags](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*2sw41eQGJ3JufyA6.png)

## HEAD

현재 체크아웃된 브랜치에서 가장 최근의 커밋은 HEAD로 표시되며, 이는 저장소 내의 모든 참조를 가리키는 포인터 역할을 합니다. 특정 브랜치에 있을 때, HEAD는 해당 브랜치의 최신 커밋을 가리킵니다. 때로는 브랜치의 끝을 가리키는 대신, HEAD가 특정 커밋을 직접 가리킬 수 있습니다. (분리된 HEAD 상태)

## Stages

Git Stages를 이해하는 것은 Git 워크플로를 탐색하는 데 중요합니다. 이 단계는 파일에 대한 변경 사항이 저장소에 커밋되기 전에 발생하는 논리적 전환을 나타냅니다. Git 단계의 개념을 자세히 살펴봅시다.
![git stages](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*WkcWFYgsKpYyj2b3.png)

### 작업 디렉토리 👷

작업 디렉토리는 프로젝트를 위해 파일을 편집, 수정, 생성하는 곳입니다. 로컬 컴퓨터에서 파일의 현재 상태를 나타냅니다.

### 스테이징 영역 🚉

스테이징 영역은 변경 사항을 커밋하기 전에 준비하는 임시 영역입니다. 일종의 보류 구역 또는 사전 커밋 구역입니다.

유용한 명령어: `git add` 또한 `git rm`을 사용하여 변경 사항을 스테이징 해제할 수 있습니다.

### 로컬 저장소 🗄️

로컬 저장소는 Git이 커밋된 변경 사항을 영구적으로 저장하는 곳입니다. 프로젝트의 히스토리를 검토하고, 이전 상태로 되돌리며, 동일한 코드베이스에서 다른 사람과 협업할 수 있게 해줍니다.

스테이징 영역에서 준비된 변경 사항을 커밋하려면: `git commit`

### 원격 저장소 🛫

원격 저장소는 일반적으로 서버(GitHub, GitLab 또는 Bitbucket 등)에 호스팅되어 프로젝트를 다른 사람과 공유하고 협업할 수 있는 중앙 집중식 위치입니다.

`git push` 및 `git pull`과 같은 명령어를 사용하여 로컬 저장소에서 원격 저장소로 커밋된 변경 사항을 푸시/풀 할 수 있습니다.

### Git 시작하기

어디서부터 시작해야 하는지 알아봅시다. Git에서는 `workspace`에서 시작해야 합니다. 기존 저장소를 포크하거나 클론하여 해당 workspace의 사본을 가질 수 있습니다. 또는 완전히 새로운 로컬 폴더에서 시작하는 경우 `git init` 명령어를 사용하여 해당 폴더를 Git 저장소로 변환해야 합니다. 다음 단계는 간과해서는 안 될 중요한 단계로,credentials(자격 증명)을 설정하는 것입니다.

![git layout](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*7FR2Oh-61AxXCVzW.png)

## 자격 증명 설정

원격 저장소로 푸시(pushing) 및 풀링(pulling)할 때마다 사용자 이름과 비밀번호를 입력하지 않으려면 다음 명령어를 실행하세요:

```bash
git config --global credential.helper store
```

처음 원격 저장소와 상호 작용할 때, Git은 사용자 이름과 비밀번호를 입력하도록 요청할 것입니다. 그 후로는 다시 요청하지 않습니다.

자격 증명은 `.git-credentials` 파일에 평문 형식으로 저장된다는 점을 유의해야 합니다.

설정된 자격 증명을 확인하려면 다음 명령어를 사용할 수 있습니다:

```bash
git config --global credential.helper
```

## 브랜치 작업

로컬에서 작업할 때 현재 어느 브랜치에 있는지 아는 것이 중요합니다. 다음 명령어들이 유용합니다:

```bash
# 로컬 저장소의 변경 사항을 보여줍니다.
git branch
# 브랜치를 직접 생성하려면
git branch feature-branch-name
```

브랜치 간 전환하려면:

```bash
git switch
```

또한 브랜치 간 전환과 더불어, 다음 명령어를 사용할 수 있습니다:

```bash
git checkout
# 아직 생성되지 않은 브랜치로 전환하는 바로 가기 -b 플래그 사용
git checkout -b feature-branch-name
```

저장소의 상태를 확인하려면:

```bash
git status
```

현재 브랜치를 항상 명확하게 보기 위해 터미널에서 직접 확인하는 것이 좋습니다. 이를 돕는 여러 터미널 애드온이 있습니다. 여기 하나의 예시가 있습니다.

[Add Git Branch Name to Terminal Prompt (Linux/Mac)](https://gist.github.com/joseluisq/1e96c54fa4e1e5647940)

![git terminal addon](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*5_tQsZ-hqYh5jDHg.png)

## 커밋 작업

커밋 작업 시 `git commit -m` 명령어로 변경 사항을 기록하고, `git amend` 명령어로 가장 최근 커밋을 수정하며, 커밋 메시지 규칙을 준수하는 것이 좋습니다.

```bash
git commit -m "의미 있는 메시지"
```

최근 커밋에 변경 사항이 있는 경우, 새로운 커밋을 만들 필요 없이 `--amend` 플래그를 사용하여 스테이징된 변경 사항으로 가장 최근 커밋을 수정할 수 있습니다.

```bash
# 변경 사항을 만듭니다.
git add .
git commit --amend
# 필요한 경우 기본 텍스트 편집기를 열어 커밋 메시지를 수정할 수 있습니다.
git push origin your_branch --force
```

> ⚠️ `--force`를 사용할 때는 주의해야 합니다. 타겟 브랜치의 히스토리를 덮어쓸 수 있기 때문입니다. 주로 main/master 브랜치에는 적용하지 않는 것이 좋습니다.

일반적으로 자주 커밋하는 것이 더 낫습니다. 진행 상황을 잃거나 스테이징되지 않은 변경 사항을 실수로 초기화하는 것을 방지할 수 있습니다. 이후에 여러 커밋을 스쿼시하거나 인터랙티브 리베이스(interactive rebase) 통해 히스토리를 재작성할 수 있습니다.

>

`git log` 명령어를 사용하여 가장 최근 커밋부터 시작하여 커밋 역순으로, 시간 순으로 커밋 목록을 보여줍니다,

## 히스토리 조작

히스토리 조작에는 몇 가지 강력한 명령어가 포함됩니다. 리베이스는 커밋 히스토리를 재작성하고, 스쿼싱은 여러 커밋을 하나로 결합하며, 체리 피킹은 특정 커밋을 선택합니다.

## 리베이스와 머지

리베이스와 머지를 비교하는 것이 의미가 있습니다. 그들의 목표는 동일하지만 이를 달성하는 방법은 다릅니다. 중요한 차이점은 리베이스가 프로젝트의 히스토리를 재작성한다는 것입니다. 명확하고 이해하기 쉬운 프로젝트 히스토리를 중시하는 프로젝트에 적합한 선택입니다. 반면, 머지는 새로운 병합 커밋을 생성하여 두 브랜치의 히스토리를 유지합니다.

리베이스하는 동안, 기능 브랜치의 커밋 히스토리가 재구조화되면서 메인 브랜치의 HEAD로 이동됩니다.

![rebase and merge](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*K3zOgE4K_nHx89uQ.png)

작업 흐름은 매우 간단합니다.

리베이스하려는 브랜치에 있고 원격 저장소에서 최신 변경 사항을 가져왔는지(fetch) 확인합니다:

```bash
git checkout your_branch
git fetch
```

이제 리베이스할 브랜치를 선택하고 다음 명령어를 실행합니다:

```bash
git rebase upstream_branch
```

리베이스 후, 브랜치가 이미 원격 저장소에 푸시된 경우 변경 사항을 강제로 푸시해야 할 수도 있습니다:

```bash
git push origin your_branch --force
```

> ⚠️ `--force`를 사용할 때는 주의해야 합니다. 타겟 브랜치의 히스토리를 덮어쓸 수 있기 때문입니다. 주로 main/master 브랜치에는 적용하지 않는 것이 좋습니다.

## Squashing

Git 스쿼싱은 여러 커밋을 단일하고 일관된 커밋으로 압축하는 데 사용됩니다.

![squash](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*E_aIeGBb-J-Krud_.png)

개념은 이해하기 쉽고, 특히 리베이스를 사용하는 코드 통합 방식이 유용합니다. 히스토리가 변경되므로, 프로젝트 히스토리에 미치는 영향에 주의를 기울이는 것이 중요합니다.

인터랙티브 리베이스를 사용하여 스쿼시를 수행하는 데 어려움을 겪었던 적이 있습니다. 다행히도 우리를 도와줄 도구들이 있습니다. 제가 선호하는 스쿼싱 방법은 스테이징된 변경 사항을 유지하면서 HEAD 포인터를 X개의 커밋 뒤로 이동시키는 것입니다.

```bash
# 스쿼시하려는 커밋 수에 따라 HEAD~ 뒤의 숫자를 변경하십시오.
git reset --soft HEAD~X
git commit -m "Your squashed commit message"
git push origin your_branch --force
```

> ⚠️ `--force`를 사용할 때는 주의해야 합니다. 타겟 브랜치의 히스토리를 덮어쓸 수 있기 때문입니다. 주로 main/master 브랜치에는 적용하지 않는 것이 좋습니다.

## Cherry-picking

Cherry-picking은 전체 브랜치를 병합하는 것이 바람직하지 않거나 가능하지 않을 때 한 브랜치에서 다른 브랜치로 선택적으로 변경 사항을 포함하는 데 유용합니다. 그러나 잘못 적용하면 중복된 커밋과 분기된 히스토리를 초래할 수 있으므로 신중하게 사용해야 합니다.

![cherry-pick](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*7DjXUtEoc5D5i9a8.png)

이를 수행하려면 먼저 `git log`를 사용하여 가져오려는 커밋의 해시를 식별해야 합니다. 커밋 해시를 식별한 후 다음 명령어를 실행할 수 있습니다:

```bash
git checkout target_branch
git cherry-pick <commit-hash>  # 여러 커밋을 원할 경우 여러 번 수행
git push origin target_branch
```

## 고급 Git 명령어

### 커밋 서명하기

커밋 서명은 Git에서 커밋의 신뢰성과 무결성을 검증하는 방법입니다. 이는 GPG(GNU Privacy Guard) 키를 사용하여 커밋을 암호학적으로 서명함으로써 Git이 커밋 작성자가 실제 작성자임을 확인할 수 있도록 합니다. 다음 단계를 통해 GPG 키를 생성하고 Git이 커밋할 때 해당 키를 사용하도록 구성할 수 있습니다:

```bash
# GPG 키 생성
gpg --gen-key

# Git이 GPG 키를 사용하도록 구성
git config --global user.signingkey <your-gpg-key-id>

# 공개 키를 GitHub 계정에 추가

# -S 플래그로 커밋에 서명
git commit -S -m "Your commit message"

# 서명된 커밋 보기
git log --show-signature
```

### Git reflog

Git 참조에 대해 탐구하지 않았던 주제로, 참조가 저장소 내의 다양한 객체(주로 커밋, 태그 및 브랜치)를 가리키는 포인터라는 것입니다. 참조는 Git 히스토리의 이름이 지정된 지점으로, 사용자가 저장소의 타임라인을 탐색하고 프로젝트의 특정 스냅샷에 접근할 수 있게 해줍니다. Git 참조를 탐색하는 방법을 아는 것은 매우 유용할 수 있으며, `git reflog`를 사용하여 이를 수행할 수 있습니다. 다음은 몇 가지 이점입니다:

- 잃어버린 커밋이나 브랜치 복구
- 디버깅 및 문제 해결
- 실수 되돌리기

### **Interactive rebase**

인터랙티브 리베이스는 커밋 히스토리를 상호작용 방식으로 재작성할 수 있게 해주는 강력한 Git 기능입니다. 이를 통해 커밋을 수정, 재정렬, 결합 또는 삭제한 후 브랜치에 적용할 수 있습니다.

사용하려면 다음과 같은 가능한 작업에 익숙해져야 합니다:

- Pick (“p“): 커밋을 그대로 둡니다.
- Reword (“r“): 커밋 메시지를 수정합니다.
- Edit (“e“): 커밋 내용을 수정합니다.
- Squash (“s“): 커밋을 이전 커밋과 결합합니다.
- Drop (“d“): 커밋을 삭제합니다.

![interactive rebase](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*0JsQJSp-ot6C0O_x.png)

터미널에서 인터렉티브 리베이스를 어떻게 이용하는지에 대한 유용한 비디오를 참고하세요.

[Advanced Git Tutorial - Interactive Rebase, Cherry-Picking, Reflog, Submodules and more](https://www.youtube.com/watch?v=qsTthZi23VE)

## Git을 이용한 협업

### Origin vs Upstream

`origin`은 로컬 Git 저장소와 연관된 기본 원격 저장소로, 저장소를 클론할 때 기본으로 설정됩니다. 저장소를 포크한 경우, 그 포크가 기본적으로 "origin" 저장소가 됩니다.

반면에 `upstream`은 저장소가 포크된 원본 저장소를 의미합니다.

포크한 저장소를 원본 프로젝트의 최신 변경 사항으로 업데이트하려면 `upstream` 저장소에서 변경 사항을 가져와 로컬 저장소에 병합하거나 리베이스합니다.

로컬 Git 저장소와 연관된 원격 저장소를 확인하려면 다음 명령어를 실행하세요:

```bash
git remote -v
```

### 충돌

브랜치를 병합하거나 리베이스하려고 할 때 충돌이 발생하면 당황하지 마세요. 이는 저장소의 동일 파일이나 파일의 다른 버전 간에 충돌하는 변경 사항이 있다는 것을 의미하며, 대부분의 경우 쉽게 해결할 수 있습니다.

![git merge](https://miro.medium.com/v2/resize:fit:800/format:webp/0*i6L16rbIkHv8dEk2.gif)

보통 충돌 파일 내에서 Git이 충돌 마커 <<<<<<<, ======= 및 >>>>>>>를 삽입하여 충돌하는 부분을 강조 표시합니다. 유지할 변경 사항을 결정하고, 수정하거나 제거하여 결과 코드가 의미 있고 의도한 기능을 유지하도록 합니다.

충돌 파일에서 수동으로 충돌을 해결한 후, 충돌 마커 <<<<<<<, ======= 및 >>>>>>>를 제거하고 필요한 대로 코드를 조정합니다.

충돌이 해결되면 변경 사항을 저장합니다.

충돌을 해결하는 데 어려움이 있다면, 이 비디오가 잘 설명하고 있습니다.

[How to resolve merge conflicts in Git](https://www.youtube.com/watch?v=xNVM5UxlFSA)

## 인기 있는 Git 워크플로우

![git workflow](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*gGKScUnzECg5XgPR.png)
다양한 Git 워크플로우가 존재하지만, 보편적으로 "최고"인 Git 워크플로우는 없습니다. 대신, 각 접근 방식에는 고유한 장단점이 있습니다. 이러한 다양한 워크플로우를 탐구하여 각각의 강점과 약점을 이해해 봅시다.

![team work](https://miro.medium.com/v2/resize:fit:958/format:webp/0*FuqVTVVErR7upZgI.gif)

### **Feature Branch Workflow** 🌱

각 새로운 기능이나 버그 수정을 별도의 브랜치에서 개발하고 완료되면 메인 브랜치에 병합합니다.

**장점**: 변경 사항의 격리 및 충돌 감소
**단점**: 복잡해질 수 있으며 신중한 브랜치 관리가 필요합니다.

### Gitflow **Workflow** 🌊

Gitflow는 다양한 개발 작업을 위한 사전 정의된 브랜치를 가진 엄격한 브랜칭 모델을 정의합니다.

여기에는 main, develop, feature 브랜치, release 브랜치, hotfix 브랜치와 같은 장기 브랜치가 포함됩니다.

**장점**: 정기 릴리즈와 장기 유지 보수가 필요한 프로젝트에 적합합니다.
**단점**: 작은 팀에는 너무 복잡할 수 있습니다.

### **Forking** **Workflow** 🍴

이 워크플로우에서 각 개발자는 메인 저장소를 클론하지만, 변경 사항을 직접 푸시하는 대신 자신의 포크된 저장소로 푸시합니다. 그런 다음 개발자는 변경 사항을 제안하는 풀 리퀘스트를 생성하여 메인 저장소에 병합하기 전에 코드 리뷰와 협업을 할 수 있습니다.

이 워크플로우는 오픈 소스 Glasskube 저장소에서 협업할 때 사용합니다.

**장점**: 메인 저장소에 대한 직접 쓰기 권한 없이 외부 기여자의 협업을 장려합니다.
**단점**: 포크와 메인 저장소 간의 동기화를 유지하는 것이 어려울 수 있습니다.

### **Pull Request Workflow**⏩

포킹 워크플로우와 유사하지만 포킹 대신 개발자는 메인 저장소에서 직접 기능 브랜치를 생성합니다.

**장점**: 코드 리뷰, 협업 및 팀원 간의 지식 공유를 촉진합니다.
**단점**: 인간 코드 리뷰어에 의존하면 개발 과정에서 지연이 발생할 수 있습니다.

### Trunk 기반 개발 🪵

빠른 반복 및 지속적인 배포에 중점을 둔 팀이라면, 트렁크 기반 개발을 사용하여 개발자는 메인 브랜치에서 직접 작업하여 작은 빈번한 변경 사항을 커밋합니다.

**장점**: 빠른 반복, 지속적인 통합 및 작은 빈번한 변경 사항을 프로덕션에 전달하는 데 중점을 둡니다.
**단점**: 메인 브랜치의 안정성을 보장하기 위해 강력한 자동화된 테스트 및 배포 파이프라인이 필요하며, 엄격한 릴리즈 일정이나 복잡한 기능 개발이 있는 프로젝트에는 적합하지 않을 수 있습니다.

### fork가 뭔가요?

포크는 오픈 소스 프로젝트에서 협업할 때 강력히 추천됩니다. 저장소의 사본에 대한 완전한 제어 권한을 가지며, 변경을 가하거나, 새로운 기능을 실험하거나, 버그를 수정할 수 있습니다.

> 💡 포크된 저장소는 별도의 엔터티로 시작하지만, 원본 저장소와의 연결을 유지합니다. 이를 통해 원본 프로젝트의 변경 사항을 추적하고 다른 사람이 만든 업데이트와 동기화할 수 있습니다.

이로 인해 자신의 origin저장소로 푸시하더라도 변경 사항은 원격에서도 반영됩니다.

### Git **Cheatsheet**

```bash
bash코드 복사
# 저장소 클론하기
git clone <repository_url>

# 커밋을 위한 변경 사항 스테이징
git add <file(s)>

# 변경 사항 커밋
git commit -m "Commit message"

# 변경 사항 원격 저장소로 푸시
git push

# 변경 사항 강제 푸시 (주의)
git push --force

# 작업 디렉토리를 마지막 커밋으로 리셋
git reset --hard

# 새로운 브랜치 생성
git branch <branch_name>

# 다른 브랜치로 전환
git checkout <branch_name>

# 다른 브랜치에서 변경 사항 병합
git merge <branch_name>

# 다른 브랜치로 리베이스 (주의)
git rebase <base_branch>

# 작업 디렉토리 상태 보기
git status

# 커밋 히스토리 보기
git log

# 마지막 커밋 취소 (주의)
git reset --soft HEAD^

# 작업 디렉토리의 변경 사항 폐기
git restore <file(s)>

# 잃어버린 커밋 참조 복구
git reflog

# 커밋을 재배치하기 위한 인터랙티브 리베이스
git rebase --interactive HEAD~3

```

### 유용한 Git 도구 및 리소스

- 인터랙티브 리베이스 도구
  [interactiveRebaseTool](https://github.com/MitMaro/git-interactive-rebase-tool)
- 색상으로 강조된 점진적 차이를 볼 수 있는 Cdiff
  [cdiff](https://github.com/amigrave/cdiff)
- 인터랙티브 Git 브랜칭 연습장
  [Learn Git Branching](https://learngitbranching.js.org/?locale=en_US)

이와 같은 콘텐츠가 마음에 드신다면, GitHub에서 저희에게 Star를 주셔서 지원해주시면 감사하겠습니다. 🙏

![cat](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OTGGI6NNJyE-s83Z.jpeg)
