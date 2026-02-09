---
layout: post
title: "[Web Security] Nginx & Apache 필수 보안 설정: 상세 버전 및 파일 목록 숨기기"
date: 2026-02-09 10:00:00 +0900
categories: [Security, Web]
tags: [Nginx, Apache, Security, WebServer]
---

# 우리 서버의 'TMI'를 막아라: Nginx & Apache 필수 보안 설정

개발자로 일하며 다양한 프로젝트를 관리하다 보면, 코드를 잘 짜는 것만큼이나 **서버가 내뱉는 불필요한 정보를 단속하는 것**이 중요합니다. 해커는 아주 작은 정보(서버 버전, 파일 목록)만으로도 공격의 실마리를 찾기 때문이죠. 오늘은 가장 기초적이면서도 강력한 방어선인 **Directory Listing 차단**과 **Server Tokens 숨기기**를 정리합니다.

---

## 1. Directory Listing (파일 목록 노출) 차단

**"현관문에 우리 집 가구 배치도를 붙여두지 마세요"**

* **현상**: 특정 폴더에 접근했을 때 `index.html` 파일이 없으면, 서버가 해당 폴더 내의 모든 파일 목록을 브라우저에 리스트로 보여주는 현상입니다.
* **위험성**: 백업 파일(`.zip`, `.bak`), 설정 파일, 혹은 개발 과정에서 남은 민감한 로그가 그대로 노출되어 정보 유출의 시발점이 됩니다.

### 🛠️ 설정 방법

| 서버 종류 | 설정 파일 경로 | 핵심 설정 코드 |
| :--- | :--- | :--- |
| **Nginx** | `/etc/nginx/sites-available/default` | `autoindex off;` |
| **Apache** | `/etc/apache2/apache2.conf` | `Options -Indexes` |

> **Nginx 설정 예시:**
> ```nginx
> location / {
>     autoindex off; # 목록 노출 기능을 명시적으로 끕니다.
> }
> ```

---

## 2. Server Tokens (서버 버전 정보) 은닉

**"도둑에게 우리 집 도어락 모델명을 알려주지 마세요"**

* **현상**: HTTP 응답 헤더(`Server`)에 서버 이름뿐만 아니라 `Nginx/1.18.0 (Ubuntu)` 처럼 **상세 버전**까지 포함되어 전송되는 상태입니다.
* **위험성**: 특정 버전에만 존재하는 알려진 취약점(CVE)이 있을 경우, 공격자는 버전을 확인하는 즉시 해당 취약점을 찌르는 자동화 공격 도구를 실행할 수 있습니다.

### 🛠️ 설정 방법

| 서버 종류 | 설정 파일 경로 | 핵심 설정 코드 |
| :--- | :--- | :--- |
| **Nginx** | `/etc/nginx/nginx.conf` | `server_tokens off;` |
| **Apache** | `/etc/apache2/conf-available/security.conf` | `ServerTokens Prod` |

> **Apache 설정 예시:**
> ```apache
> # 상세 버전을 숨기고, 에러 페이지 하단의 서버 정보(Signature)도 삭제합니다.
> ServerTokens Prod
> ServerSignature Off
> ```

---

## 📊 최종 요약 및 체크리스트

1. **Directory Listing**: 파일 구조 노출 차단 → `403 Forbidden` 응답 확인
2. **Server Tokens**: 상세 버전 은닉 → `Server: nginx` 또는 `Server: Apache`만 노출 확인

---

## 💡 마무리하며

정보보안기사나 ISMS-P 관점에서도 이러한 기초 설정은 **'정보시스템 접근 통제'**의 기본 중의 기본입니다. 설정 한 줄을 추가하는 것만으로도 공격자가 우리 서버를 보고 **"여기는 기본기가 탄탄해서 뚫기 까다롭겠다"**라고 느끼게 만드는 심리적 방어 효과를 얻을 수 있습니다.