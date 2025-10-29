# BuptleBiz 2.0

가이드 링크 :\
[https://docs.google.com/document/d/1oaFVscJZf2K6Zkv4qH\_7wgTy9YiyT8I7dSvyYzEfgsI/edit?tab=t.0](https://docs.google.com/document/d/1oaFVscJZf2K6Zkv4qH_7wgTy9YiyT8I7dSvyYzEfgsI/edit?tab=t.0)

***

## 0. 사전 작업&#x20;

### 0-1. 깃허브 초대 요청(팀장님 or 파트장님)&#x20;

* 개발 코드 : [https://github.com/Buptle/BuptleBiz](https://github.com/Buptle/BuptleBiz)
* 문서 변환기 : [https://github.com/Buptle/buptle\_doc\_manager.git](https://github.com/Buptle/buptle_doc_manager.git)

### 0-2. 설치 파일 설치&#x20;

* python 3.6.5 로 진행
* java 설치 안하면 환경 변수 설정 X&#x20;
* 설치 파일 링크 : \
  [https://drive.google.com/drive/folders/1uDx598arN8Cs2yuhxr-OVDtfxp22\_Pim](https://drive.google.com/drive/folders/1uDx598arN8Cs2yuhxr-OVDtfxp22_Pim)

<figure><img src="../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

***

## 1. 파이참 연동

### 1-1. github 연동하기&#x20;

1. 파이참 실행&#x20;
2. VCS -> Get From Version Control&#x20;
3. url 에 BuptleBiz 깃헙 url([https://github.com/Buptle/BuptleBiz](https://github.com/Buptle/BuptleBiz)) 입력 후 clone&#x20;
   1. git 로그인 문제 발생 시 git 프로필 pat 설정 후 사용 (설정 방법은 서칭)
   2. git 로그인 프로필 ssl 세팅후 사용 권장&#x20;

### 1-2. 파이썬 인터프리터 설정&#x20;

* file -> settings -> Python -> interpreter -> Add Interpreter (Python 3.6 버전 선택)&#x20;

<figure><img src="../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

* 터미널 창에서(.venv) 나오고 pip list 나오면 성공&#x20;

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

### 1-3. Edit Configurations

* 파이참 우상단 Edit Configurations 클릭 -> + 버튼 클릭 -> Python 선택 -> 정보 입력
  * parameters: `runserver 0:80 --settings=buptlebiz.settings.local`
  * Environment variables: `PYTHONUNBUFFERED=1;DJANGO_SETTINGS_MODULE=buptlebiz.settings.local`
  * 1-2. 에서 추가한 인터프리터 선택&#x20;
* -> Apply -> OK

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

* 환경변수 추가 (환경변수 추가 후 윈도우 재시작 필요)&#x20;

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

### 1-4. 필요 패키지 설치

* 파이참 터미널에서 아래 명령어 순서대로  실행&#x20;
  * `set DJANGO_SETTINGS_MODULE=buptlebiz.settings.local`
  * `pip install -r .requirements\deploy_windows_backup.txt` (뭔가 안되면 CMD로 진행)

> #### 에러 이슈 정리&#x20;
>
> * markupsafe 설치 에러시 1.1.1로 버전 변경
> * ipdb 설치 에러 발생시 setuptools 다운 그레이드 (pip install -U setuptools=="57.0.0")
> * VS 관련 오류로 Jpype1 설치 에러 발생시 아래 링크 다운&#x20;
>   * [https://download.microsoft.com/download/6/A/A/6AA4EDFF-645B-48C5-81CC-ED5963AEAD48/vc\_redist.x64.exe](https://download.microsoft.com/download/6/A/A/6AA4EDFF-645B-48C5-81CC-ED5963AEAD48/vc_redist.x64.exe)
> * 라이브러리 설치중 VS C++ 관련 Build Tools 설치 오류 발생시 오류 로그내 설치 링크를 통해 VS Build Tools 설치 - tool 설치할때 C++도 같이 설치 추가해야함
> * xmlsec 설치 중 절대경로 오류 발생시 xmlsec 먼저 설치 후 다시 pip install -r …. 실행
>   * pip install xmlsec==1.3.13&#x20;

### 1-5. .config\_secret

* 프로젝트 폴더 하위에 .config\_secret 폴더 확인
* .config\_secret\settings\_local.json 파일 수정&#x20;
  * database - default - PASSWORD : postgre 설치 시 설정한 비밀번호로 입력

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

## 2. DB 설정&#x20;

### 2-1. DB 설정&#x20;

* pgadmin 실행&#x20;
* Create -> Database
  * Database 명 : buptle\_biz

### 2-2. 마이그레이션

참고 링크(Swit 권한추가 필요) : [https://app.swit.io/buptle/project/221201074070GllvdnB?task\_detail=221201074570wTYGl9W](https://app.swit.io/buptle/project/221201074070GllvdnB?task_detail=221201074570wTYGl9W)

> #### 마이그레이션 과정의 의미&#x20;
>
> 이 과정은 Django 와 `django-tenant-schemas` 라이브러리를 사용하여 프로젝트의 코드가 변경된 내용(새로운 테이블, 필드 수정 등)을 PostgreSQL 데이터베이스에 실제로 반영하는 것을 의미합니다.
>
> 핵심 명령어는 2가지 입니다.&#x20;
>
> 1. `makemigrations` (변경 사항 감지):
>    * Django에게 "현재 프로젝트의 모든 모델(Python 코드)을 확인하고, 데이터베이스에 적용된 상태와 비교해서 달라진 점을 마이그레이션 파일로 만들어라"고 지시합니다.
>    * **이 단계는 실제 DB를 건드리지 않고, 단지 `.py` 형태의 설계도(마이그레이션 파일)만 만듭니다.**
> 2. `migrate_schemas` (DB에 적용):
>    * `makemigrations`로 생성된 설계도(`0001_initial.py`, `0002_...py` 등)를 읽어와 데이터베이스에 실제로 SQL 쿼리를 실행하여 테이블을 생성하거나 수정합니다.
>    * `django-tenant-schemas`를 사용하기 때문에 일반 `migrate` 대신 `migrate_schemas`를 사용하여 **기본 스키마(`public`)**&#xC5D0; 테이블들을 생성합니다.

1. 환경 설정&#x20;

* Django 프로젝트가 사용할 설정 파일(`buptlebiz.settings.local`)을 환경 변수로 지정합니다.

```bash
set DJANGO_SETTINGS_MODULE=buptlebiz.settings.local
```

2. 마이그레이션 파일 생성 (`makemigrations`)

* 모델 변경 사항이 있는 앱들의 목록을 명시적으로 지정하여 마이그레이션 파일을 생성합니다.
* 성공 확인: 터미널에 `No changes detected in app 'app_name'` 또는 `Migrations for 'app_name':` 메시지가 출력되는지 확인하세요.

```bash
python manage.py makemigrations main manage contract workflows companies security_grades counsel letter doc_issue lawsuit guide_doc widgets api doc_received epic claim batch component buptle_calendar document_tools digital_certificate --settings=buptlebiz.settings.local
```

3. 데이터베이스 스키마 적용 (`migrate_schemas`)

* 생성된 마이그레이션 파일(설계도)을 읽어와 PostgreSQL 데이터베이스의 `public` 스키마에 모든 테이블을 생성하고 업데이트합니다.
* 성공 확인: 터미널에 `Applying app_name.00XX_initial... OK`와 같은 메시지가 프로젝트의 모든 앱에 대해 오류 없이 출력되는지 확인하세요. 이 단계가 성공해야 `companies_bcompany` 테이블이 생성되어 다음 단계를 진행할 수 있습니다.

```bash
python manage.py migrate_schemas --settings=buptlebiz.settings.local
```

### 2-3. Tenant 설정 (Public Tenant 데이터 등록)

1. 파이썬 쉘 실행 및 Django 환경 초기화

```python
# 파이썬 실행
python

# Django 환경 세팅
import django
django.setup()
```

2. Public Tenant 데이터 생성 및 저장

```bash
from buptlebiz.companies.models import BCompany

tenant = BCompany(domain_url='buptlelocal.com',
                  schema_name='public',
                  name='public',
                  biz_regnum="")

# 데이터베이스에 로우(Row)를 삽입합니다.
tenant.save()
```

3. 파이썬 쉘 종료

```bash
exit()
```

### 2-4. superuser 생성

> Superuser는 Django 애플리케이션의 최고 관리자 권한을 갖는 계정입니다. 이 계정을 생성해야 웹에서 `http://buptlelocal.com/django_admin/` 주소로 접속하여 프로젝트의 데이터 및 설정을 관리할 수 있습니다.
>
> 멀티 테넌시 환경에서는 관리자 계정이 어떤 테넌트 스키마에 속할지 지정해야 하므로, 여기서는 기본 관리 영역인 `public` 스키마에 계정을 생성합니다.

1. 파이참 터미널에서 명령어 실행&#x20;

```bash
python manage.py createsuperuser --settings=buptlebiz.settings.local
```

2. 질문 응답&#x20;
   1. Enter Tenant Schema ('?' to list schemas): public
   2. Email address: [dev@buptle.com](mailto:common@buptle.com)
   3. Password: asd1234!

## 3. Django 어드민 설정&#x20;

### 3-1. hosts 파일 변경&#x20;

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
#192.168.56.10 buptlelocal.com
#192.168.56.10 test.buptlelocal.com
127.0.0.1 test.buptlelocal.com
127.0.0.1 buptlelocal.com
#192.168.56.10 daekyo.buptlelocal.com
192.168.0.11 d-local.buptlelocal.com
211.217.152.250  sgw.coocon.co.kr

```

### 3-2. 장고 어드민 설정&#x20;

1. 파이참 서버 런&#x20;

```bash
python manage.py runserver
```

2. [http://buptlelocal.com:8000/django\_admin/login/?next=/django\_admin/](http://buptlelocal.com:8000/django_admin/login/?next=/django_admin/)
   1. 아이디: dev@buptle.com
   2. 비번: asd1234!
3. COMPANIES 에서 B companys 추가&#x20;

![](<../../../.gitbook/assets/unknown (1).png>)

4. COMPANIES 에서 B sys configs 추가&#x20;

![](<../../../.gitbook/assets/unknown (2).png>)

![](<../../../.gitbook/assets/unknown (3).png>)

![](<../../../.gitbook/assets/unknown (4).png>)

* MODULES 안에 Config value 는 아래 참고 (바뀔 수 있으니 최신 설정을 개발팀에게 요청)

```
CUSTOM_DASHBOARD, COUNSEL_MANAGE,
ALARM_SEND_EMAIL_ALL_USER,

ENABLE_PDF_UPLOAD,
TMP_DASH,

ENABLE_HWP_UPLOAD,
TEMPLATE_ENABLE_UPLOAD_PDF,
DISABLED_PAPER_PRINT_BTN,
HIDE_SEND_BTN_BEFORE_SIGN,
ENABLED_FINAL_REVIEW,
DOA,
ENABLE_HWP_UPLOAD,
APP_LAWSUIT,
DIVIDE_DAEKYO_VERSION,
TEMPORARY_SIGN_TYPE_DESOLVE,
SECURITY_GRADE_ENABLED,
NEW_DASHBOARD,

OUTER_SIGN_BUTTON_ADDED,
NEW_PGROGRESS_PROCESS,
CUSTOM_DASHBOARD,
CONTRACT_STATUS_SEPARATION_TYPE_A,

STANDARD_TYPE,
LANGUAGE_TYPE,
IS_NEW_CONTRACT,
DOC_RECEIVED,
SHOW_EMAIL_COLLECT,
CONTRACT_ALARM_DATE_AUTO_CALCULATE,
ENABLED_PDF_CONVERT_ON_PAPER,
PDF_WT_MARK_CONFIRM,


SHOW_EMAIL_COLLECT,
COMPARE_DRAFTABLE,
RELATED_CASE_ENABLED,
MODULE_DOC_RECEIVED,

CAN_CHANGE_HISTORYDATA_BORDER_COLOR,
COMMON_TIMELINE_VIEW,
RECORD_TEMPLATE_VERSION,
REQUIRED_FIELD,

EPIC_MANAGE,
DOWNLOAD_FILE_NAME_TYPE_1,
PROCESS_ALL_STOP,
MULTI_RECEIVER_CONTRACT_IN_DOCX_CONTRACT,


ENABLED_CONTRACT_RELATION_USER_MULTI_ADD,
ONLY_ADMIN_HANDLE_TAG,

HIDE_SEND_BTN,
REVIEWER_ASSIGN_CONFIG_TREE_VIEW_ACTIVATE,
BACK_TO_LAWYER_CONFIRM,
SHOW_OPPONENT_INFO,

NEW_DOWNLOAD_LOGIC,

PDF_WT_MARK_RCV_CONT,
PDF_WT_MARK_DIRECT_CONFIRM,
WATERMARK_ON_RECEIVE_CONTRACT,
CONVERT_PAPER_TO_ONLINE,

OPPONENT_SIGN_USER,

SEAL_STAMP,

 ,CON_MULTI_DOC_ENABLED,
REQUEST_CREATE_APPROVAL
```

### 3-3. 관리자 아이디 설정&#x20;

1. [http://test.buptlelocal.com:8000/makefirsttemplate/](http://test.buptlelocal.com:8000/makefirsttemplate/)
   1. 법틀 비밀번호: 80768076
   2. 관리자 이름: 관리자
   3. 관리자 이메일: dev@buptle.com
   4. 관리자 비번: asd1234!

### 3-4. 암호화 key 발급&#x20;

1. [http://test.buptlelocal.com:8000/dev\_dashboard/init/](http://test.buptlelocal.com:8000/dev_dashboard/init/)

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

2. 장고 어드민 사이트 등록 확인&#x20;

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>
