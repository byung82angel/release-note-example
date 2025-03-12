수정3
# 릴리즈 노트 자동 생성 예제
이 예제는 릴리즈 노트를 자동으로 생성하는 예제입니다. 릴리즈 노트 자동 생성을 위해 Github Action을 활용해야 하며, Pull Request와 함께 Label을 활용하는 방법에 대해 소개합니다.

## Warning
drafter.yaml 파일은 가장 마지막 tag를 기준으로 릴리즈 버전을 체크합니다. 만약 릴리즈 한 tag 정보가 없을 경우, 혹은 처음 release 노트 생성할 때 base 태그가 없다면 `v0.1.0`으로 생성됩니다. 따라서 release 노트 생성 시 필요하다면 tag를 수정해주어야 합니다.

## Addtional Labels

* major : Auto increment `major` version when release
* minor : Auto increment `minor` version when release
* patch : Auto increment `patch` version when release

## Workflows - .github/workflows/drafter.yaml

release 브랜치에서 main 브랜치로 merge할 때 릴리즈 노트 draft 버전을 생성합니다. draft 버전의 문서는 자동으로 publish 되지 않기 때문에 생성된 문서를 한 번 더 확인 후 publish 가능합니다.


```yaml
name: Draft new release
on:
  push:
    branches:
      - main
jobs:
  build:
    permissions:
      contents: write
      pull-requests: write
    runs-on: ubuntu-latest
    steps:
    - name: Release
      # reference : https://github.com/release-drafter/release-drafter
      uses: release-drafter/release-drafter@v6
      with:
        config-name: drafter-config.yaml
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```


## Workflows - .github/drafter-config.yaml

Draft 문서를 만드는 Template입니다. commitish는 default가 master이기 때문에 main 브랜치로 변경해주었고, 각각 카테고리별 문서를 자동으로 생성해주기 위해 label로 구분할 수 있습니다.


```yaml
name-template: 'v$RESOLVED_VERSION 🌈'
tag-template: 'v$RESOLVED_VERSION'
commitish: main
categories:
  - title: '🚀 Features'
    labels:
      - 'feature'
      - 'enhancement'
  - title: '🐛 Bug Fixes'
    labels:
      - 'fix'
      - 'bugfix'
      - 'bug'
  - title: '🧰 Maintenance'
    label: 'chore'
change-template: '- $TITLE @$AUTHOR (#$NUMBER)'
change-title-escapes: '\<*_&' # You can add # and @ to disable mentions, and add ` to disable code blocks.
version-resolver:
  major:
    labels:
      - 'major'
  minor:
    labels:
      - 'minor'
  patch:
    labels:
      - 'patch'
  default: patch
template: |
  ## Changes

  $CHANGES
```
