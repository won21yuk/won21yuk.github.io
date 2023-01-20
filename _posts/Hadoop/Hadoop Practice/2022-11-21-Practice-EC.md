---
title: Hadoop Practice - Erasure Coding(EC)
categories: [Hadoop, Hadoop Practice]
---

# 개요

이 포스팅은 EC에 대해 공부하면서 혼자 생각하던 내용을 기록으로 남기기 위해 작성한다. 내용의 정확성보다는 내 추론과 실제 EC의 사용결과를 비교해서 얻어낸 개인적인 결론에 관한 이야기들 뿐이다.

![EC-practice0](/images/EC-practice0.png)

RS-6-3-1024k 정책을 사용하기 위해 이전에 GCP에 구축해 뒀던 고가용성 하둡 클러스터에 6개의 데이터 노드를 추가 확장시켜 총 9개의 데이터노드가 있는 클러스터로 구성했다.

총 세개의 파일을 넣었고 fsck 명령어로 확인한 결과는 아래와 같다.

## TEST1. 약 1.7GB

![EC-practice1](/images/EC-practice1.png)

## TEST2. 약 90MB

![EC-practice2](/images/EC-practice2.png)

## TEST3. 약 400MB

![EC-practice3](/images/EC-practice3.png)

# Questions

# 1. Block group은 어떻게 형성되는가?

## 추론

HDFS에 들어오는 파일은 block size 설정과 EC정책에 따라 logical block으로 나눠질 것이다. 만약에 logical block을 EC 정책에 따라 묶지 않으면 사실상 모든 데이터에 대해 스트라이핑이 이루어지는 것과 다를 바 없을 것이고, 그러면 조각난 셀들을 가지고 block group을 만든다는 것 자체가 상당히 모호해지는 듯 하다.

그런데 logical block이 block size 설정과 EC 정책에 따라 생성된다고 생각을 해보면, block group의 생성이 자연스러워진다. RS(6, 3) 정책인 경우에 6개의 블록 당 3개의 패리티 블록이 필요하니까 6개씩 일단 Storage block을 묶고 스트라이핑하고 셀을 기준으로 다시한번 EC를 적용해서 패리티 셀을 만들고 패리티 블록까지 만든다면 block group이 자연스럽게 구성된다는 것이다.

1.7gb의 데이터를 넣은 경우에 block group이 3개가 만들어진다. Storage block이 13개 정도 만들어지니까 6 / 6 / 1 이렇게 보면 3개 만들어지는게 맞다. 마지막 블록 한개는 스트라이핑 된후 6개의 데이터 블록과 3개의 패리티 블록으로 만들어질 것이니 각 block group의 크기가 9개가 된다. 이는 위의 모든 사진들의 Average block group size의 값이 9.0인것에서 확인할 수 있다.

그러면 문제는 용량인데 fsck명령어로는 blcok group의 평균적인 용량밖에 볼 수 없다. 대략 570mb정도인데 앞서 추론했던 128*6=768에는 크게 못미친다. 이는 당연히 128mb보다 작은 logical block으로 구성된 그룹 때문이므로, 이를 반영해서 (768+768+128)/3 = 554로 계산을 하면 얼추 비슷해진다.

실제 logical block이 하나만 생성되는 90mb데이터의 경우에는 block group의 크기가 원래 파일의 크기와 정확이 일치한다. 또한 logical block이 두개 생성되는 400mb의 데이터도 EC 규칙에 따라 6개씩 묶여야하는데 2개밖에 없으니 2개의 logical block으로 그룹을 지은 후 계산하기 때문에 block group의 크기가 원본 파일의 크기와 정확히 일치하게 된다.

## 결론

block group은 HDFS block size와 EC 정책의 블록 개수에 따라 logical block이 형성이 된다. 그리고 패리티 블록을 만들고 block group이된다. 그리고 이는 fsck명령어로 확인한 파일의 상태 정보 중 Average block group size가 9인 것으로 증명가능하다.

# 2. Block group에 패리티 블록이 포함되는가?

## 추론

logical block으로 파일이 나누어지면, 스트라이핑 되고 EC정책에 따라 Storage block 6개와 패리티 블록 3개가 만들어지고 그렇게 하나의 block group된다고 결론을 냈다. 그러면 결국 logical block과 패리티 블록의 집합이라는 두 집단으로 block group이 나뉜다고 보는게 맞을 텐데, 이 패리티 블록 집합을 block group에 포함시키지 않는 다는 건 상당히 이상한 이야기로 느껴진다.

그런데 앞선 단락에서보면 block group의 평균 용량이 fsck명령어로 확인되는데 block group에 포함된 logical block의 용량과 일치하는 것을 볼 수 있다. 이 이야기는 결국 패리티 블록은 전혀 계산에 포함되지 않는다는 것인데, 이 자체로만 block group에 패리티 블록이 포함되지 않는다고 말할 수 있느냐하면 또 복잡하다.

![EC-practice4](/images/EC-practice4.png)

위의 그림은 야후 재팬에서 사용한 ppt의 일부인데 block group을 패리티 블록도 포함해서 정의하고 있다. 사실 block group을 명확하게 정의하고 있는 자료를 사실 찾기 상당히 어려웠다. 이게 유일하게 block group에 대한 뚜렷한 정의가 나타난 자료인데 이 자료를 전적으로 신뢰하자면, logical block이 최초에 block group으로 묶이면서 가상의 패리티 블록 즉, logical parity block도 생성되었다고 보는게 맞지않는가 하는 추론은 가능하다.

근데 그럴러면 logical block은 block group으로 묶인 순간 완전히 그 정체성이 사라져야함이 옳다. logical block은 단순히 block group을 형성하는데만 사용되고 이후에는 스트라이핑 되어야할 하나의 파일 조각으로 치부하는 것이 합리적이지 않나 싶다는 것이다. 하지만 logical block은 storage block의 집합으로 block ID를 부여받고 blockMap에서 관리되고 있다.

![EC-practice5](/images/EC-practice5.png)

그러면 이 상태에서 또 다른 논리를 전개해보면, 스트라이핑해서 EC 정책에 따라 6개의 데이터 셀로 3개의 패리티 셀을 만들고 라운드 로빈방식으로 하나씩 데이터 노드에 집어넣으면 총 9개의 storage blcok이 생성이 되고 이 storage block 9개가 묶여서 하나의 block group이 될 수 있다. 즉, EC 정책에서 말하는 6개의 블록과 3개의 패리티 블록이라는 것이 사실상 관념적인 것에 불과하다면 어떨까?

만약 스트라이핑된 셀이 완전히 데이터 셀과 패리티 셀 크게 구분 없이 라운드로빈방식으로 9개의 데이터노드에 저장된다고 생각하면, 나중에 만들어진 패리티 셀들의 크기는 인식하지 못하지만, 기존파일을 논리적으로 나눈 logical block들을 스트라이핑한 데이터셀의 용량은 여전히 기억하는 것이 가능하지 않을까하는 생각이든다.

아니면 지극히 단순한 단 한가지의 가능성이 있다면, block group이 데이터 블록과 패리티 블록으로 구성이 되어있는데 fsck 명령어로 패리티 블록의 용량만 계산하지만 않는다는 것 정도? 이렇게 넘겨버려도 그냥저냥 넘어갈 수 는 있을지도 모르겠다.

## 결론

어쨋든 블록 그룹의 크기는 9가 맞는데, 블록 그룹이 갖는 평균 용량에는 패리티 블록(셀)이 포함이 안된다는 사실 하나는 확실하다.

# 3. 패리티 블록의 크기는 얼마인가?

## 추론

fsck로는 확인할 수 없지만, hdfs 상의 용량을 확인해보면 패리티블록이 만들어졌는지 확인이 가능하지 않을까하는 생각이 든다.

hdfs에 저장된 파일을 모두 지운 후 hdfs dfs -df 명령어로 확인한 hdfs에서 사용중인 용량은 3505815644 바이트 = 약 3.5기가 정도가 상시적으로 사용중이다.

![EC-practice6](/images/EC-practice6.png)

여기서 1.7기가 정도의 데이터를 한번 넣어본다.

![EC-practice7](/images/EC-practice7.png)

그러면 사용량이 6136520795 바이트 = 약 6.1GB 인데 이전과 비교해보면 약 2.6GB 정도 추가된 것을 확인할 수 있다. 이론적으로 1.7GB의 데이터를 집어넣었을 때 그 크기의 50%가 패리티 블록의 크기니까 약 850mb정도가 패리티 블록의 크기라고 생각하면 얼추 2.6GB되니 패리티 블록이 hdfs에 저장된 것을 확신할 수 있다.

## 결론

EC를 사용하면 패리티블록이 파일 용량의 50%만큼 만들어진다. 이는 파일을 넣기 전후의 용량 비교를 통해 확인할 수 있다.

# 마무리

사실 대단한 마무리가 있는 것은 아니지만, 어쨌든 내가 내린 최종 결론은 ‘block group에 패리티 블록이 포함된다’이다. 사실 이 가설을 기각할만한 근거도 사실 부족하고 채택하기도 부족하고 한게 사실인데, 그래도 야후를 믿기로 했다. 야후는 어쨌든 하둡의 근간이라고 불러도 어색하지 않을 만큼 하둡의 시작에 있는 기업인데 그 기업이 가장 인기가 좋은 나라가 또 일본이다. 그러니 일본에서 EC에대해 만든 PPT가 거짓일리는 없다고 생각한다. 결국 무슨 이유에서인지는 찾지 못했지만 fsck로 파일의 상태를 확인할 때는 패리티 블록의 크기는 명확하게 표시되지 않는다는 사실 정도로 마무리 할 수 있을 듯하다. 다만, 추후 새로이 알게되는 내용이 있다면 그때 반드시 보충할 것이다.