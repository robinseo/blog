---
title: ActiveRecord Tips & Tricks
---

### QueryMethod

**activerecord/lib/active_record/relation/query_methods.rb**

여기 열어보면 대부분의 쿼리 메서드들의 구현체가 있습니다. 구현체는 추상화 되어 있어 따라가기 어렵지만, 메서드 위에 있는 설명과 예시들이 공식 문서보다 훨씬 정리가 잘되어 있어서 ActiveRecord를 쓰다가
뭐가 좀 안되면 이 파일에 들어와서 메서드 이름 검색해보고 사용법 찾아보시면 좋습니다.

### Migration 관련 유용한 팁


**현재 스코어 Migration 확인** 

CLI에서 지금 마이그레이션이 뭐가 어디까지 되어있는지 확인 할 수 있습니다.

```shell
$ rails db:migrate:status


 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20240322121144  Create users
   up     20240322121152  Create contacts

```

Rails로 Database로 만들었다면 Rails가 자동으로 DB에 `schema_migrations` 테이블을 만들어두고 rails db migrateion이 실행 될 때 마다 db/migrate 디렉토리의 migration 파일의 아이디와 자동으로 싱크를 맞춰두고 `schema.rb` 파일도 업데이트 후 마지막 migration 버전을 기록합니다.

TODO : 롤백 어쩌고 좀 정리 

