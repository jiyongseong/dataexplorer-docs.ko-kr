---
title: Jupyter Notebook를 사용 하 여 Azure 데이터 탐색기에서 데이터 분석
description: 이 항목에서는 Jupyter Notebook 및 kqlmagic 확장을 사용 하 여 Azure 데이터 탐색기에서 데이터를 분석 하는 방법을 보여 줍니다.
author: orspod
ms.author: orspodek
ms.reviewer: maraheja
ms.service: data-explorer
ms.topic: how-to
ms.date: 10/20/2020
ms.openlocfilehash: 857c654c70a2170a42902514718a52fbf7b02944
ms.sourcegitcommit: 455d902bad0aae3e3d72269798c754f51442270e
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93349463"
---
# <a name="use-a-jupyter-notebook-and-kqlmagic-extension-to-analyze-data-in-azure-data-explorer"></a>Jupyter Notebook 및 kqlmagic 확장을 사용 하 여 Azure에서 데이터 분석 데이터 탐색기

Jupyter Notebook은 라이브 코드, 수식, 시각화 및 내레이션 텍스트를 포함하는 문서를 만들고 공유할 수 있는 오픈 소스 웹 애플리케이션입니다. 사용에는 데이터 정리 및 변환, 숫자 시뮬레이션, 통계 모델링, 데이터 시각화 및 기계 학습이 포함됩니다.
[Jupyter Notebook](https://jupyter.org/)은 추가 명령을 지원하여 커널의 기능을 확장하는 매직 함수를 지원합니다. kqlmagic는 Jupyter Notebook에서 Python 커널의 기능을 확장 하는 명령으로, Kusto 언어 쿼리를 기본적으로 실행할 수 있습니다. 명령과 통합 된 리치 Plot.ly 라이브러리를 사용 하 여 Python 및 Kusto 쿼리 언어와 손쉽게 통합 하 여 데이터를 쿼리하고 시각화할 수 있습니다 `render` . 쿼리 실행을 위한 데이터 원본이 지원됩니다. 이러한 데이터 원본에는 로그 및 원격 분석 데이터에 대 한 빠르고 확장성이 뛰어난 데이터 탐색 서비스인 Azure 데이터 탐색기와 Azure Monitor 로그 및 Application Insights 포함 됩니다. 또한 kqlmagic은 Azure Data Studio, Jupyter 랩 및 Visual Studio Code Jupyter 확장을 사용 합니다.

## <a name="prerequisites"></a>사전 요구 사항

- Azure Active Directory (Azure AD)의 멤버인 조직 전자 메일 계정입니다.
- 로컬 컴퓨터에 설치 된 Jupyter Notebook [Azure Data Studio](/sql/azure-data-studio/notebooks/notebooks-kqlmagic?view=sql-server-ver15) 사용

## <a name="install-kqlmagic-library"></a>Kqlmagic 라이브러리 설치

1. Kqlmagic 설치:

    ```python
    !pip install Kqlmagic --no-cache-dir  --upgrade
    ```

1. Load kqlmagic:

    ```python
    %reload_ext Kqlmagic
    ```
    > [!NOTE]
    > 커널을 클릭 하 여 커널 버전을 Python 3.6로 변경 합니다. 커널 > 변경 커널 > Python 3.6
    
## <a name="connect-to-the-azure-data-explorer-help-cluster"></a>Azure Data Explorer 도움말 클러스터에 연결

다음 명령을 사용하여 ‘도움말’ 클러스터에 호스트된 ‘샘플’ 데이터베이스에 연결합니다. Microsoft Azure AD 않은 사용자의 경우 테 넌 트 이름을 `Microsoft.com` AZURE AD 테 넌 트로 바꿉니다.

```python
%kql AzureDataExplorer://tenant="Microsoft.com";code;cluster='help';database='Samples'
```

> [!Note]
> 고유의 ADX 클러스터를 사용하는 경우 다음과 같이 연결 문자열에서 지역을 포함해야 합니다.   
   ```%kql azuredataexplorer://tenant="youecompany.com";code;cluster='mycluster.westus';database='mykustodb'```

## <a name="query-and-visualize"></a>쿼리 및 시각화

[렌더링 연산자](kusto/query/renderoperator.md)를 사용하여 데이터를 쿼리하고 ploy.ly 라이브러리를 사용하여 데이터를 시각화합니다. 이 쿼리 및 시각화는 네이티브 KQL을 사용하는 통합 환경을 제공합니다. Kqlmagic은 `timepivot`, `pivotchart` 및 `ladderchart`를 제외한 대부분의 차트를 지원합니다. `kind`, `ysplit` 및 `accumulate`를 제외한 모든 특성에서 렌더링이 지원됩니다. 

### <a name="query-and-render-piechart"></a>원형 차트 쿼리 및 렌더링

```python
%%kql
StormEvents
| summarize statecount=count() by State
| sort by statecount 
| limit 10
| render piechart title="My Pie Chart by State"
```

### <a name="query-and-render-timechart"></a>시간 차트 쿼리 및 렌더링

```python
%%kql
StormEvents
| summarize count() by bin(StartTime,7d)
| render timechart
```

> [!NOTE]
> 이러한 차트는 대화형으로 작동합니다. 시간 범위를 선택하여 특정 시간을 확대합니다.

### <a name="customize-the-chart-colors"></a>차트 색 사용자 지정

기본 색상표가 마음에 들지 않으면 색상표 옵션을 사용 하 여 차트를 사용자 지정 합니다. 사용 가능한 색상표는 [kqlmagic 쿼리 차트 결과에 대 한 색상표 선택](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb) 에서 찾을 수 있습니다.

1. 팔레트 목록의 경우 다음을 수행합니다.

    ```python
    %kql --palettes -popup_window
    ```

1. `cool` 색상표를 선택하고 쿼리를 다시 렌더링합니다.

    ```python
    %%kql -palette_name "cool"
    StormEvents
    | summarize statecount=count() by State
    | sort by statecount
    | limit 10
    | render piechart title="My Pie Chart by State"
    ```

## <a name="parameterize-a-query-with-python"></a>Python을 사용하여 쿼리를 매개 변수화

Kqlmagic을 통해 Kusto 쿼리 언어와 Python을 간단하게 바꿔서 사용할 수 있습니다. 자세한 정보: [Python을 사용 하 여 kqlmagic 쿼리 매개 변수화](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb)

### <a name="use-a-python-variable-in-your-kql-query"></a>KQL 쿼리에 Python 변수 사용

쿼리에 Python 변수 값을 사용하여 데이터를 필터링할 수 있습니다.

```python
statefilter = ["TEXAS", "KANSAS"]
```

```python
%%kql
let _state = statefilter;
StormEvents 
| where State in (_state) 
| summarize statecount=count() by bin(StartTime,1d), State
| render timechart title = "Trend"
```

### <a name="convert-query-results-to-pandas-dataframe"></a>쿼리 결과를 Pandas DataFrame으로 변환

Pandas DataFrame에서 KQL 쿼리의 결과에 액세스할 수 있습니다. `_kql_raw_result_` 변수를 통해 마지막으로 실행된 쿼리 결과에 액세스하고 다음과 같이 결과를 Pandas DataFrame으로 쉽게 변환합니다.

```python
df = _kql_raw_result_.to_dataframe()
df.head(10)
```

### <a name="example"></a>예제

다양한 분석 시나리오에서 많은 쿼리를 포함하고 한 쿼리의 결과를 후속 쿼리에 피드하는 재사용 가능한 Notebook을 만들 수 있습니다. 아래 예제에서는 Python 변수 `statefilter`를 사용하여 데이터를 필터링합니다.

1. 쿼리를 실행하여 `DamageProperty`가 최댓값인 상위 10개 주를 확인합니다.

    ```python
    %%kql
    StormEvents
    | summarize max(DamageProperty) by State
    | order by max_DamageProperty desc
    | limit 10
    ```

1. 쿼리를 실행하여 상위 주를 추출하고 Python 변수에 설정합니다.

    ```python
    df = _kql_raw_result_.to_dataframe()
    statefilter =df.loc[0].State
    statefilter
    ```

1. `let` 문과 Python 변수를 사용하여 쿼리를 실행합니다.

    ```python
    %%kql
    let _state = statefilter;
    StormEvents 
    | where State in (_state)
    | summarize statecount=count() by bin(StartTime,1d), State
    | render timechart title = "Trend"
    ```

1. help 명령을 실행합니다.

    ```python
    %kql --help "help"
    ```

> [!TIP]
> 사용 가능한 모든 구성에 대 한 정보를 받으려면를 사용 `%config Kqlmagic` 합니다. 연결 문제 및 잘못 된 쿼리와 같은 Kusto 오류 문제를 해결 하 고 캡처하려면 다음을 사용 합니다. `%config Kqlmagic.short_errors=False`

## <a name="next-steps"></a>다음 단계

help 명령을 실행하여 지원되는 모든 기능을 포함하는 다음 샘플 Notebook을 탐색합니다.
- [Azure 데이터 탐색기에 대 한 kqlmagic 시작](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FQuickStart.ipynb) 
- [Application Insights에 대 한 kqlmagic 시작](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FQuickStartAI.ipynb) 
- [Azure Monitor 로그에 대 한 kqlmagic 시작](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FQuickStartLA.ipynb) 
- [Python을 사용 하 여 kqlmagic 쿼리 Parametrize](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb) 
- [Kqlmagic 쿼리 차트 결과에 대 한 색상표 선택](https://mybinder.org/v2/gh/Microsoft/jupyter-kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb)