---
title: endofday ()-Azure 데이터 탐색기 | Microsoft Docs
description: 이 문서에서는 Azure 데이터 탐색기의 endofday ()에 대해 설명 합니다.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d87f188cf0ae3639df4261d1813ebbe2a6788192
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92245083"
---
# <a name="endofday"></a>endofday()

지정 된 경우 오프셋으로 이동한 날짜를 포함 하는 일의 끝을 반환 합니다.

## <a name="syntax"></a>구문

`endofday(`*날짜* [ `,` *오프셋*]`)`

## <a name="arguments"></a>인수

* `date`: 입력 날짜입니다.
* `offset`: 입력 날짜 (정수, 기본값-0)의 선택적 오프셋 일 수입니다.

## <a name="returns"></a>반환

지정 된 경우 오프셋을 사용 하 여 지정 된 *날짜* 값에 대 한 날짜의 끝을 나타내는 날짜/시간입니다.

## <a name="example"></a>예제

```kusto
  range offset from -1 to 1 step 1
 | project dayEnd = endofday(datetime(2017-01-01 10:10:17), offset) 
```

|dayEnd|
|---|
|2016-12-31 23:59:59.9999999까지|
|2017-01-01 23:59:59.9999999까지|
|2017-01-02 23:59:59.9999999까지|