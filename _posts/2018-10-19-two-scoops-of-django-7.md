---
layout: post
    title: "two scoops of django 7장"
    description: "TIL:django"
    date: 2018-10-19
    tags: [TIL]
    comments: true
    share: true목
    
---

## Two scoops of django 7장 - 쿼리와 데이터베이스 레이어음

- 예외처리
    - 단일 객체에서 `get` 대신에  `get_object_or_404()` 이용하기
        - 뷰에서만 이용하자.
    - ObjectDoesNotExist는 어떤 모델 객체에서도 이용할 수 있지만 DoesNotExist는 특정 모델에서만 이용할 수 있음
    - 여러개의 객체가 반환될 때는 ⇒ MultipleObjectsReturned
- 쿼리를 좀 더 명확하게 하기 위해 지연 연산 이용하기!
    - ORM이 지연 연산을 하는 특징을 이용해 ORM메서드와 함수를 원하는 만큼 연결해서 코드를 쓸 수 있음
    - ⇒ 가독성 향상과 유지보수 더 쉽게 해줌
    - 나쁜 코드!

            Promo.objects.active().filter(Q(name__startswith=name)|Q(description_icontains=name))

    - 이렇게 쓰자!

            results = Promo.objects.active()
            results = results.filter(
            						Q(name__startswith=name)|
            						Q(description_icontains=name)
            				) 
            results = results.exclude(status='melted')
            results = results.select_related('flavors')

- 고급 쿼리 도구 이용하기
    - 파이썬을 이용하여 데이터를 가공하기 이전에 장고의 고급 쿼리 도구들을 이용해서 데이터 가공을 시도하자
        - ⇒ 성능이 향상되고(데이터베이스가 데이터 관리와 가공에서 월등히 빠르다) 파이썬 기반 데이터 가공보다 더 잘 테스트되어 나온 코드를 이용하는 장점이 생긴다.  (고급 쿼리 도구는 장고와 대부분의 데이터베이스 사이에서 지속적인 테스트를 실행해 줒ㅁ)
        - 쿼리 표현식
            - 아이스크림 상점을 방문한 모든 고객 중 한 번 방문할 때마다 평균 한 주걱 이상의 아이스크림을 주문한 모든 고객 목록 가져오기
            - 나쁜 코드!

                    for customer in Customer.objects.ierate():
                    		if customer.scoops_ordered > customer.store_visits:
                    				customer.append(customer)

                ⇒  디비안의 모든 고객 레코드에 대해 하나하나 파이썬을 이용, 매우 느리고 메모리 많이 이용

                ⇒ 코드가 얼마나 이용되는지에 상관없이, 코드 자체가 경합 상황에 직면하게됨('READ' 실행중에 'UPDATE'가 처리되는 환경에서라면 데이터 분실이 생길 수 있음)

            - 이렇게 쓰자!

                    customers = Customer.objects.filter(scoop_ordere_gt=F('store_visits')

        - 데이터베이스 함수들
            - UPPER(), LOWER(), COALESCE(), CONCAT(), LENGTH(), SUBSTR() 등
            - 사용하면 생기는 장점
                - 이용이 매우 쉽고 간결
                - 데이터 베이스 함수들은 (비즈니스) 로직을 파이썬 코드에서 데이터베이스로 더 많이 이전할 수 있게 해줌 ⇒ 성능 향상
                - 디비별로 다르게 구현된 디비 함수를 하나로 통합해서 사용할 수 있음
                - 결국 이 함수들은 일반적인 패턴을 가진 쿼리 표현식임 점

        - 필수불가결한 상황이 아니면 로우 SQL은 지향할 것
            - 이식성이 떨어짐
            - 다른 환경의 데이터베이스로 마이그레이션해야 하는 경우 매우 복잡한 문제 발생
        - 로우 SQL을 사용하는 경우
            - 파이썬 코드나 ORM을 통해 생성된 코드보다 훨씬 간결해지고 단축되는 경우
            - 예를들면, 큰 데이터 세트에 적용되는 다수의 쿼리세트가 연동되는 경우
    - 필요에 따라 인덱스 사용하기
    - 트랜잭션
        - 각각의 HTTP요청을 트랜잭션으로 처리하기

                # settings/base.py
                
                DATABASES = {
                	'defaultt: {
                		...
                		'ATOMIC_REQUESTS':True
                	}
                }

            ⇒ 데이터를 포함한 모든 요청이 트랜잭션으로 처리 됨 but 성능 저하

            ⇒ 오직 에러가 발생하고 나서야 디비 상태가 롤백됨 

        - 명시적인 트랜잭션

                @transaction.non_atomic_requests
                def posting_flavor_status(request, pk, status):
                		...
                		with transaction.aomic():
                		# 트랜잭션 안에서 실행됨

            - 개발할 때 더 많은 시간을 요구하는 것이 단점, 성능 문제가 정말 심각하지 않는 한 ATOMIC_REQUESTS 를 사용하는 것이 좋다.
        - 가이드라인
            - 디비 변경이 생기지 않는 디비 작업은 트랜잭션 처리하지 않는다.
            - 디비에 변경이 생기는 작업은 반드시 트랜잭션 처리
            - 디비 읽기 작업을 수반하는 디비 변경 작업 또는 디비 성능에 관련된 특별한 경우는 앞의 두 가이드라인을 모두 고려
            - 데이터 생성 `create(), bulk_create(), get_or_create()` , 수정 `update()` , 삭제 `delete()`  ⇒ 트랜잭션 이용
            - 데이터 가져오기 `get() , fitler(), count(), iterate(), exists(), exclude(), in_bulk`  ⇒ 이용 X
            - 독립적인 ORM 메서드 호출을 트랜잭션 처리하지 말자 ⇒ 장고에서 이미 내부적으로 사용하고 있음

---

- [django transactio문서]([https://docs.djangoproject.com/ko/2.1/topics/db/transactions/#managing-autocommit](https://docs.djangoproject.com/ko/2.1/topics/db/transactions/#managing-autocommit))
