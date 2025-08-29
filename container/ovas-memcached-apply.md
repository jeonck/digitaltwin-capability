NVIDIA OVAS(Omniverse Application Streaming)의 배포 가이드에서 **memcached**는 쉐이더 캐싱을 통해 애플리케이션(예: USD Viewer)의 시작 지연 시간을 줄이고 성능을 최적화하는 데 중요한 역할을 합니다. 아래는 memcached 측면에서 NVIDIA OVAS 배포 가이드에 기반한 상세 설명입니다. 이 정보는 NVIDIA 공식 문서(특히 Kit App Streaming 관련 문서)와 관련 웹 결과를 바탕으로 구성되었습니다.[](https://docs.omniverse.nvidia.com/ovas/latest/architecture/shader-cache.html)[](https://docs.omniverse.nvidia.com/ovas/latest/deployments/infra/installation.html)[](https://docs.omniverse.nvidia.com/ovas/latest/developer/kit-ovas-dev.html)

### 1. memcached의 역할
- **목적**: memcached는 분산 메모리 기반 키-값 스토어로, 컴파일된 쉐이더 데이터를 저장하여 USD Viewer와 같은 Omniverse Kit 기반 애플리케이션의 초기 컴파일 지연(5~10분)을 줄입니다.
- **기능**: 동일 Kubernetes 클러스터 내 또는 여러 노드에서 쉐이더 데이터를 공유하여, 컨테이너가 기동할 때 캐시된 데이터를 빠르게 로드하도록 지원.
- **성능 이점**: 쉐이더 캐싱을 통해 애플리케이션 시작 시간을 수초로 단축하며, GPU 리소스 사용 효율성을 높입니다.

### 2. memcached 배포 가이드
NVIDIA OVAS 배포 가이드에서 memcached 설정 및 통합에 대한 주요 단계를 아래와 같이 정리할 수 있습니다.

#### (1) memcached 설치
- **Kubernetes 클러스터 내 배포**:
  - memcached는 OVAS 애플리케이션 컨테이너(예: USD Viewer)와 동일한 Kubernetes 네임스페이스(예: `omni-streaming`)에 설치해야 합니다.
  - NVIDIA는 Bitnami의 Helm 차트를 사용하여 memcached를 배포할 것을 권장합니다. 예시 명령어:
    ```bash
    helm upgrade --install \
      -n omni-streaming --create-namespace \
      -f helm/memcached/values.yml \
      --repo https://charts.bitnami.com/bitnami \
      memcached-service-r3 memcached
    ```
    - 이 명령은 `omni-streaming` 네임스페이스에 memcached를 설치하며, `values.yml` 파일로 사용자 정의 설정을 적용합니다.
    - 설치 결과 예시:
      ```
      NAME: memcached-service-r3
      LAST DEPLOYED: Tue Apr 23 20:05:21 2024
      NAMESPACE: omni-streaming
      STATUS: deployed
      REVISION: 1
      CHART NAME: memcached
      CHART VERSION: 7.0.3
      APP VERSION: 1.6.26
      ```

- **노드 선택**:
  - memcached 서비스 팟(Pod)은 특정 워커 노드 또는 노드 그룹에서 실행되도록 설정해야 합니다. 이를 위해 `values.yml` 파일에서 노드 레이블(예: `NodeGroup=system`)을 지정:
    ```yaml
    nodeSelector:
      NodeGroup: system
    ```
  - 노드 레이블은 memcached가 올바른 워커 노드에서 실행되도록 보장합니다.

#### (2) memcached 설정
- **환경 변수 설정**:
  - Omniverse Kit 애플리케이션은 memcached 인스턴스의 URL을 참조하도록 설정해야 합니다. 이를 위해 `AUTO_LOAD_DRIVER_CACHE_WRAPPER` 환경 변수를 사용:
    ```bash
    export AUTO_LOAD_DRIVER_CACHE_WRAPPER=<memcached_url>
    ```
    - `<memcached_url>`은 memcached 서비스의 엔드포인트(예: `memcached-service-r3.omni-streaming.svc.cluster.local:11211`)입니다.
  - Kit 106 이상 버전에서는 이 환경 변수가 자동으로 설정되므로 별도 설정이 필요 없을 수 있습니다.[](https://docs.omniverse.nvidia.com/ovas/latest/developer/kit-ovas-dev.html)
  - Kit 105.1의 경우, `omni.hssc` 확장을 로드해야 공유 쉐이더 캐싱이 활성화됩니다.

- **values.yml 구성**:
  - `helm/memcached/values.yml` 파일에서 memcached의 메모리 할당, 포트, 네트워크 설정 등을 사용자 정의:
    ```yaml
    memcached:
      replicaCount: 1
      resources:
        requests:
          memory: "512Mi"
          cpu: "500m"
        limits:
          memory: "1Gi"
          cpu: "1"
      service:
        type: ClusterIP
        port: 11211
    ```
    - 메모리와 CPU 리소스는 쉐이더 데이터 크기와 예상 워크로드에 따라 조정해야 합니다.

#### (3) 쉐이더 캐싱 활성화
- **공유 쉐이더 캐시 설정**:
  - OVAS는 memcached를 통해 쉔이더 데이터를 캐싱하도록 설정됩니다. 컨테이너가 기동하면:
    1. 로컬 캐시 확인 후, 없으면 memcached에 쿼리.
    2. 캐시 히트 시 데이터를 로드, 캐시 미스 시 쉐이더를 컴파일하여 memcached에 저장.
  - `AUTO_ENABLE_DRIVER_SHADER_CACHE_WRAPPER` 환경 변수를 설정하여 자동 캐싱을 활성화(권장). 이 변수를 컨테이너 시작 스크립트에 하드코딩하는 것은 권장되지 않음(변경 시 컨테이너 재생성 필요).[](https://docs.omniverse.nvidia.com/ovas/latest/developer/kit-ovas-dev.html)

#### (4) 네트워크 및 보안
- **네트워크 접근**:
  - memcached는 클러스터 내부에서만 접근 가능하도록 `ClusterIP` 서비스로 설정하는 것이 일반적입니다.
  - 외부 접근이 필요한 경우, 네트워크 정책(NetworkPolicy)을 설정하여 보안을 강화.
- **보안 설정**:
  - memcached는 기본적으로 인증 메커니즘이 없으므로, Kubernetes 네임스페이스와 네트워크 정책으로 접근을 제한.
  - TLS 암호화를 위해 추가 설정이 필요할 수 있음(예: AWS NLB와 통합 시 TLS 인증서 사용).[](https://docs.omniverse.nvidia.com/ovas/latest/deployments/infra/requirements.html)

#### (5) 모니터링 및 유지보수
- **모니터링**:
  - memcached의 메모리 사용량, 히트/미스 비율, 캐시 제거(eviction) 등을 모니터링하여 성능을 최적화.
  - Datadog와 같은 툴을 사용해 메트릭을 수집할 수 있음(예: `memcache.slabs.*`, `memcache.items.*`).[](https://docs.datadoghq.com/integrations/memcached/)
- **캐시 관리**:
  - 쉐이더 데이터가 많아지면 memcached의 메모리 한계로 인해 오래된 데이터가 제거될 수 있으므로, 메모리 크기를 적절히 설정.
  - GPU 드라이버 또는 Kit 버전 업데이트 시 캐시 무효화 가능성이 있으므로, 주기적인 캐시 갱신 계획 필요.

### 3. 배포 시 고려 사항
- **리소스 요구 사항**:
  - memcached는 쉐이더 데이터의 크기와 요청 빈도에 따라 메모리와 네트워크 대역폭이 많이 필요할 수 있음. 적절한 리소스 할당(예: 512Mi~1Gi 메모리)을 권장.[](https://docs.omniverse.nvidia.com/ovas/latest/deployments/infra/requirements.html)
- **클러스터 내 통합**:
  - memcached는 OVAS 애플리케이션과 동일 네임스페이스(예: `omni-streaming`)에 배포되어야 하며, 서비스 디스커버리를 통해 접근 가능해야 함.
- **확장성**:
  - 다중 memcached 인스턴스를 배포하여 고가용성과 부하 분산을 지원할 수 있음. Helm 차트에서 `replicaCount`를 조정.
- **드라이버 호환성**:
  - 모든 GPU 워커 노드는 지원되는 NVIDIA 드라이버 버전을 사용해야 하며, 쉐이더 캐시가 GPU 아키텍처와 호환되어야 함.[](https://docs.omniverse.nvidia.com/ovas/latest/deployments/infra/requirements.html)

### 4. 예시 워크플로우
1. **memcached 배포**:
   - Helm 차트를 사용하여 `omni-streaming` 네임스페이스에 memcached 설치.
2. **USD Viewer 컨테이너 설정**:
   - 컨테이너의 환경 변수에 memcached URL 설정(예: `AUTO_LOAD_DRIVER_CACHE_WRAPPER`).
3. **쉐이더 캐싱**:
   - 첫 번째 USD Viewer 컨테이너가 쉐이더를 컴파일하여 memcached에 저장.
   - 이후 컨테이너는 memcached에서 캐시된 쉐이더를 로드하여 빠르게 기동.
4. **스트리밍 시작**:
   - 쉐이더 로드 후, USD Viewer는 OpenUSD와 RTX 렌더링을 통해 클라이언트로 스트리밍.

### 5. 추가 참고 자료
- **NVIDIA 공식 문서**:
  - [Kit App Streaming - Installation](https://docs.omniverse.nvidia.com)[](https://docs.omniverse.nvidia.com/ovas/latest/deployments/infra/installation.html)
  - [Shared Shader Caching](https://docs.omniverse.nvidia.com)[](https://docs.omniverse.nvidia.com/ovas/latest/architecture/shader-cache.html)
  - [Kit Development](https://docs.omniverse.nvidia.com)[](https://docs.omniverse.nvidia.com/ovas/latest/developer/kit-ovas-dev.html)
- **권장 사항**: 최신 배포 가이드는 NVIDIA Developer 포럼(developer.nvidia.com) 또는 NVIDIA OVAS 문서에서 확인하세요. Omniverse Launcher는 2025년 10월 1일자로 지원 종료되므로, 최신 가이드를 참고해야 합니다.

### 6. 요약
NVIDIA OVAS 배포 가이드에서 memcached는 쉐이더 캐싱을 위해 필수적인 구성 요소로, 동일 Kubernetes 클러스터 내에서 USD Viewer와 같은 컨테이너의 성능을 최적화합니다. Helm 차트를 통해 memcached를 설치하고, 환경 변수를 설정하여 쉐이더 캐싱을 활성화하며, 네트워크 및 리소스 설정을 통해 안정성과 확장성을 보장합니다.

