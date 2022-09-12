---
title: "ktlint 쉽게 적용하기"
excerpt: "kotlin 공식 코드 스타일로 검사하는 ktlint 쉽게 적용하기"

categories:
  - Android
tags:
  - [Android]

permalink: /android/ktlint-setup/

toc: true
toc_sticky: true

date: 2022-08-24
last_modified_at: 2022-08-29
---

## Ktlint
* Ktlint는 Kotlin의 공식 가이드 기반으로 코드 스타일을 검사합니다.

### Ktlint 구성 하기 
* 해당 예제는 Ktlint plugin중 가장 많이 사용되는 [jlleitschuh.gradle.ktlint](https://github.com/JLLeitschuh/ktlint-gradle) 를 사용합니다. 
 
#### 1) .editconfig 구성하기
* root 경로에 .editconfig 추가 [editconfig](https://plugins.jetbrains.com/plugin/7294-editorconfig/)

```
root = true

[*]
indent_style = space
indent_size = 2

end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{java,kt,kts,scala,rs,xml,kt.spec,kts.spec}]
indent_size = 4

[*.{kt,kts}]
ij_kotlin_imports_layout=*

[{Makefile,*.go}]
indent_style = tab
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[gradle/verification-metadata.xml]
indent_size = 3
```

#### 2) project 단위 build.gradle.kts 설정
```
plugins {
  id("org.jlleitschuh.gradle.ktlint") version "<current_version>"
}

repositories {
  // Required to download KtLint
  mavenCentral()
}

subprojects {
    apply(plugin = "org.jlleitschuh.gradle.ktlint") // Version should be inherited from parent

    repositories {
        // Required to download KtLint
        mavenCentral()
    }

    // Optionally configure plugin
    configure<org.jlleitschuh.gradle.ktlint.KtlintExtension> {
        debug.set(true)
    }
}
```

#### 3) gradle.properties 설정
```
# Kotlin code style for this project: "official" or "obsolete":
kotlin.code.style=official
```

#### 4) editor 설정
* preference → editor → kotlin → set form에서 kotlin style guide를 선택

<img src="/assets/images/posts_img/ktlint-setup/ktlint1.png" width="600">
 
#### 5) Task 실행해서 Intellij에 적용
```
./gradlew ktlintApplyToIdea
```

#### 6) POSIX 명세에 따른 파일의 마지막에 개행문자 자동 추가 설정

<img src="/assets/images/posts_img/ktlint-setup/ktlint2.png" width="600">

이제 자동 정렬 단축키로 Ktlint 적용이 가능합니다!

### Git Hook 구성 하기
* Git에 Push 및 Commit 등의 이벤트가 발생 하기 전과 후에 호출되는 스크립트가 Hook입니다.
* 우리는 Hook에 Ktlint을 추가해 Commit 전에 코드검사를 진행하고자 합니다.

#### ktlinCheck 태스크를 등록 
```
./gradlew addKtlintCheckGitPreCommitHook
```

#### ktlinFormat 태스크를 등록 
```
./gradlew addKtlintFormatGitPreCommitHook
```
ktlinFormat 태스크는 스타일에 맞지 않는 코드를 자동으로 바꿔주지만 파일을 삭제하는 경우가 있습니다. 

그래서 ktlinCheck 태스크로 수동검사 후 직접 수정하는 것을 권장합니다. 

#### Hook 삭제 
```
rm .git/hooks/pre-commit
```
