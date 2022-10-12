---
title: Hadoop Deep Inside - Ch.2 구글 파일 시스템(Google File System) (2)
categories: [Hadoop, Deep Inside]
---

# 3. GFS의 상호작용(Interactions)

GFS의 설계자들은 모든 작업에 대해 마스터의 참여가 최소화되로록 시스템을 설계했습니다.
그래서 이번 챕터에서는 클라이언트, 마스터, 그리고 청크서버 간의 상호작용에서 어떻게 마스터의 개입은 최소화되며, 이 상호작용을 통해 데이터의 변화(mutation), 원자적 레코드 append, 그리고 스냅샷 오퍼레이션이 어떻게 실현되는지를 알아볼 것 입니다.

