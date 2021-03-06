---
title: 수집을 위해 Azure 데이터 탐색기에서 지 원하는 데이터 형식입니다.
description: 수집을 위해 Azure 데이터 탐색기에서 지 원하는 다양 한 데이터 및 압축 형식에 대해 알아봅니다.
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/19/2020
ms.openlocfilehash: 4f7e61755b12c84fc49373a12edc0b507aee9bf4
ms.sourcegitcommit: 2764e739b4ad51398f4f0d3a9742d7168c4f5fd7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91712054"
---
# <a name="data-formats-supported-by-azure-data-explorer-for-ingestion"></a>수집을 위해 Azure 데이터 탐색기에서 지 원하는 데이터 형식

데이터 수집은 테이블에 데이터를 추가 하 고 Azure 데이터 탐색기에서 쿼리에 사용할 수 있도록 하는 프로세스입니다. 수집 후 쿼리를 제외한 모든 수집 메서드의 경우 데이터는 지원 되는 형식 중 하나 여야 합니다. 다음 표에서는 Azure 데이터 탐색기에서 데이터 수집에 대해 지 원하는 형식을 나열 하 고 설명 합니다.

|서식   |확장명   |Description|
|---------|------------|-----------|
|ApacheAvro|`.avro`    |[논리적 형식](https://avro.apache.org/docs/current/spec.html#Logical+Types) 및 압축 코덱에 대 한 지원을 포함 하는 [Avro](https://avro.apache.org/docs/current/) 형식 `snappy` 입니다.|
|Avro     |`.avro`     |[Avro 컨테이너 파일](https://avro.apache.org/docs/current/)에 대 한 레거시 구현입니다. 지원 되는 코드는 다음과 같습니다. `null` , `deflate` ( `snappy` 사용 하는 경우 `apacheavro` 파일 형식).|
|CSV      |`.csv`      |쉼표(`,`)로 구분된 값을 사용하는 텍스트 파일입니다. [ _쉼표로 구분 된 값 (CSV) 파일에 대 한 RFC 4180: 일반 형식 및 MIME 형식_](https://www.ietf.org/rfc/rfc4180.txt)을 참조 하세요.|
|JSON     |`.json`     |JSON 개체가 `\n` 또는 `\r\n`으로 구분된 텍스트 파일입니다. [JSON Lines(JSONL)](http://jsonlines.org/)를 참조하세요.|
|MultiJSON|`.multijson`|JSON 속성 모음 배열(각각 레코드를 나타냄) 또는 공백으로 구분된 여러 속성 모음(`\n` 또는 `\r\n`)이 있는 텍스트 파일입니다. 각 속성 모음을 여러 줄에 분배할 수 있습니다. `JSON`데이터가 속성 모음이 아닌 경우에는이 형식이 보다 좋습니다.|
|ORC      |`.orc`      |[Orc 파일](https://en.wikipedia.org/wiki/Apache_ORC).|
|Parquet  |`.parquet`  |[Parquet 파일](https://en.wikipedia.org/wiki/Apache_Parquet).|
|PSV      |`.psv`      |파이프(<code>&#124;</code>)로 구분된 값을 사용하는 텍스트 파일입니다.|
|RAW      |`.raw`      |전체 내용이 단일 문자열 값인 텍스트 파일입니다.|
|SCsv     |`.scsv`     |세미콜론(`;`)으로 구분된 값을 사용하는 텍스트 파일입니다.|
|SOHsv    |`.sohsv`    |SOH로 구분된 값을 사용하는 텍스트 파일입니다. (SOH는 ASCII 코드 포인트 1이며, 이 형식은 Hive on HDInsight에서 사용됩니다.)|
|TSV      |`.tsv`      |탭(`\t`)으로 구분된 값을 사용하는 텍스트 파일입니다.|
|TSVE     |`.tsv`      |탭(`\t`)으로 구분된 값을 사용하는 텍스트 파일입니다. 백슬래시(`\`) 문자가 이스케이프에 사용됩니다.|
|TXT      |`.txt`      |`\n`으로 구분된 줄을 사용하는 텍스트 파일입니다. 빈 줄은 건너뜁니다.|
|W3CLOGFILE |`.log`    |W3C의 [웹 로그 파일](https://www.w3.org/TR/WD-logfile.html) 형식입니다.|

## <a name="supported-data-compression-formats"></a>지원 되는 데이터 압축 형식

다음 압축 알고리즘을 통해 blob 및 파일을 압축할 수 있습니다.

|압축|확장명|
|-----------|---------|
|GZip       |.gz      |
|Zip        |.zip     |

Blob 또는 파일 이름에 확장을 추가 하 여 압축을 표시 합니다.

예:
* `MyData.csv.zip` CSV로 포맷 되 고 ZIP으로 압축 된 (archive 또는 단일 파일) blob 또는 파일을 나타냅니다.
* `MyData.json.gz` GZip으로 압축 된 blob 또는 JSON 형식의 파일을 나타냅니다.

형식 확장을 포함 하지 않는 Blob 또는 파일 이름 (예: `MyData.zip` )도 지원 됩니다. 이 경우 파일 형식을 유추할 수 없으므로 수집 속성으로 지정해야 합니다.

> [!NOTE]
> 일부 압축 형식은 압축된 스트림의 일부로 원래 파일 확장명을 추적합니다. 이 확장명은 일반적으로 파일 형식을 확인하는 경우에는 무시됩니다. 압축된 Blob 또는 파일의 이름에서 파일 형식을 확인할 수 없는 경우 `format` 수집 속성을 통해 이를 지정해야 합니다.

## <a name="next-steps"></a>다음 단계

* [데이터](ingest-data-overview.md) 수집에 대 한 자세한 정보
* [Azure 데이터 탐색기 데이터 수집 속성](ingestion-properties.md) 에 대해 자세히 알아보기
