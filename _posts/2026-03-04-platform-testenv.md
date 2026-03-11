---
title: platform 서버 - 테스트 환경 구축
author: cgoing4dsk
date: 2026-03-04 15:00:00 +0900
categories: [3DEXPERIENCE platform, Server]
tags: [server, test, backup, restore]
---

참고 : R2025x FD03 환경에서 기존에 구축된 데이터를 활용하여 신규 테스트 환경을 신속하게 구성하는 가이드입니다. 신규 설치(3~5일 소요) 대비 **약 1일 내외**로 작업을 완료할 수 있습니다.

---

## 1. Backup (기존 환경)

### 1-1. 서비스 종료

- 실행 중인 모든 **3DEXPERIENCE platform 관련 서비스**를 중지합니다.

### 1-2. 폴더 백업 및 압축

다음 주요 폴더들을 압축하여 백업합니다.

- **Apache:** `C:\Apache24`
    
- **Install Folder:** `C:\DassaultSystemes\R2025x`
    
- **TomEE Folder:** `C:\DassaultSystemes\R2025x\TomEE`
    
- **File Folder:** `C:\DassaultSystemes` (위의 설치/TomEE 폴더를 제외한 나머지 데이터 폴더)
    

### 1-3. DB Export

CMD(명령 프롬프트)에서 아래 명령어를 순차적으로 수행하여 덤프 파일을 생성합니다.


```CMD
expdp system/orcl schemas=X3DPASS_ADMIN dumpfile=X3DPASS_ADMIN.dmp logfile=X3DPASS_ADMIN.log
expdp system/orcl schemas=X3DPASS_TOKEN dumpfile=X3DPASS_TOKEN.dmp logfile=X3DPASS_TOKEN.log
expdp system/orcl schemas=X3DDASH_ADMIN dumpfile=X3DDASH_ADMIN.dmp logfile=X3DDASH_ADMIN.log
expdp system/orcl schemas=X3DNOTIF_ADMIN dumpfile=X3DNOTIF_ADMIN.dmp logfile=X3DNOTIF_ADMIN.log
expdp system/orcl schemas=X3DCOMMENT_ADMIN dumpfile=X3DCOMMENT_ADMIN.dmp logfile=X3DCOMMENT_ADMIN.log
expdp system/orcl schemas=X3DSWYM_ADMIN dumpfile=X3DSWYM_ADMIN.dmp logfile=X3DSWYM_ADMIN.log
expdp system/orcl schemas=X3DSWYM_MEDIA dumpfile=X3DSWYM_MEDIA.dmp logfile=X3DSWYM_MEDIA.log
expdp system/orcl schemas=X3DSWYM_WIDGET dumpfile=X3DSWYM_WIDGET.dmp logfile=X3DSWYM_WIDGET.log
expdp system/orcl schemas=X3DSPACE dumpfile=X3DSPACE.dmp logfile=X3DSPACE.log
expdp system/orcl schemas=FIPERACS dumpfile=FIPERACS.dmp logfile=FIPERACS.log
```

> [!TIP]
> 
> **덤프 파일 위치:** 생성된 파일들은 기본적으로 `C:\oracle\admin\orcl\dpdump` 경로에 저장됩니다. 해당 폴더를 압축하여 보관하세요.

---

## 2. Restore (신규 환경)

### 2-1. 사전 준비

- 신규 서버에 설치 가이드를 참조하여 **Oracle Database** 설치까지 완료합니다.
    

### 2-2. 데이터 복원

- 백업한 압축 파일들을 해제하고, **기존 환경과 동일한 경로**(`C:\...`)로 이동시킵니다.
    

### 2-3. Apache 서비스 등록

- 복사된 Apache 폴더를 기반으로 Windows 서비스를 등록합니다.
    

### 2-4. DB Import

백업된 덤프 파일들을 신규 환경의 Oracle Default 폴더로 이동시킨 후, CMD에서 아래 명령어를 수행합니다.


```batch
# EXCLUDE=USER 옵션을 사용하여 기존 유저 정보를 유지하며 데이터를 덮어씁니다.
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DPASS_ADMIN dumpfile=X3DPASS_ADMIN.dmp logfile=X3DPASS_ADMIN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DPASS_TOKEN dumpfile=X3DPASS_TOKEN.dmp logfile=X3DPASS_TOKEN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DDASH_ADMIN dumpfile=X3DDASH_ADMIN.dmp logfile=X3DDASH_ADMIN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DSPACE dumpfile=X3DSPACE.dmp logfile=X3DSPACE.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DNOTIF_ADMIN dumpfile=X3DNOTIF_ADMIN.dmp logfile=X3DNOTIF_ADMIN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DCOMMENT_ADMIN dumpfile=X3DCOMMENT_ADMIN.dmp logfile=X3DCOMMENT_ADMIN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DSWYM_ADMIN dumpfile=X3DSWYM_ADMIN.dmp logfile=X3DSWYM_ADMIN.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DSWYM_MEDIA dumpfile=X3DSWYM_MEDIA.dmp logfile=X3DSWYM_MEDIA.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=X3DSWYM_WIDGET dumpfile=X3DSWYM_WIDGET.dmp logfile=X3DSWYM_WIDGET.log
impdp system/orcl EXCLUDE=USER table_exists_action=replace schemas=FIPERACS dumpfile=FIPERACS.dmp logfile=FIPERACS.log
```

### 2-5. 서비스 재구성 및 등록

1. **3DNotification, 3DSwYm:** 각 서비스의 `reconfig`를 실행합니다.
    
    - _주의:_ 3DSwYm의 경우 Index 관련 에러가 발생할 수 있으나 무시하고 진행하십시오.
        
    - 완료 시 `3DNotification Node`, `3DSwym ExternalConverterSvc` 서비스가 생성됩니다.
        
2. **Index 서비스 등록:** 아래 명령어를 통해 수동으로 서비스를 등록합니다.

```batch
:: 3DSwym Index 서비스 등록
sc create "Exalead CloudView - R2025x_3DSwym" binPath="C:\DassaultSystemes\R2025x\3DSwym\win_b64\datadir\bin\..\bin\cvd.exe" start="demand"

:: 3DSpace Index 서비스 등록
sc create "3DEXPERIENCE R2025x 3DSpace Index" binPath="C:\DassaultSystemes\R2025x\3DSpaceIndex\win_b64\cv\data\bin\..\bin\cvd.exe" start="demand"
```

### 2-6. 최종 확인

- 각 서비스를 순차적으로 실행한 후, **3DEXPERIENCE platform**에 접속하여 정상 작동 여부를 테스트합니다.
    
