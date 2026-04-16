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


---

## 3. 데이터 디렉토리 위치 변경(마이그레이션)

데이터베이스 실제 파일 경로를 별도의 디렉터리(예: `/home/db/main`)로 이동하는 과정입니다.

###  데이터 이동

```bash
sudo systemctl stop postgresql
sudo mv /var/lib/postgresql/18/main /home/db/main
```

### 권한 설정 (핵심)

```bash
sudo chown -R postgres:postgres /home/db/main
sudo chmod 700 /home/db/main
sudo chmod 750 /home/db
```

### 설정 변경

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

- 파일 내에서 아래 항목을 수정 또는 추가:

```
data_directory = '/home/db/main'
```

> 변경 후, PostgreSQL을 재시작하여 정상작동하는지 반드시 확인하세요!
> 
> ```bash
> sudo systemctl start postgresql
> sudo systemctl status postgresql
> ```
> 
> 
> 에러 발생 시 로그(`/var/log/postgresql/postgresql-18-main.log`) 확인

---



### 권한 자동화 부여 예시

```sql
-- 1. readonly_role, master_role 생성
CREATE ROLE readonly_role;
CREATE ROLE master;

-- (유저 생성 및 역할 할당은 상황에 맞게 별도 수행. 예시 생략)

-- 2. DB 접속 권한 부여
GRANT CONNECT ON DATABASE "DB_name" TO readonly_role, master;

-- 3. 모든 스키마에 master/readonly_role 권한 자동 부여

DO $$
DECLARE
    sch TEXT;
BEGIN
    FOR sch IN SELECT nspname FROM pg_namespace WHERE nspname NOT LIKE 'pg_%' AND nspname != 'information_schema'
    LOOP
        -- master: 모든 스키마 권한
        EXECUTE format('GRANT ALL ON SCHEMA %I TO master', sch);

        -- readonly_role: 스키마 접근만
        EXECUTE format('GRANT USAGE ON SCHEMA %I TO readonly_role', sch);
    END LOOP;
END
$$;

-- 4. 모든 스키마 내 테이블/시퀀스 자동 권한 부여
DO $$
DECLARE
    sch TEXT;
BEGIN
    FOR sch IN SELECT nspname FROM pg_namespace WHERE nspname NOT LIKE 'pg_%' AND nspname != 'information_schema'
    LOOP
        -- master: 모든 테이블/시퀀스에 모든 권한
        EXECUTE format('GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA %I TO master', sch);
        EXECUTE format('GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA %I TO master', sch);

        -- readonly_role: SELECT만
        EXECUTE format('GRANT SELECT ON ALL TABLES IN SCHEMA %I TO readonly_role', sch);
        EXECUTE format('GRANT SELECT ON ALL SEQUENCES IN SCHEMA %I TO readonly_role', sch);
    END LOOP;
END
$$;

-- 5. 향후 생성되는 테이블 권한 (public 예시, 필요시 반복 적용)
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO master, readonly_role;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO master;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly_role;
```

> ⚠️ 롤/계정/암호/DB/스키마명 등은 반드시 사내 정책에 맞게 사용하세요.  
> 예시 암호, 계정명, DB명, IP 등은 실서비스에 노출 금지.

---

## 3. 외부 접속 허용 및 확인

- `pg_hba.conf` 파일에서 외부 IP 접속 허용 필요 (DB 설정 담당자와 협의 필요)
- 방화벽, 공유기 포트포워딩, PostgreSQL 포트(기본 5432) 허용 설정 필요  
  (IP, 포트번호 등은 사내 가이드 및 보안 정책 따름)

### CLI 접속 예시 (개인정보/내부정보 미노출)

```sh
psql -h [host] -p [port] -U [user] -d [database]
```
- `[host]`, `[port]`, `[user]`, `[database]`는 개인별로 지정한 정보 사용

---

## 4. DBeaver 접속 설정 예시

- Host: _(내부/외부 IP 등은 별도 내부문서 참고)_
- Port: _(기본 5432, 포트포워딩 등 정책에 따라 별도 지정)_
- Database: mimiciv-2.2, mimiciv-1.0, eicu 등
- Username/Password: 개별 생성 정보 입력

> **실제 계정, IP, 패스워드 등 민감정보 유출 주의**

---

### DBeaver 및 SQL 권한 부여 예시

```sql
-- 유저 생성 예시
CREATE ROLE "[유저이름]" WITH LOGIN PASSWORD '[유저비밀번호]';

-- DB 접속 권한
GRANT CONNECT ON DATABASE "[데이터베이스명]" TO "[유저이름 또는 롤]";

-- 스키마 접근 권한
GRANT USAGE ON SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";

-- 테이블 권한 (필요에 따라 선택)
GRANT SELECT ON ALL TABLES IN SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA "[스키마명]" TO "[유저이름 또는 롤]";

-- 향후 생성 테이블 권한 자동 부여
ALTER DEFAULT PRIVILEGES IN SCHEMA "[스키마명]" GRANT SELECT ON TABLES TO "[유저이름 또는 롤]";
```

---

## 참고 및 보안 안내

- 실제 운영환경에서는 계정/암호/IP/포트 등 민감정보는 절대 외부에 노출하지 마십시오.
- 변경사항 적용 시 기존 세션종료 후 재접속 필요.
- 기타 자세한 DB 보안 및 운영정책은 별도 보안 담당자와 협의하세요.

---