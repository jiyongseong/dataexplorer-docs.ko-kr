---
title: funnel_sequence 플러그 인-Azure 데이터 탐색기
description: 이 문서에서는 Azure 데이터 탐색기에서 funnel_sequence 플러그 인을 설명 합니다.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 34159fb6d02cd30907924109c861d5e9fd963568
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92252343"
---
# <a name="funnel_sequence-plugin"></a>funnel_sequence 플러그 인

상태 시퀀스를 취한 사용자의 고유 개수와 시퀀스를 실행 하 고 그 뒤에 오는 이전/다음 상태의 분포를 계산 합니다. 

```kusto
T | evaluate funnel_sequence(id, datetime_column, startofday(ago(30d)), startofday(now()), 10m, 1d, state_column, dynamic(['S1', 'S2', 'S3']))
```

## <a name="syntax"></a>구문

*T* `| evaluate` `funnel_sequence(` *idcolumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *MaxSequenceStepWindow*, *Step*, *StateColumn*, *Sequence*`)`

## <a name="arguments"></a>인수

* *T*: 입력 테이블 형식 식입니다.
* *IdColum*: 열 참조는 원본 식에 있어야 합니다.
* *TimelineColumn*: 타임 라인을 나타내는 열 참조는 원본 식에 있어야 합니다.
* *Start*: 분석 시작 기간의 스칼라 상수 값입니다.
* *끝*: 분석 종료 기간의 스칼라 상수 값입니다.
* *MaxSequenceStepWindow*: 시퀀스의 두 순차 단계 사이에서 허용 되는 최대 timespan의 스칼라 상수 값입니다.
* *단계*: 분석 단계 기간 (bin)의 스칼라 상수 값입니다.
* *StateColumn*: 상태를 나타내는 열 참조는 원본 식에 있어야 합니다.
* *Sequence*: 시퀀스 값이 포함 된 상수 동적 배열입니다 (값은에서 조회 됨 `StateColumn` ).

## <a name="returns"></a>반환

는 분석 된 시퀀스에 대 한 sankey 다이어그램을 생성 하는 데 유용한 세 개의 출력 테이블을 반환 합니다.

* 테이블 #1-이전-시퀀스-다음 `dcount` TimelineColumn: 분석 된 시간 창 이전: 이전 상태 (검색 된 시퀀스에 대 한 이벤트만 있지만 이벤트의 이벤트가 아닌 사용자가 있을 경우 비어 있을 수 있음) 
    다음: 다음 상태 (검색 된 시퀀스에 대 한 이벤트만 포함 하 고 그 다음에 나오는 이벤트는 없는 사용자가 있을 경우 비어 있을 수 있음). 
    `dcount`: `IdColumn` 전환 된 기간 내 고유 수입니다 `prev`  -->  `Sequence`  -->  `next` . 
    samples: `IdColumn` 행의 시퀀스에 해당 하는 id (에서)의 배열입니다 (최대 128 id가 반환 됨). 

* 테이블 #2-이전 시퀀스 `dcount` TimelineColumn: 분석 된 시간 창 이전: 이전 상태 (검색 된 시퀀스에 대 한 이벤트만 있지만 이벤트의 이벤트가 아닌 사용자가 있을 경우 비어 있을 수 있음) 
    `dcount`: `IdColumn` 전환 된 기간 내 고유 수입니다 `prev`  -->  `Sequence`  -->  `next` . 
    samples: `IdColumn` 행의 시퀀스에 해당 하는 id (에서)의 배열입니다 (최대 128 id가 반환 됨). 

* 테이블 #3-시퀀스-다음 `dcount` TimelineColumn: 분석 된 시간 창 다음: 다음 상태 (검색 된 시퀀스에 대 한 이벤트만 포함 하 고 그 다음에 나오는 이벤트는 없는 사용자가 있을 경우 비어 있을 수 있음). 
    `dcount`: `IdColumn` 전환 된 기간 내 고유 수입니다 `prev`  -->  `Sequence`  -->  `next` .
    samples: `IdColumn` 행의 시퀀스에 해당 하는 id (에서)의 배열입니다 (최대 128 id가 반환 됨). 


## <a name="examples"></a>예

### <a name="exploring-storm-events"></a>스톰 이벤트 탐색 

다음 쿼리는 테이블 StormEvents (2007에 대 한 날씨 통계)를 확인 하 고 2007에서 모든 토네이도 이벤트가 발생 하기 전/후에 발생 한 이벤트를 보여 줍니다.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Looking on StormEvents statistics: 
// Q1: What happens before Tornado event?
// Q2: What happens after Tornado event?
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, dynamic(['Tornado']))
```

결과에는 다음 세 개의 테이블이 포함 됩니다.

* 표 #1: 시퀀스 전후에 발생 하는 모든 가능한 변형입니다. 예를 들어 두 번째 줄은 다음 시퀀스가 있는 87 다른 이벤트가 있음을 의미 합니다. `Hail` -> `Tornado` -> `Hail`


|`StartTime`|`prev`|`next`|`dcount`|
|---|---|---|---|
|2007-01-01 00:00:00.0000000|||293|
|2007-01-01 00:00:00.0000000|우박|우박|87|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|뇌우를 동반한 바람|77|
|2007-01-01 00:00:00.0000000|우박|뇌우를 동반한 바람|28|
|2007-01-01 00:00:00.0000000|우박||28|
|2007-01-01 00:00:00.0000000||우박|27|
|2007-01-01 00:00:00.0000000||뇌우를 동반한 바람|25|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|우박|24|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람||24|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|갑작스러운 홍수|12|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|갑작스러운 홍수|8|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수||8|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|뇌우를 동반한 바람|6|
|2007-01-01 00:00:00.0000000||깔때기형 클라우드|6|
|2007-01-01 00:00:00.0000000||갑작스러운 홍수|6|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|깔때기형 클라우드|6|
|2007-01-01 00:00:00.0000000|우박|갑작스러운 홍수|4|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|뇌우를 동반한 바람|4|
|2007-01-01 00:00:00.0000000|우박|깔때기형 클라우드|4|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|우박|4|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드||4|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|깔때기형 클라우드|3|
|2007-01-01 00:00:00.0000000|과도 한 비|뇌우를 동반한 바람|2|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|깔때기형 클라우드|2|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|우박|2|
|2007-01-01 00:00:00.0000000|강한 바람|뇌우를 동반한 바람|1|
|2007-01-01 00:00:00.0000000|과도 한 비|갑작스러운 홍수|1|
|2007-01-01 00:00:00.0000000|과도 한 비|우박|1|
|2007-01-01 00:00:00.0000000|우박|홍수|1|
|2007-01-01 00:00:00.0000000|번개|우박|1|
|2007-01-01 00:00:00.0000000|과도 한 비|번개|1|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|과도 한 비|1|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|홍수|1|
|2007-01-01 00:00:00.0000000|홍수|갑작스러운 홍수|1|
|2007-01-01 00:00:00.0000000||과도 한 비|1|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|번개|1|
|2007-01-01 00:00:00.0000000|번개|뇌우를 동반한 바람|1|
|2007-01-01 00:00:00.0000000|홍수|뇌우를 동반한 바람|1|
|2007-01-01 00:00:00.0000000|우박|번개|1|
|2007-01-01 00:00:00.0000000||번개|1|
|2007-01-01 00:00:00.0000000|Tropical 스톰|허리케인 (Typhoon)|1|
|2007-01-01 00:00:00.0000000|Coastal 홍수||1|
|2007-01-01 00:00:00.0000000|현재 Rip||1|
|2007-01-01 00:00:00.0000000|눈 덮인||1|
|2007-01-01 00:00:00.0000000|강한 바람||1|

* 테이블 #2: 이전 이벤트 별로 그룹화 된 모든 고유 이벤트를 표시 합니다. 예를 들어 두 번째 줄에는 이전에 발생 한 총 150 이벤트가 있음을 보여 줍니다 `Hail` `Tornado` .

|`StartTime`|`prev`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||331|
|2007-01-01 00:00:00.0000000|우박|150|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|135|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|28|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|22|
|2007-01-01 00:00:00.0000000|과도 한 비|5|
|2007-01-01 00:00:00.0000000|홍수|2|
|2007-01-01 00:00:00.0000000|번개|2|
|2007-01-01 00:00:00.0000000|강한 바람|2|
|2007-01-01 00:00:00.0000000|눈 덮인|1|
|2007-01-01 00:00:00.0000000|현재 Rip|1|
|2007-01-01 00:00:00.0000000|Coastal 홍수|1|
|2007-01-01 00:00:00.0000000|Tropical 스톰|1|

* 테이블 #3: 다음 이벤트로 그룹화 된 모든 고유 이벤트를 표시 합니다. 예를 들어 두 번째 줄에는 이후 발생 한 총 143 이벤트가 있음을 보여 줍니다 `Hail` `Tornado` .

|`StartTime`|`next`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||332|
|2007-01-01 00:00:00.0000000|우박|145|
|2007-01-01 00:00:00.0000000|뇌우를 동반한 바람|143|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|32|
|2007-01-01 00:00:00.0000000|깔때기형 클라우드|21|
|2007-01-01 00:00:00.0000000|번개|4|
|2007-01-01 00:00:00.0000000|과도 한 비|2|
|2007-01-01 00:00:00.0000000|홍수|2|
|2007-01-01 00:00:00.0000000|허리케인 (Typhoon)|1|

이제 다음 시퀀스가 어떻게 진행 되는지 확인해 보겠습니다.  
`Hail` -> `Tornado` -> `Thunderstorm Wind`

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, 
dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']))
```

및를 건너뛰고를 `Table #1` `Table #2` 살펴보면 `Table #3` `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` 92 이벤트의 시퀀스가이 시퀀스와 함께 종료 되 고 41 이벤트에서와 같이 계속 해 서 `Hail` 14로 다시 설정 `Tornado` 되는 것을 확인할 수 있습니다.

|`StartTime`|`next`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||92|
|2007-01-01 00:00:00.0000000|우박|41|
|2007-01-01 00:00:00.0000000|토네이도|14|
|2007-01-01 00:00:00.0000000|갑작스러운 홍수|11|
|2007-01-01 00:00:00.0000000|번개|2|
|2007-01-01 00:00:00.0000000|과도 한 비|1|
|2007-01-01 00:00:00.0000000|홍수|1|
