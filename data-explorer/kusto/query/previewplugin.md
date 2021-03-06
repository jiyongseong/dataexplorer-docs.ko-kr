---
title: preview 플러그 인-Azure 데이터 탐색기 | Microsoft Docs
description: 이 문서에서는 Azure 데이터 탐색기의 미리 보기 플러그 인을 설명 합니다.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 455ff1d4d3a42c09a39673028405d51b7acd1f5b
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92251642"
---
# <a name="preview-plugin"></a>preview 플러그 인

입력 레코드 집합에서 지정 된 수의 행을 포함 하는 테이블 및 입력 레코드 집합의 총 레코드 수를 반환 합니다.

```kusto
T | evaluate preview(50)
```

## <a name="syntax"></a>구문

`T` `|` `evaluate` `preview(` *NumberOfRows* `)`

## <a name="returns"></a>반환

`preview`플러그 인은 다음과 같은 두 개의 결과 테이블을 반환 합니다.
* 지정 된 수의 행을 포함 하는 테이블입니다.
  예를 들어 위의 예제 쿼리는를 실행 하는 것과 같습니다 `T | take 50` .
* 입력 레코드 집합의 레코드 수를 포함 하는 단일 행/열이 있는 테이블입니다.
  예를 들어 위의 예제 쿼리는를 실행 하는 것과 같습니다 `T | count` .

> [!TIP]
> `evaluate`이 앞에 복잡 한 필터를 포함 하는 테이블 형식 원본이 나 대부분의 원본 테이블 열을 참조 하는 필터가 있는 경우 함수를 사용 하는 것이 좋습니다 [`materialize`](materializefunction.md) . 예:

```kusto
let MaterializedT = materialize(T);
MaterializedT | evaluate preview(50)
```