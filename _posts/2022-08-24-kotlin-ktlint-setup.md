---
title: "ktlint"
excerpt: "ktlint 쉽게 적용하기"

categories:
  - Kotlin
tags:
  - [Kotlin]

permalink: /kotlin/ktlint-setup/

toc: true
toc_sticky: true

date: 2022-08-24
last_modified_at: 2022-08-29
---

## Ktlint
<https://github.com/JLLeitschuh/ktlint-gradle> 

### Ktlint? 
* Ktlint는 Kotlin의 공식 가이드 기반으로 코드 스타일을 검사합니다.

### Ktlint 구성 하기 
* 해당 예제는 Ktlint plugin중 가장 많이 사용되는 jlleitschuh.gradle.ktlint 를 사용합니다. 
 
#### 1. .editconfig 구성하기
* root 경로에 .editconfig 추가 [editconfig]: https://plugins.jetbrains.com/plugin/7294-editorconfig/

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

#### 2.  project 단위 build.gradle.kts 설정
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

#### 3. gradle.properties 설정
```
# Kotlin code style for this project: "official" or "obsolete":
kotlin.code.style=official
```

#### 4. editor 설정
* preference → editor → kotlin → set form에서 kotlin style guide를 선택

<img src="/assets/images/posts_img/ktlint-setup/ktlint1.png" width="400">
 


