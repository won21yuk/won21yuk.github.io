---
title: 엘라스틱서치2. Elasticsearch-.py와 Elasticsearch_dsl
categories: [Elastic, Elasticsearch]
---

> Elasticsearch : 7.17.6   \
> Elasticsearch_dsl : 7.4.0

# 1. 개요
django에서 검색엔진을 구현할 때는 Elasticsaerch 모듈을 사용하고, 검색 랭킹 집계를 구현할 때는 Elasticsearch_dsl을 사용했다.

이는 집계할때 dsl이 더 사용하기 좋았기 때문인데, 실제 elasticsearch_dsl 공식 사이트에서도 'Elasticsearch_dsl은 Elasticsearch에 대한 쿼리를 작성하고 실행하는 데 도움이 되는 고급 라이브러리(High-level-library)'라고 설명하고 있다.

반면 공식 elastic사에서 제공하는 elasticsearch-py 라이브러리는 가볍고 간편하게 python 환경에 연결이 가능하게 해주는 이점이 있었다.

그래서 집계가 아닌 단순 검색기능을 구현할 때만 elasticsearch-py를 사용하게 된 것이다.

# 2. 문제점
문제는 django로 랭킹시스템과 검색기능을 동시에 구현하여 웹서비스를 켰을때, 둘중 하나의 기능이 먹통이 되는 현상을 발견하면서 부터였다.

각각 실행시키거나 하나만 기능을 구현한 채로 넣었을 때는 아무런 문제가 발생하지 않았지만, 두 기능을 동시에 적용시켜서 웹을 실행하면 둘 중 하나의 기능이 작동하지 않았다.


# 3. 문제해결
에러는 authorization 관련한 메세지였다. 난 이 에러를 중복된 connection 연결로 인한 권한 충돌이라고 이해했다.

왜냐면, elasticsearh_dsl이 elasticsearch-py 라이브러리를 기반을 만들어 진 것이기 때문에 django의 views에서 두개의 커넥션이 생성 되어버리면, 충돌이 발생하면서 권한에러가 발생한 것일 수 도 있다고 판단한 것이다.

그래서 views.py에 검색기능함수와 집계기능함수를 모두 elasticsearch_dsl으로 통일하기로 결정했고, 이는 즉각적으로 효과를 봤다.

다음에 특정 라이브러리를 기반으로 만들어진 고급 라이브러리를 중복사용해서 에러가 난다면 이 사례를 참고해 볼 수 있을 듯 하다.




