---
layout: post
title: Server programming with go (1)
excerpt: Go server
date: 2022-3-30 12:00
tags: [Go, HTTP, TDD]
use_math: true
--- 
## Go 서버 프로그래밍

### 목표
- 서버 구현하기
- 클린코드 작성하기
- TDD 개발

[작업가이드](https://www.notion.so/Go-Programming-Server-Client-engineering-68406fdac7fa4ef9ab1e8974b474c444)
- REST API에서 추구하는 리소스 중심의 메서드를 설계하였다.
- 4개의 리소스, 6개의 메서드를 TDD로 구현하고 서버 구현이 완료된 이후에는 클라이언트로 서버 성능을 테스트 할 것이다.


-----