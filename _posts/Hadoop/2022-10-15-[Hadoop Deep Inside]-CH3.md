---
title: Hadoop Deep Inside - Ch.3 하둡 분산 파일 시스템(HDFS)
categories: [Hadoop, Deep Inside]
---

이번 포스팅에서는 HDFS에 대한 내용을 본격적으로 다룹니다. 당연한 소리지만 HDFS가 GFS에서 시작된 만큼 초기 HDFS는 GFS의 원리, 특성 그리고 아키텍쳐까지도 상당부분 유사합니다. 용어가 조금씩 달라지는 부분이 있다는 정도로 이해하시면 편할 겁니다.

물론 이후의 하둡이 버전업되면서 독자적인 노선을 밟게 됩니다. 특히 2.0 버전부터 Namenode의 이중화, yarn 도입과 같은 큰 변화들이 있었습니다. 이러한 점에서 HDFS에 대한 포스팅은 크게 두 파트로 나누어 작성할 예정입니다. 첫째는, HDFS의 초기 모델에 대한 이야기 그리고 둘째는 하둡 2.0버전 이후의 HDFS에 대한 이야기입니다.

# 1. GFS와 HDFS

GFS에 대해 공부한만큼, GFS와 HDFS를 간단히 비교하는 것으로 첫 단락을 시작해보겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a4b5f0df-58bb-4a27-b63f-453c37fa97f5/Untitled.png)

대부분의 분산 시스템은 크게 두 가지 유형으로 나눌 수 있습니다. Master-Slave 구조와 Master가 없는 구조로 말이죠. 보시다시피 GFS는 대표적인 Master-Slave 구조의 분산 시스템입니다. GFS master가 이름 그대로 master이고 GFS chunkserver가 slave입니다.

일반적으로 Master-Slave구조의 시스템에서 slave는 다수로 구성이 되고 쉽게 확장가능합니다. 이를 ‘scale out이 용이하다’라고 말합니다. 이는 비단 하둡에 국한된 것이 아니라 대부분의 분산 플랫폼들이 slave의 scale out이 용이한 아키텍쳐를 가지고 있습니다.

실제로 GFS는 단일 master와 다수의 slave구조를 가지고 있습니다. 이러한 구조에서 중요한건 master에 부하가 가지 않는 상황을 만들어 주는 것입니다. 이는 마스터 쪽에 장애가 발생 시, 전체 클러스터의 장애로 이어지기 때문입니다. 위의 그림에서 굵은 화살표로 표현된 chunk data 부분을 보면, 실질적인 데이터를 주고받는 작업은 클라이언트와 chunkserver 사이에서만 일어난다는 것을 확인할 수 있습니다. 데이터를 주고받는 일은 트래픽이 많이 발생하기 때문에 이를 chunkserver에 전담시킴으로서 최대한 마스터의 부하가 안가도록 하기위한 설계한 것이죠.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29cc0d2d-0aeb-4fc8-97d3-1a7c718b63d9/Untitled.png)

이번엔 HDFS를 보도록 하죠. HDFS도 Master-slave 구조를 가지고 있습니다. 앞서 말했듯 GFS와 상당히 유사한 형태죠. 다만, 일부 다른점도 존재합니다. 가령 GFS에서는 Master라고 불리던 서버가 HDFS에서는 Name Node로 불리고 GFS에서 Chunk Server라고 불리던 서버는 Data Node라고 불립니다. 이뿐만아니라 GFS에서는 본적없던 Secondary namenode가 등장합니다.

또한 HDFS도 GFS와 마찬가지로 단일 master와 다수의 slave구조를 가지고 있습니다. 이때문에 HDFS 역시 scale out이 용이하죠. 그리고 HDFS도 master의 부하를 줄이기위해 실제 데이터를 주고받는 작업은 Client와 Datanode 사이에서만 발생하록 설계되었습니다. 대신 namenode는 namespace에서 block(GFS에서의 chunk) 데이터들의 metadata를 보관합니다. 이는 GFS의 master도 마찬가지입니다.
