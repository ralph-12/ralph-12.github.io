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
GitHub - JLLeitschuh/ktlint-gradle: A ktlint gradle plugin 

### Ktlint? 
Ktlint는 Kotlin의 공식 가이드 기반으로 코드 스타일을 검사합니다. (Kotlin coding conventions) 

개발자는 Git commit 전이나 빌드 단계에서 Koltin의 가이드에서 벗어나는지 확인하여 수정할 수 있습니다.

### Ktlint 구성 하기 
 Knotkoq은 Ktlint plugin중 가장 많이 사용되는 jlleitschuh.gradle.ktlint 를 사용합니다. 
