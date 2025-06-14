# 로드 밸런싱 (Load Balancing) 이란?

로드 밸런싱은 **애플리케이션을 지원하는 여러 서버(리소스 풀) 전체에 걸쳐 네트워크 트래픽을 효율적으로 분산**하는 기술입니다. 이를 통해 특정 서버에 부하가 집중되는 것을 막고, 시스템 전체의 **가용성(availability), 신뢰성(reliability), 그리고 응답 속도(response time)를 향상**시키는 핵심적인 역할을 합니다.

## 서버 확장 방식: Scale Up vs Scale Out

서버의 처리 용량을 늘리는 방법은 크게 두 가지로 나뉩니다.

### 1. Scale Up (수직 확장)

기존 서버 자체의 성능을 강화하는 방식입니다. (예: CPU, RAM, 디스크 성능 업그레이드)

* **✅ 장점:**
    * 인프라 구조가 비교적 단순하게 유지됩니다 (서버 관리 대수 불변).
    * 기존 소프트웨어 변경 없이 하드웨어 업그레이드만으로 성능 향상을 기대할 수 있습니다.
* **⚠️ 단점:**
    * **물리적인 한계:** 서버 한 대가 가질 수 있는 리소스에는 명확한 한계가 존재합니다.
    * **단일 장애 지점 (SPOF):** 해당 서버에 장애가 발생하면 전체 서비스가 중단될 위험이 있습니다.
    * **비용:** 고성능 하드웨어는 가격이 기하급수적으로 증가하는 경향이 있습니다.

### 2. Scale Out (수평 확장)

여러 대의 서버를 추가하여 처리 능력을 늘리고, 로드 밸런서를 통해 들어오는 요청을 분산시키는 방식입니다.

* **✅ 장점:**
    * **높은 확장성:** 필요에 따라 서버를 계속 추가하여 거의 무한대에 가깝게 시스템을 확장할 수 있습니다.
    * **내결함성 (Fault Tolerance):** 일부 서버에 장애가 발생해도 다른 서버들이 서비스를 계속 제공할 수 있어 장애 격리에 유리합니다.
    * **유연성:** 클라우드 환경이나 컨테이너 기반의 MSA(Microservices Architecture)와 같은 현대적인 아키텍처와 잘 어울립니다.
* **⚠️ 단점:**
    * **로드 밸런서 필수:** 트래픽 분산을 위해 로드 밸런서 도입 및 관리가 필수적입니다. (로드 밸런서 자체의 이중화(HA) 구성도 고려해야 합니다.)
    * **상태 관리 복잡성:** 여러 서버에서 사용자 세션(Session)을 일관되게 유지하거나 데이터를 동기화하는 등 상태 관리가 복잡해질 수 있습니다.
    * **네트워크 복잡도 증가:** 서버 간 통신 및 로드 밸런서 설정 등으로 인해 네트워크 구성이 더 복잡해질 수 있습니다.

## 주요 로드 밸런싱 기법 (알고리즘)

로드 밸런서는 다양한 알고리즘을 사용하여 서버에 트래픽을 분배합니다. 다음은 대표적인 기법들입니다.

### 1. 라운드 로빈 (Round Robin)

가장 기본적이고 단순하며 널리 사용되는 방식입니다. 들어오는 요청을 서버 목록에 순서대로 하나씩 분배합니다.

* **특징:**
    * 구현이 매우 간단합니다. (Nginx의 기본 분배 방식)
    * 모든 서버의 성능과 처리 능력이 유사하다고 가정할 때 효과적입니다.
    * 서버가 클라이언트의 이전 요청 상태를 기억할 필요가 없는 (Stateless) 서비스에 적합합니다.
* **단점:**
    * 서버 간의 실제 성능 차이나 현재 처리 중인 부하 상태를 고려하지 않습니다. 특정 서버가 느려지더라도 동일한 수의 요청을 받게 됩니다.
* **Nginx 예시:**
    ```nginx
    upstream backend {
        server server1;
        server server2;
        server server3;
    }
    ```

### 2. 가중 라운드 로빈 (Weighted Round Robin)

각 서버에 가중치(Weight)를 설정하여, 설정된 가중치 비율에 따라 요청을 분배합니다.

* **특징:**
    * 서버의 성능 차이를 고려하여 트래픽을 분배할 수 있습니다. (예: 고사양 서버에 더 높은 가중치 부여)
    * 서버 리소스를 보다 효율적으로 활용할 수 있습니다.
* **Nginx 예시:**
    ```nginx
    upstream backend {
        server server1 weight=5; # server1이 5/8 비율로 요청 처리
        server server2 weight=2; # server2가 2/8 비율로 요청 처리
        server server3 weight=1; # server3이 1/8 비율로 요청 처리
    }
    ```

### 3. 최소 연결 (Least Connections)

현재 활성 연결(Active Connection) 수가 가장 적은 서버에게 다음 요청을 보내는 방식입니다.

* **특징:**
    * 각 요청의 처리 시간이 다르거나, 세션이 길게 유지되는 경우 서버의 부하를 균등하게 유지하는 데 유리합니다.
    * 실시간 트래픽 부하를 고려한 분산이 가능합니다.
* **단점:**
    * 활성 연결 수만을 기준으로 하므로, 서버의 실제 CPU나 메모리 사용량 같은 리소스 상태를 직접적으로 반영하지는 못할 수 있습니다.
* **Nginx 예시:**
    ```nginx
    upstream backend {
        least_conn;
        server server1;
        server server2;
        server server3;
    }
    ```

### 4. IP 해시 (IP Hash)

클라이언트의 IP 주소를 해싱(Hashing)하여 특정 서버로만 요청을 보내는 방식입니다.

* **특징:**
    * **세션 지속성 (Session Persistence / Sticky Session):** 특정 클라이언트의 요청은 항상 동일한 서버로 전달되므로, 사용자의 세션 정보를 해당 서버에서 유지해야 하는 경우 유용합니다. (예: 로그인 상태, 장바구니 정보 등)
* **단점:**
    * 특정 IP 대역에서 많은 요청이 들어올 경우, 해당 요청이 매핑된 서버에만 부하가 집중될 수 있습니다 (IP 편중 현상).
    * 프록시 서버나 NAT 환경에서는 여러 클라이언트가 동일한 IP로 인식되어 부하 분산 효과가 떨어질 수 있습니다.
* **Nginx 예시:**
    ```nginx
    upstream backend {
        ip_hash;
        server server1;
        server server2;
        server server3;
    }
    ```

---

**[보강 내용 제안]**

* **다양한 로드 밸런싱 기법:** 위에서 소개된 기법 외에도 **Least Response Time** (응답 시간이 가장 빠른 서버 선택), **Source IP Hash** (Source IP와 Port 기반 해싱), **URL Hash** (요청 URL 기반 해싱) 등 다양한 알고리즘이 존재합니다. 서비스의 특징과 요구사항에 맞는 최적의 알고리즘을 선택하는 것이 중요합니다.
* **L4 vs L7 로드 밸런싱:** 로드 밸런서는 작동하는 네트워크 계층에 따라 크게 **L4 로드 밸런서**와 **L7 로드 밸런서**로 나뉩니다.
    * **L4 로드 밸런서:** Transport 계층(OSI 4계층)에서 작동하며, IP 주소와 Port 번호를 기반으로 트래픽을 분산합니다. 속도가 빠르지만, 패킷 내용을 보지 못해 정교한 분산 제어는 어렵습니다. (예: TCP, UDP)
    * **L7 로드 밸런서:** Application 계층(OSI 7계층)에서 작동하며, HTTP 헤더, 쿠키, URL 등 애플리케이션 레벨의 정보를 바탕으로 트래픽을 분산합니다. 더 지능적이고 세밀한 부하 분산 규칙 적용이 가능합니다. (예: HTTP, HTTPS)
* **하드웨어 vs 소프트웨어 로드 밸런서:** 로드 밸런서는 전용 하드웨어 장비 형태로 제공되기도 하고(예: F5 BIG-IP), Nginx, HAProxy, AWS ELB 등과 같이 소프트웨어 형태로 구현될 수도 있습니다.

**[결론 요약]**

로드 밸런싱은 단순히 트래픽을 나누는 것을 넘어, **애플리케이션의 확장성, 안정성, 성능을 보장**하는 현대적인 서비스 아키텍처의 필수 구성 요소입니다. Scale Out 전략과 함께 사용될 때 그 효과가 극대화되며, 서비스의 특성에 맞는 적절한 로드 밸런싱 기법과 설정을 선택하는 것이 중요합니다.