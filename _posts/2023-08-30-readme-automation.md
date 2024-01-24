---
layout: post
title: "[Git] README에 파일 목차와 링크 생성하기(file tree)"
subtitle:
categories: Git
tags: []
---

결과 미리보기
![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2023-08-30-blog0.png?raw=true)

---

매일 기록하기를 마음먹고 github에 TIL repository를 생성했습니다. 기존에 있던 알고리즘 repository도 TIL 아래로 옮기고 이번주 부터 읽기 시작한 두 권의 책에 대한 기록도 남기고 나니 README에 파일 목차(tree)와 링크를 걸어주면 나중에 보기 편하겠다는 생각이 들었습니다. 찾아보니 많은 분들이 이미 그렇게 하고 계셨고 이 작업을 자동화 해 두신 분도 계시지않을까?하는 생각이 들었습니다. 검색하자마자 역시나(!!) 제가 하고 싶은 일과 98% 유사한 작업을(유사한 환경에서) 이미 해두신 분을 찾았습니다. 

[potados님의 블로그](https://blog.potados.com/dev/directory-listing-in-readme/)를 보고 차근히 따라하고, 제 환경에 맞춰 2%의 변화를 준 부분을 기록에 남깁니다. 
제가 스크립트를 실행한 환경은 macOS 13.3.1 입니다.


## 파일 목차(링크) markdown 형식으로 생성하기
- tree 와 gsed 이용합니다.
- 1), 2)번은 tree와 gsed의 동작을 살펴보는 것으로 이미 잘 아시는 분들은 3)번부터 읽으셔도 무방합니다 :)

### 1) tree

파일 목차를 만들고 싶은 프로젝트 폴더에서 tree 명령어를 입력해봅니다. 
`command not found: tree`가 뜬다면 설치가 되지 않은 것이니 터미널에 `brew install tree` 명령으로 설치해줍니다. 

```shell
$ tree
```

![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2023-08-30-blog1.png?raw=true)

그럼 현재 프로젝트의 트리를 볼 수 있습니다. 하지만 우리가 원하는 것은 프로젝트 목차와 링크!입니다. 제목만 보고 글로 바로 가고 싶거든요. 

### 2) gsed

`gsed` 가 없을 경우 역시 터미널에 `brew install gnu-sed` 명령으로 설치해줍니다.

```shell
$ tree -tf --noreport -I '*~' --charset ascii $1 | gsed -e '1d' -e 's/    /   |/g' -e 's/| \+/  /g' -e 's/[|`]-\+/ */g' -e 's:\(* \)\(\(.*/\)\([^/]\+\)\):\1[\4](\2):g' 
```
위 명령이 잘 실행되면 아래와 같이 변환이 됩니다.

![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2023-08-30-blog2.png?raw=true)

gsed 명령을 한 줄 씩 보면 아래와 같습니다. 

```shell
gsed -e 'id'                                              # . 으로 나오는 첫 줄을 삭제합니다.
gsed -e 's/    /   |/g'                                   # 연속된 공백 문자 4칸을 공백문자 3칸+'|'로 대치(트리 계층의 통일성을 위해)
gsed -e 's/| \+/  /g'                                     # 중복된 '|' 삭제
gsed -e 's/[|`]-\+/ */g'                                  # '|--' 를 '*'로 대치
gsed -e 's:\(* \)\(\(.*/\)\([^/]\+\)\):\1[\4](\2):g'      # 링크를 걸기
```

### 3) README 템플릿 생성

root directory에 아래와 같이 템플릿 파일을 만들고 `.readme_template.md` 으로 저장했습니다. `__PROJECT_TREE__` 부분이 파일 목차로 대치될 부분입니다. 다른 내용은 자유롭게 작성해주시면 됩니다. 

```markdown
# TIL

## Today I Learned- 매일 보고 들은 모든 것

__PROJECT_TREE__
```

### 4) README 업데이트 스크립트

`update-readme` 파일을 아래와 같이 생성합니다.

```shell
#!/bin/bash

function generate_project_tree() {
  # Original from https://stackoverflow.com/a/35889620/11929317
  # This is a ported version for mac
  IGNORED="venv|update-readme|new-boj|README.md"
  SED_FOR_MAC="gsed"

  if [[ "$OSTYPE" == "darwin"* ]]; then
    if command -v $SED_FOR_MAC >/dev/null; then
      SED=$SED_FOR_MAC
    else
      echo "$SED_FOR_MAC not installed!" && exit 1
    fi
  else
    SED="sed"
  fi

  tree -tf --noreport -I '*~' --charset ascii $1 |
    $SED -e '1d'  |
    $SED -e 's/    /   |/g' |
    $SED -e 's/| \+/  /g'  |
    $SED -e 's/[|`]-\+/ */g' |
    $SED -e 's:\(* \)\(\(.*/\)\([^/]\+\)\):\1[\4](\2):g' |
    grep -vE "img|update-readme"
}

function generate_readme() {
  readme="$1/README.md"
  readme_template="$1/.readme_template.md"

  perl -p0e 's/__PROJECT_TREE__/`cat`/se' "$readme_template" > "$readme"
}

cd "$(dirname "$0")" || exit 1
generate_project_tree . | generate_readme .
```

`update-readme` 가 어떤 일을 하는지 `generate_project_tree`, `generate_readme`를 보며 알아봅시다. 

**generate_project_tree**
- mac에서 SED 변수에 gsed 를 저장해줍니다.
- 다음 2)에서 실행한 것과 동일한 코드를 실행하여 동일한 결과(markdown tree & link)를 얻습니다.
generate_project_tree 의 output은 파이프라인 '|'을 통해 generate_readme로 전해집니다. 

**generate_readme**
- perl 을 이용해 .readme_template.md의 `__PROJECT_TREE__`부분을 generate_project_tree 의 output으로 대치하여 README.md에 저장합니다.
- generate_project_tree 의 output은 `cat`으로 읽어옵니다. 


### 5) git hook(README 업데이트 자동화)

다 왔습니다!! 이제 git에 commit하면 update-readme 파일을 실행하도록 하는 hook을 등록합니다. 
프로젝트의 `.git/hooks`에 `pre-commit` 파일을 아래와 같이 생성하고 `chmod +x pre-commit`으로 실행 권한을 줍니다. 

```shell
#!/bin/bash

GIT_DIR=$(git rev-parse --git-dir)
cd "$GIT_DIR/.." || exit 1 
bash ./update-readme
```

