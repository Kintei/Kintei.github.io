---
layout: single
title:  "[모의해킹] 03. HTTP 프로토콜"
date: 2026-05-11
categories: [security study]
tags: [network, Authentication, Authorization]
---

## 인증(Authentication)

- 사용자의 신원을 검증하는 행위 ( 비밀번호, OTP, 지문 인식 등 )
- 서버가 사용자의 신원을 확인 하기 위해 필요한 절차

### 인증의 주요 유형

1. 지식 기반 : 사용자가 알고 있는 정보
	- 예 : Password, PIN, 보안 질문
2. 소유 기반 : 사용자가 소유한 기기 / 물건
	- OTP, SMS 인증번호, 스마트 카드, 보안 토큰
3. 생체 기반 : 신체적 특징
	- 지문, 홍채, 얼굴 인식, 목소리
4. 행위 기반 : 사용자의 고유 행동 패턴
	- 서명, 걸음 걸이


### 인증 방식 분류

1. SFA(Single-Factor-Authentication) : 단일 요소 인증
	- ID / PW 하나만 사용하는 방식
2. 2FA(Two-Factor-Authentication) : 2가지 서로 다른 인증 요소를 결합
	- 비밀번호 + OTP, 비밀번호 PIN 등 2가지의 인증 방식을 사용
3. MFA(Multi-Factor-Authentication) : 2개 이상의 서로 다은 요소를 조합
	- 비밀번호 + OTP + 지문 인증 등 2가지 이상의 인증 요소를 사용하는 경우
	- 일반적으로는 2FA도 MFA 포함시키지만 좁은 의미로는 3개 이상을 의미할때 사용
4. API Key 인증 : API 요청시 고유 키를 헤더에 포함하는 방식
5. 공동인증서 : 은행 인증서 등

### 주요 인증 기술
1. FIDO (Fast IDentity Online) : 생체 인증 등을 활용한 국제 표준 인증 프레임워크
	- 국제 인증 표준 중 하나
	- 비밀번호를 대체하는 UAF와 U2F로 나누어진다.
	- 대표적인 경우로는 삼성 페이의 생체 인증을 둘 수 있다.
	1. UAF(Universal Authentication Factor) : 지문, 얼굴 등 사용자 생체 정보를 인증하는 모바일 중심의 인증 방식
	2. CTAP1(U2F, Universal 2nd Factor) : 2단계 인증 방식.
		일단 ID와 비밀번호를 통해 로그인 후, FIDO 인증시험을 통과한 보안 키를 USB, 저전력 블루투스, NFC 등을 통해 기기와 연결해 PIN을 입력하는 등 추가 인증을 하는 방식이다.
		물리적인 보안 키를 이용하여 비밀번호가 탈취되어도 인증이 되지 않게 할 수 있습니다.
		현대에는 암호화폐 보관을 위한 방법으로 주로 사용됩니다.
	3. CTAP2
		FIDO 보안 키, 생체 인식 장치 등을 통해 휍 및 운영체제에서 비밀번호 없이 인증을 하는데 사용된다.
		U2F와 유사하지만, 비밀번호가 없고 이를 생체 인증으로 대체할 수 있다.
		삼성 페이의 생체 인증 방식이 이에 해당 한다.
2. SSO (Single Sign-On) : 한 번의 로그인으로 여러 시스템을 접근하는 방식.
3. OAuth 2.0 / OpenID Connect : 소셜 로그인(구글, 카카오 등)에 사용되는 표준 인증/인가 프로토콜


## 인가(Authorization)