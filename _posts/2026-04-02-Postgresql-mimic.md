---
title: "PostgreSQL 계정 생성 및 권한 관리 실전 가이드"
date: 2026-04-02
description: PostgreSQL 환경에서 계정 생성, 롤(Role)‧권한 관리, 외부 접속, GUI(DBeaver) 연결의 보안적이고 실용적인 방법을 예시와 함께 제공합니다.
categories: [devops]
tags: [postgresql, user, 권한, 계정, mimic, dbeaver, 데이터베이스, role]
---

## 1. 계정 생성 (유저 이름 규칙)

### 사용자 계정 생성

```sql
-- 계정 생성 예시 (이름 약어 사용)
CREATE USER jhkim WITH PASSWORD 'test';
```
- **계정명**: 영문 이름 약어 + 성 (예: ‘김정현’ → `jhkim`)
- **초기 비밀번호**: `test` (최초 로그인 후 비밀번호 변경 권장)

### 비밀번호 변경

```sql
-- 로그인한 본인 계정에서 비밀번호 변경
ALTER USER jhkim WITH PASSWORD '새비밀번호';
```

---

## 2. 권한(Role)구조 설계 및 적용

### 권한 수준

- **superuser** : 모든 권한 보유
- **master_user** : CRUD 권한 (생성/조회/수정/삭제)
- **readonly_user** : SELECT(조회) 권한만 보유(비밀번호 수정 가능, DDL/DML 불가)
- **스키마**: public만 테이블 생성가능하도록 부여  
  (mimiciv-1.0, mimiciv-2.0, eicu 등에도 동일 원칙 적용)

### 권한 제거

```sql
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA "[스키마이름]" FROM readonly_user, master_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA "[스키마이름]" REVOKE ALL PRIVILEGES ON TABLES FROM readonly_user, master_user;
REVOKE ALL PRIVILEGES ON SCHEMA "[스키마이름]" FROM readonly_user, master_user;

REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM readonly_user, master_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public REVOKE ALL PRIVILEGES ON TABLES FROM readonly_user, master_user;
REVOKE ALL PRIVILEGES ON SCHEMA public FROM readonly_user, master_user;

REVOKE CONNECT ON DATABASE "[DB이름]" FROM readonly_user, master_user;
```

### 기존 롤 삭제 (필요 시)

```sql
DROP ROLE IF EXISTS readonly_user, master_user;
```

### 롤/유저 생성 및 권한 부여

```sql
-- Role 생성
CREATE ROLE readonly_role;
CREATE ROLE master_role;

-- 유저 생성 (이름 예시)
CREATE ROLE jhkim WITH LOGIN PASSWORD 'test';
-- 역할 할당
GRANT readonly_role TO jhkim;

-- DB 접속 권한
GRANT CONNECT ON DATABASE "[DB이름]" TO readonly_role, master_role;

-- 스키마(예: public) 접근 및 권한
GRANT USAGE ON SCHEMA public TO readonly_role, master_role;
GRANT ALL PRIVILEGES ON SCHEMA public TO master_role;

-- mimiciv-1.0, mimiciv-2.0, eicu 스키마 동일 방식 적용

-- 테이블 권한
GRANT SELECT ON ALL TABLES IN SCHEMA "[스키마이름]" TO readonly_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA "[스키마이름]" TO master_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO master_role;

-- 향후 생성 테이블 권한 부여
ALTER DEFAULT PRIVILEGES IN SCHEMA "[스키마이름]" GRANT SELECT ON TABLES TO readonly_role;
ALTER DEFAULT PRIVILEGES IN SCHEMA "[스키마이름]" GRANT ALL PRIVILEGES ON TABLES TO master_role;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON TABLES TO master_role;

-- 특정 권한 수동 부여/회수 예시
GRANT "[권한이름]" TO "[부여대상]";
REVOKE "[권한이름]" FROM "[대상]";
```

> ⚠️ 롤/계정/암호/DB/스키마명은 반드시 사내 정책에 따라 변경하여 사용하세요.  
> 예시 암호, 계정명, IP는 절대 실서비스에 노출하지 말 것.

---

## 3. 외부 접속 허용 및 확인

- `pg_hba.conf` 파일에서 외부 IP 접속 허용 필요 (DB 설정 담당자와 협의)
- 방화벽, 공유기 포트포워딩, PostgreSQL 포트(기본 5432) 허용 설정 필요  
  (실제 IP, 포트는 사내 가이드 및 보안 원칙에 따라 별도 지정)

### CLI 접속 예시 (개인정보/내부정보 미노출)

```sh
psql -h [host] -p [port] -U [user] -d [database]
```
- `[host]`, `[port]`, `[user]`, `[database]`는 개별 생성 정보 사용

---

## 4. DBeaver 접속 설정 예시

- Host: _(내부/외부 IP는 내부문서 별도 참조, 미노출)_
- Port: _(default 5432, 포트포워딩 시 정책에 따른 포트 사용)_
- Database: mimiciv-2.2, mimiciv-1.0, eicu, mimiciv-2.2-ed 등
- Username/Password: 개별 생성 정보에 따라 입력

> **실제 계정, IP, 암호 등 정보가 외부에 유출되지 않도록 반드시 주의하세요.**

---

### DBeaver 및 SQL 권한 부여 예시

```sql
-- 유저 생성 예시
CREATE ROLE "[유저이름]" WITH LOGIN PASSWORD '[유저비밀번호]';

-- DB 접속 권한
GRANT CONNECT ON DATABASE "[데이터베이스명]" TO "[유저이름 또는 롤]";

-- 스키마 접근 권한
GRANT USAGE ON SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";

-- 테이블에 대한 권한 부여
GRANT SELECT ON ALL TABLES IN SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";  -- 필요에 따라 선택

-- 향후 생성테이블 일괄 권한
ALTER DEFAULT PRIVILEGES IN SCHEMA "[스키마명]" GRANT SELECT ON TABLES TO "[유저이름 또는 롤]";
```

---

## 참고 및 보안 안내

- 실제 운영환경에서는 계정/암호/IP/포트 등 민감정보는 절대 외부에 노출하지 마십시오.
- 변경사항 적용 시 기존 세션종료 후 재접속 필요.
- 기타 자세한 DB 보안 및 운영정책은 별도 보안 담당자와 협의하세요.

---