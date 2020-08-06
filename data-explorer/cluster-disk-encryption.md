---
title: Azure 데이터 탐색기에서 디스크 암호화를 사용 하 여 클러스터 보안 설정-Azure Portal
description: 이 문서에서는 Azure Portal 내에서 Azure 데이터 탐색기의 디스크 암호화를 사용 하 여 클러스터를 보호 하는 방법을 설명 합니다.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/02/2020
ms.openlocfilehash: 4f0ed42727759dae54f75fcb33ba1f024c218a4a
ms.sourcegitcommit: d9fbcd6c9787f90de62e8e832c92d43b8090cbfc
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/03/2020
ms.locfileid: "87515932"
---
# <a name="secure-your-cluster-using-disk-encryption-in-azure-data-explorer---azure-portal"></a>Azure 데이터 탐색기에서 디스크 암호화를 사용 하 여 클러스터 보안 설정-Azure Portal

[Azure Disk Encryption](/azure/security/azure-security-disk-encryption-overview) 는 조직의 보안 및 규정 준수 약정에 맞게 데이터를 보호 하 고 보호 합니다. 클러스터 가상 컴퓨터의 OS 및 데이터 디스크에 대 한 볼륨 암호화를 제공 합니다. 또한 디스크 암호화 키와 암호를 제어 하 고 관리할 수 있으며 VM 디스크의 모든 데이터가 암호화 되도록 하는 [Azure Key Vault](/azure/key-vault/)와 통합 됩니다. 
  
## <a name="enable-encryption-at-rest-in-the-azure-portal"></a>Azure Portal에서 휴지 상태의 암호화 사용
  
클러스터 보안 설정을 사용 하 여 클러스터에서 디스크 암호화를 사용 하도록 설정할 수 있습니다. 클러스터에서 [휴지 상태의 암호화](/azure/security/fundamentals/encryption-atrest) 를 사용 하면 저장 된 데이터에 대 한 데이터 보호를 제공 합니다 (미사용). 

1. Azure Portal에서 Azure 데이터 탐색기 클러스터 리소스로 이동 합니다. **설정** 제목 아래에서 **보안**을 선택 합니다. 

    ![미사용 암호화 설정](media/manage-cluster-security/security-encryption-at-rest.png)

1. **보안** 창에서 **디스크 암호화** 보안 설정에 대해 **켜기** 를 선택 합니다. 

1. **저장**을 선택합니다.
 
> [!NOTE]
> 사용 하도록 설정 된 후 암호화를 사용 하지 않도록 설정 하려면 **해제** 를 선택 합니다.

## <a name="azure-data-explorer-stores-data-within-a-region"></a>Azure 데이터 탐색기는 지역 내에 데이터를 저장 합니다.

모든 Azure 데이터 탐색기 클러스터는 단일 지역의 전용 리소스에서 실행 됩니다. 모든 데이터는 지역 내에 저장 됩니다. 

## <a name="next-steps"></a>다음 단계

[클러스터 상태 확인](check-cluster-health.md)