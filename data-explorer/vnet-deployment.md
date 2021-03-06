---
title: Virtual Network에 Azure 데이터 탐색기 배포
description: Virtual Network에 Azure 데이터 탐색기를 배포 하는 방법을 알아봅니다.
author: orspod
ms.author: orspodek
ms.reviewer: basaba
ms.service: data-explorer
ms.topic: how-to
ms.date: 10/31/2019
ms.openlocfilehash: 196d1f5a72b0b99186f4739ce6b6a25c23b6c325
ms.sourcegitcommit: 2bdb904e6253c9ceb8f1eaa2da35fcf27e13a2cd
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97091371"
---
# <a name="deploy-azure-data-explorer-cluster-into-your-virtual-network"></a>Virtual Network에 Azure 데이터 탐색기 클러스터 배포

이 문서에서는 Azure 데이터 탐색기 클러스터를 사용자 지정 Azure Virtual Network에 배포할 때 표시 되는 리소스에 대해 설명 합니다. 이 정보는 클러스터를 Virtual Network (VNet)의 서브넷에 배포 하는 데 도움이 됩니다. Azure Virtual Network에 대 한 자세한 내용은 [azure Virtual Network 란?](/azure/virtual-network/virtual-networks-overview) 을 참조 하세요.

:::image type="content" source="media/vnet-deployment/vnet-diagram.png" alt-text="계통도 가상 네트워크 아키텍처를 보여 주는 다이어그램"::: 

Azure 데이터 탐색기는 VNet (Virtual Network)의 서브넷에 클러스터를 배포 하는 것을 지원 합니다. 이 기능을 통해 다음을 수행할 수 있습니다.

* Azure 데이터 탐색기 클러스터 트래픽에 nsg ( [네트워크 보안 그룹](/azure/virtual-network/security-overview) ) 규칙을 적용 합니다.
* 온-프레미스 네트워크를 Azure 데이터 탐색기 클러스터의 서브넷에 연결 합니다.
* [서비스 엔드포인트](/azure/virtual-network/virtual-network-service-endpoints-overview)를 사용 하 여 데이터 연결 원본 ([이벤트 허브](/azure/event-hubs/event-hubs-about) 및 [Event Grid](/azure/event-grid/overview))을 보호 합니다.

## <a name="access-your-azure-data-explorer-cluster-in-your-vnet"></a>VNet에서 Azure 데이터 탐색기 클러스터에 액세스

각 서비스에 대 한 다음 IP 주소 (엔진 및 데이터 관리 서비스)를 사용 하 여 Azure 데이터 탐색기 클러스터에 액세스할 수 있습니다.

* **개인 IP**: VNet 내의 클러스터에 액세스 하는 데 사용 됩니다.
* **공용 IP**: 관리 및 모니터링을 위해 VNet 외부에서 클러스터에 액세스 하는 데 사용 되며, 클러스터에서 시작 된 아웃 바운드 연결의 원본 주소로 사용 됩니다.

서비스에 액세스 하기 위해 생성 되는 DNS 레코드는 다음과 같습니다. 

* `[clustername].[geo-region].kusto.windows.net` (엔진) `ingest-[clustername].[geo-region].kusto.windows.net` (데이터 관리)는 각 서비스에 대 한 공용 IP에 매핑됩니다. 

* `private-[clustername].[geo-region].kusto.windows.net`(엔진) `ingest-private-[clustername].[geo-region].kusto.windows.net` \\ `private-ingest-[clustername].[geo-region].kusto.windows.net` (데이터 관리)는 각 서비스에 대 한 개인 IP에 매핑됩니다.

## <a name="plan-subnet-size-in-your-vnet"></a>VNet에서 서브넷 크기를 계획 합니다.

Azure 데이터 탐색기 클러스터를 호스트 하는 데 사용 되는 서브넷의 크기는 서브넷이 배포 된 후에 변경할 수 없습니다. VNet에서 Azure 데이터 탐색기는 각 VM에 대해 하나의 개인 IP 주소를 사용 하 고 내부 부하 분산 장치 (엔진 및 데이터 관리)에 대해 두 개의 개인 IP 주소를 사용 합니다. 또한 Azure 네트워킹은 각 서브넷에 대해 5 개의 IP 주소를 사용 합니다. Azure 데이터 탐색기는 데이터 관리 서비스용 두 Vm을 프로 비전 합니다. 엔진 서비스 Vm은 사용자 구성 확장 용량 마다 프로 비전 됩니다.

총 IP 주소 수:

| 기능 | 주소 수 |
| --- | --- |
| 엔진 서비스 | 인스턴스당 1 개 |
| 데이터 관리 서비스 | 2 |
| 내부 부하 분산 장치 | 2 |
| Azure 예약 주소 | 5 |
| **합계** | **#engine_instances + 9** |

> [!IMPORTANT]
> Azure 데이터 탐색기 배포 된 후에는 서브넷 크기를 변경할 수 없으므로 미리 계획 해야 합니다. 따라서 필요에 따라 서브넷 크기를 예약 합니다.

## <a name="service-endpoints-for-connecting-to-azure-data-explorer"></a>Azure 데이터 탐색기에 연결 하기 위한 서비스 끝점

[Azure 서비스 끝점](/azure/virtual-network/virtual-network-service-endpoints-overview) 을 사용 하면 azure 다중 테 넌 트 리소스를 가상 네트워크로 보호할 수 있습니다.
Azure 데이터 탐색기 클러스터를 서브넷에 배포 하면 Azure 데이터 탐색기 서브넷에 대 한 기본 리소스를 제한 하는 동시에 [이벤트 허브](/azure/event-hubs/event-hubs-about) 또는 [Event Grid](/azure/event-grid/overview) 를 사용 하 여 데이터 연결을 설정할 수 있습니다.

> [!NOTE]
> [저장소](/azure/storage/common/storage-introduction) 및 [이벤트 허브](/azure/event-hubs/event-hubs-about)에서 eventgrid 설치를 사용 하는 경우 구독에 사용 되는 저장소 계정은 [방화벽 구성](/azure/storage/common/storage-network-security)에서 신뢰할 수 있는 azure platform 서비스를 허용 하는 동시에 azure 데이터 탐색기의 서브넷에 대 한 서비스 끝점을 사용 하 여 잠글 수 있지만, 이벤트 허브는 신뢰할 수 있는 [Azure Platform 서비스](/azure/event-hubs/event-hubs-service-endpoints)를 지원 하지 않으므로 서비스 끝점을 사용 하도록 설정할 수 없습니다

## <a name="private-endpoints"></a>프라이빗 엔드포인트

[개인 끝점](/azure/private-link/private-endpoint-overview) 은 Azure 리소스 (예: 저장소/이벤트 허브/Data Lake Gen 2)에 대 한 개인 액세스를 허용 하 고, Virtual Network에서 개인 IP를 사용 하 여 리소스를 VNet에 효과적으로 제공 합니다.
이벤트 허브 및 저장소와 같은 데이터 연결에서 사용 되는 리소스에 대 한 [개인 끝점](/azure/private-link/private-endpoint-overview) 을 만들고, 저장소, Data Lake Gen 2 등의 외부 테이블과 VNet에서 SQL Database 하 여 내부 리소스에 대해 개인적으로 액세스 합니다.

 > [!NOTE]
 > 개인 끝점을 설정 하려면 [DNS를 구성](/azure/private-link/private-endpoint-dns)해야 합니다. [Azure 사설 DNS 영역](/azure/dns/private-dns-privatednszone) 설치만 지원 합니다. 사용자 지정 DNS 서버는 지원 되지 않습니다. 

## <a name="dependencies-for-vnet-deployment"></a>VNet 배포에 대 한 종속성

### <a name="network-security-groups-configuration"></a>네트워크 보안 그룹 구성

[NSG (네트워크 보안 그룹)](/azure/virtual-network/security-overview) 는 VNet 내에서 네트워크 액세스를 제어 하는 기능을 제공 합니다. Azure 데이터 탐색기는 HTTPs (443) 및 TDS (1433) 라는 두 개의 끝점을 사용 하 여 액세스할 수 있습니다. 클러스터의 관리, 모니터링 및 적절 한 작업을 수행 하기 위해 이러한 끝점에 대 한 액세스를 허용 하도록 다음 NSG 규칙을 구성 해야 합니다. 추가 규칙은 보안 지침에 따라 달라 집니다.

#### <a name="inbound-nsg-configuration"></a>인바운드 NSG 구성

| **사용**   | **From**   | **수행할 작업**   | **프로토콜**   |
| --- | --- | --- | --- |
| 관리  |[Adx management addresses](#azure-data-explorer-management-ip-addresses)/AzureDataExplorerManagement (servicetag) | ADX 서브넷: 443  | TCP  |
| 상태 모니터링  | [ADX 상태 모니터링 주소](#health-monitoring-addresses)  | ADX 서브넷: 443  | TCP  |
| ADX 내부 통신  | ADX 서브넷: 모든 포트  | ADX 서브넷: 모든 포트  | 모두  |
| Azure 부하 분산 장치 인바운드 허용 (상태 프로브)  | AzureLoadBalancer  | ADX 서브넷: 80443  | TCP  |

#### <a name="outbound-nsg-configuration"></a>아웃 바운드 NSG 구성

| **사용**   | **From**   | **수행할 작업**   | **프로토콜**   |
| --- | --- | --- | --- |
| Azure Storage에 대한 종속성  | ADX 서브넷  | 저장소: 443  | TCP  |
| Azure Data Lake에 대 한 종속성  | ADX 서브넷  | AzureDataLake: 443  | TCP  |
| EventHub 수집 및 서비스 모니터링  | ADX 서브넷  | EventHub: 443, 5671  | TCP  |
| 메트릭 게시  | ADX 서브넷  | AzureMonitor: 443 | TCP  |
| Active Directory (해당 하는 경우) | ADX 서브넷 | AzureActiveDirectory: 443 | TCP |
| 인증 기관 | ADX 서브넷 | 인터넷: 80 | TCP |
| 내부 통신  | ADX 서브넷  | ADX 서브넷: 모든 포트  | 모두  |
| `sql\_request`및 플러그 인에 사용 되는 포트 `http\_request`  | ADX 서브넷  | 인터넷: 사용자 지정  | TCP  |

### <a name="relevant-ip-addresses"></a>관련 IP 주소

#### <a name="azure-data-explorer-management-ip-addresses"></a>Azure 데이터 탐색기 관리 IP 주소

> [!NOTE]
> 이후 배포의 경우 AzureDataExplorer 서비스 태그를 사용 합니다.

| 지역 | 주소 |
| --- | --- |
| 오스트레일리아 중부 | 20.37.26.134 |
| 오스트레일리아 Central2 | 20.39.99.177 |
| 오스트레일리아 동부 | 40.82.217.84 |
| 오스트레일리아 남동부 | 20.40.161.39 |
| BrazilSouth | 191.233.25.183 |
| 캐나다 중부 | 40.82.188.208 |
| 캐나다 동부 | 40.80.255.12 |
| 인도 중부 | 40.81.249.251, 104.211.98.159 |
| 미국 중부 | 40.67.188.68 |
| 미국 중부 EUAP | 40.89.56.69 |
| 중국 동부 2 | 139.217.184.92 |
| 중국 북부 2 | 139.217.60.6 |
| 동아시아 | 20.189.74.103 |
| 미국 동부 | 52.224.146.56 |
| 미국 동부2 | 52.232.230.201 |
| 동부 미국 EUAP | 52.253.226.110 |
| 프랑스 중부 | 40.66.57.91 |
| 프랑스 남부 | 40.82.236.24 |
| 일본 동부 | 20.43.89.90 |
| 일본 서부 | 40.81.184.86 |
| 한국 중부 | 40.82.156.149 |
| 한국 남부 | 40.80.234.9 |
| 미국 중북부 | 40.81.45.254 |
| 북유럽 | 52.142.91.221 |
| 남아프리카 북부 | 102.133.129.138 |
| 남아프리카 공화국 서부 | 102.133.0.97 |
| 미국 중남부 | 20.45.3.60 |
| 동남 아시아 | 40.119.203.252 |
| 인도 남부 | 40.81.72.110, 104.211.224.189 |
| 영국 남부 | 40.81.154.254 |
| 영국 서부 | 40.81.122.39 |
| 미국 국방부 중부 | 52.182.33.66 |
| 미국 국방부 동부 | 52.181.33.69 |
| USGov 애리조나 | 52.244.33.193 |
| USGov 텍사스 | 52.243.157.34 |
| USGov 버지니아 | 52.227.228.88 |
| 미국 중서부 | 52.159.55.120 |
| 서유럽 | 51.145.176.215 |
| 인도 서부 | 40.81.88.112, 104.211.160.120 |
| 미국 서부 | 13.64.38.225 |
| 미국 서부2 | 40.90.219.23 |

#### <a name="health-monitoring-addresses"></a>상태 모니터링 주소

| 지역 | 주소 |
| --- | --- |
| 오스트레일리아 중부 | 191.239.64.128 |
| 오스트레일리아 중부 2 | 191.239.64.128 |
| 오스트레일리아 동부 | 191.239.64.128 |
| 오스트레일리아 남동부 | 191.239.160.47 |
| 브라질 남부 | 23.98.145.105 |
| 캐나다 중부 | 168.61.212.201 |
| 캐나다 동부 | 168.61.212.201, 23.101.115.123 |
| 인도 중부 | 23.99.5.162 |
| 미국 중부 | 168.61.212.201, 23.101.115.123 |
| 미국 중부 EUAP | 168.61.212.201, 23.101.115.123 |
| 중국 동부 2 | 40.73.96.39 |
| 중국 북부 2 | 40.73.33.105 |
| 동아시아 | 168.63.212.33 |
| 미국 동부 | 137.116.81.189, 52.249.253.174 |
| 미국 동부 2 | 137.116.81.189, 104.46.110.170 |
| 미국 동부 2 EUAP | 137.116.81.189, 104.46.110.170 |
| 프랑스 중부 | 23.97.212.5 |
| 프랑스 남부 | 23.97.212.5 |
| 일본 동부 | 138.91.19.129 |
| 일본 서부 | 138.91.19.129 |
| 한국 중부 | 138.91.19.129 |
| 한국 남부 | 138.91.19.129 |
| 미국 중북부 | 23.96.212.108 |
| 북유럽 | 191.235.212.69, 40.127.194.147 |
| 남아프리카 북부 | 104.211.224.189 |
| 남아프리카 공화국 서부 | 104.211.224.189 |
| 미국 중남부 | 23.98.145.105, 104.215.116.88 |
| 인도 남부 | 23.99.5.162 |
| 동남 아시아 | 168.63.173.234 |
| 영국 남부 | 23.97.212.5 |
| 영국 서부 | 23.97.212.5 |
| 미국 국방부 중부 | 52.238.116.34 |
| 미국 국방부 동부 | 52.238.116.34 |
| USGov 애리조나 | 52.244.48.35 |
| USGov 텍사스 | 52.238.116.34 |
| USGov 버지니아 | 23.97.0.26 |
| 미국 중서부 | 168.61.212.201, 23.101.115.123 |
| 서유럽 | 23.97.212.5, 213.199.136.176 |
| 인도 서부 | 23.99.5.162 |
| 미국 서부 | 23.99.5.162, 13.88.13.50 |
| 미국 서부 2 | 23.99.5.162, 104.210.32.14, 52.183.35.124 |

## <a name="disable-access-to-azure-data-explorer-from-the-public-ip"></a>공용 IP에서 Azure 데이터 탐색기에 대 한 액세스 사용 안 함

공용 IP 주소를 통해 Azure 데이터 탐색기에 대 한 액세스를 완전히 사용 하지 않도록 설정 하려면 NSG에서 다른 인바운드 규칙을 만듭니다. 이 규칙은 낮은 [우선 순위](/azure/virtual-network/security-overview#security-rules) (높은 숫자)를 가져야 합니다. 

| **사용**   | **원본** | **원본 서비스 태그** | **원본 포트 범위**  | **대상** | **대상 포트 범위** | **프로토콜** | **작업** | **우선 순위** |
| ---   | --- | --- | ---  | --- | --- | --- | --- | --- |
| 인터넷에서 액세스 사용 안 함 | 서비스 태그 | 인터넷 | *  | VirtualNetwork | * | 모두 | 거부 | 위의 규칙 보다 높은 값 |

이 규칙을 사용 하면 다음 DNS 레코드 (각 서비스의 개인 IP로 매핑됨)를 통해서만 Azure 데이터 탐색기 클러스터에 연결할 수 있습니다.
* `private-[clustername].[geo-region].kusto.windows.net` 엔진
* `private-ingest-[clustername].[geo-region].kusto.windows.net` (데이터 관리)

## <a name="expressroute-setup"></a>Express 경로 설정

Express 경로를 사용 하 여 온-프레미스 네트워크를 Azure Virtual Network에 연결 합니다. 일반적인 설정은 BGP (Border Gateway Protocol) 세션을 통해 기본 경로 (0.0.0.0/0)를 보급 하는 것입니다. 이렇게 하면 트래픽을 삭제 하 여 아웃 바운드 흐름이 중단 될 수 있는 고객의 온-프레미스 네트워크에 전달 될 Virtual Network에서 들어오는 트래픽이 발생 합니다. 이 기본값을 극복 하기 위해 [UDR (사용자 정의 경로](/azure/virtual-network/virtual-networks-udr-overview#user-defined) ) (0.0.0.0/0)을 구성할 수 있으며 다음 홉이 *인터넷* 입니다. UDR이 BGP 보다 우선적으로 적용 되므로 트래픽은 인터넷으로 전송 됩니다.

## <a name="securing-outbound-traffic-with-firewall"></a>방화벽을 사용 하 여 아웃 바운드 트래픽 보안

[Azure 방화벽](/azure/firewall/overview) 또는 가상 어플라이언스를 사용 하 여 도메인 이름을 제한 하는 아웃 바운드 트래픽을 보호 하려면 다음 FQDN (정규화 된 도메인 이름)이 방화벽에서 허용 되어야 합니다.

```
prod.warmpath.msftcloudes.com:443
gcs.prod.monitoring.core.windows.net:443
production.diagnostics.monitoring.core.windows.net:443
graph.windows.net:443
*.update.microsoft.com:443
shavamanifestcdnprod1.azureedge.net:443
login.live.com:443
wdcp.microsoft.com:443
login.microsoftonline.com:443
azureprofilerfrontdoor.cloudapp.net:443
*.core.windows.net:443
*.servicebus.windows.net:443,5671
shoebox2.metrics.nsatc.net:443
prod-dsts.dsts.core.windows.net:443
ocsp.msocsp.com:80
*.windowsupdate.com:80
ocsp.digicert.com:80
go.microsoft.com:80
dmd.metaservices.microsoft.com:80
www.msftconnecttest.com:80
crl.microsoft.com:80
www.microsoft.com:80
adl.windows.com:80
crl3.digicert.com:80
```

> [!NOTE]
> [Azure 방화벽](/azure/firewall/overview)을 사용 하는 경우 다음 속성을 사용 하 여 **네트워크 규칙** 을 추가 합니다. <br>
> **프로토콜**: TCP <br> **원본 유형**: IP 주소 <br> **원본**: * <br> **서비스 태그**: azuremonitor <br> **대상 포트**: 443

또한 비대칭 경로 문제를 방지 하기 위해 다음 홉 *인터넷* 을 사용 하는 [관리 주소](#azure-data-explorer-management-ip-addresses) 및 [상태 모니터링 주소](#health-monitoring-addresses) 를 사용 하 여 서브넷에서 [경로 테이블](/azure/virtual-network/virtual-networks-udr-overview) 을 정의 해야 합니다.

예를 들어 **미국 서 부** 지역의 경우 다음 udrs를 정의 해야 합니다.

| Name | 주소 접두사 | 다음 홉 |
| --- | --- | --- |
| ADX_Management | 13.64.38.225/32 | 인터넷 |
| ADX_Monitoring | 23.99.5.162/32 | 인터넷 |

## <a name="deploy-azure-data-explorer-cluster-into-your-vnet-using-an-azure-resource-manager-template"></a>Azure Resource Manager 템플릿을 사용 하 여 VNet에 Azure 데이터 탐색기 클러스터 배포

Azure 데이터 탐색기 클러스터를 가상 네트워크에 배포 하려면 VNet Azure Resource Manager 템플릿에 [azure 데이터 탐색기 클러스터 배포](https://azure.microsoft.com/resources/templates/101-kusto-vnet/) 를 사용 합니다.

이 템플릿은 클러스터, 가상 네트워크, 서브넷, 네트워크 보안 그룹 및 공용 IP 주소를 만듭니다.
