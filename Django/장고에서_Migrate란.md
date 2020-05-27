# 장고에서 Migrate란?

이런 근본적인 문제를 해결하기위해서는 migrate가 뭔가에 대해서 이해를 하는 것이 중요할 것 같다.

1. 마이그레이션 파일 (초안) 생성하기 : `makemigrations`
2. 해당 마이그레이션 파일을 DB에 반영하기 : `migrate`

예상보다는 조금 심플한 모습을 가지고 있다. 



쉽게 이야기하면 장고에서 model을 만들면 -> `makemigrations`를 통해서 일단 SQL문으로 table화를  어떤 필드를가지고 table을 만들지 혹은 어떤 필드가 변경 됬는지 삭제 되었는지를 저장해놓는 파일이라고 보면 된다. 

-> `migrate`의 경우 이 migrations를 통해 만들어진 파일을 바탕으로 DB에 직접적으로 SQL문으로 자동으로 변환해주는 시스템이다.



1번 같은 위의 같은 문제 (no such table, column 등의 오류) 로 실제 DB를 날리거나 할 것이 아니면 여러가지 방법을 강구해야 하는데, 찾아보는 블로그에 의하면 migration오류라고 했다. 



이런 오류들을 확인하는 방법은 결국 DB에서 직접 확인하던가 혹은 migration 확인하는 방법으로 처리하는게 제일 좋아보였다. (오늘 해결할때 가장 원인을 잘 찾았던 것도 결국에는 `showmigrations` 로 어디까지 모델링 되어있는지 확인하는 방식으로 처리했었다.)



참고: https://wayhome25.github.io/django/2017/03/20/django-ep6-migrations/