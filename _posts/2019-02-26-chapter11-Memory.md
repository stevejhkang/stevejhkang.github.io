---
layout: posts
title: chapter11. Performance Tuning - Memory
date: 2019-02-26 15:37:36 +0900
type: posts
published: true
comments: true
categories: [se]
tags: [System, Server]
---


7. 메모리 성능과 용량 측정
    * 메모리는 가장 작은 유닉스 유틸리티를 실행하는 경우에서부터 아주 큰 크기의 인스턴스를 갖는 Oracle 데이터베이스 실행까지 모든 프로그램을 실행하는데 필수 불가결한 요소이다. 메모리 고갈은 시스템이 스왑을 사용하게 만들어 성능에 막대한 지장을 초래하게 된다. 따라서 메모리의 성능과 사용량을 모니터하는 것이 중요하다.
    1. 가상 메모리 구현
        * 가상 메모리는 프로세스 간에 코드와 데이터를 투명하게 공유할 수 있도록 해주며 시스템에 물리적으로 장착된 메모리보다 더 많은 메모리를 할당할 수 있게 해주고 심지어는 캐싱기능을 제공함으로써 파일 시스템의 성능에 있어 중요한 역할을 하게 된다. 가상메모리의 구현 형태는 페이징, 스왑공간, 스왑핑을 써서 적용한다.
        1. 스왑공간: 스왑공간은 일반적으로 데이터의 페이징과 스왑핑에 의해 생성된 메모리를 일시적으로 저장하기 위해 마련된 디스크 영역이다.
        2. 페이징: 페이징은 데이터를 디스크에서 메모리 그리고 메모리에서 디스크 양방향으로 전송하는 프로세스를 말한다.
        3. 스왑핑: 페이징에 메모리에 있는 프로세스의 유휴부분을 디스크로 이동하는데 반해서, 스왑핑은 실제 메모리가 부족할 때 프로세스 전체를 디스크로 이동하게 된다. 스왑핑을 주로 메모리가 적은 상황에서 일어나게 된다.
    2. 스왑 통계 출력: `swapon` 명령어를 쓰면 스왑 공간의 통계를 보여준다.
    3. vmstat를 이용한 메모리 사용량 모니터: vmstat은 가상 메모리 상황을 상세하게 보고해준다. 
        1. swap: 이용가능 스왑공간(KB)
        2. free: 이용가능 실제메모리(KB)
        3. pi: 페이지 인 연산
        4. po:페이지 아웃 연산
        5. sr: 스캔 비율 - 페이지 아웃 할 수 있는 페이지를 검색하고 있는 1초당 페이지 수 스캔비율이 높으면 더 많은 메모리를 요구하는 것임.
        * 페이지 인 연산은 데이터가 메모리로 읽혀지고 있고 또 파일 시스템의 활동을 나타내기도 한다. free list는 물리적으로 예약된 소량의 메모리인데 freelist가 하한선에 도달하면 운영체제는 유휴 페이지를 검색하고 그것을 페이지 아웃함으로써 새 프로세스가 항상 메모리를 이용할 수 있게 한다. 하지만 과도한 페이지 아웃은 시스템 성능에 큰 영향을 주게 된다.
    4. sar를 이용한 페이징 활동 모니터: `sar -p`는 페이지 인에 관련한 활동을 보고해주며, sar -g는 페이지 아웃에 관련한 활동을 보고해준다.
    5. 프로세스의 메모리 사용 모니터링 툴
        * 전체 시스템의 메모리 사용량 측정을 마친 후에는 프로세스 별 활용도를 조사해야 할 것이다. 이런 정보를 수집하기 위해 top, ps, pmap가 있다.
        1. top를 이용한 메모리 사용량 모니터: 상주 메모리(RES) 크기 순으로 프로세스 보여주고 실제 메모리(RAM)와 스왑활용도도 보여준다.
        2. ps를 이용한 메모리 사용량 모니터:
        3. pmap를 이용한 메모리 사용량 모니터
8. 메모리 성능과 스왑 성능 튜닝
    * 유닉스 운영체제의 가상 메모리 서브 시스템은 물리적 메모리와 디스크 둘 모두에게 의존하고 있다. VM서브시스템의 성능은 디스크에 있는 스왑 파티션의 최적화에 의존한다. 메모리와 스왑 성능은 대부분 최적화된 커널 코드에 밀접하다. 그러나 메모리와 스왑 성능을 최적화할 수 있도록 관리자가 할 수 있는 일은 거의 없다. 이런 튜닝 측정은 운영체제에 특정적인 다양한 설정, 스왑 파티션의 위치 최적화, 메모리의 이용을 최소화할 수 있는 프로그래밍 기법의 이용을 포함한다.
    1. 솔라리스의 우선순위 기반 페이징 이용
    2. 스왑 파티션으로의 접근 최적화: 스왑 파티션의 위치 최적화와 다중 스왑 파티션의 이용이 메모리와 스왑 성능을 튜닝할 수 있는 또다른 효과적인 방법이다. 커널이 낮은 지연대기와 가능한 높은 효율을 가지고 스왑으로 읽고 쓸 수 있을 때 페이징과 스왑핑이 가장 잘 수 행된다. 스왑 파티션을 디스크의 바깥 쪽 트랙으로 패치함으로써 가능하다. 보통 슬라이스 1이 된다. 다중 스왑 파티션은 두개 또는 그 이상의 물리적 장치에서 스왑 활동이 일어나게 해주며, 그 결과 효율을 증가시키게 된다.
    3. 공유 라이브러리의 이용: 공유 라이브러리는 많은 애플리케이션에 공통적으로 사용되는 코드를 말하며, 그 코드를 사용하는 실행 프로그램 각각이 동일한 내용을 중복하여 가지는 대신에 그 코드를 하나의 파일로 두고 공유함으로써 디스크 공간을 절약하게 된다. 또한 공유 라이브러리의 코드 세그먼트는 메모리의 프로세스들을 공유하게 된다. 즉 공유 라이브러리의 코드가 메모리로 단 한번만 로드되면 다른 프로그램을 추가적으로 로드하지 않고도 그 라이브라리에 링크시킴으로써 메모리 절약을 할 수 있다.
9. 메모리 용량과 스왑 용량 계획
    1. 프로세스의 메모리 사용 모니터: 애플리케이션의 메모리 사용량을 모니터하는데 top, ps, pmap 명령어를 사용한다. 모니터링시 비정상적인 데이터가 확인되면 즉시 그 문제를 분간할 수 있어야 한다. 
    2. 필요한 양보다 더 여유 있게 스왑 할당: 시스템에 메모리를 추가할 때 대부분의 관리자가 그에 맞춰 스왑공간을 조정하는 것을 잊곤 하는데, oracle과 같이 스왑 공간을 요구하는 애플리케이션에 심각한 문제가 될 수 있다. 새 메모리에 맞출 수 있는 가장 쉬운 방법이 운영체제를 설정할 때 필요한 것보다 더 많이 스왑을 할당하는 것이다. 이런 완충의 방법으로 메모리 업그레이드 시 한숨 돌릴 수 있는 여유가 생기게 된다.
