리눅스 컨테이너 기술은 여러 계층으로 구성된 구조를 통해 효율적으로 동작하며, 각 구성 요소는 서로 연결되어 있습니다. 이 구조는 리눅스 커널의 기능을 기반으로 하여, 프로세스 격리와 자원 관리를 가능하게 합니다. 다음은 이러한 계층 구조의 핵심 요소들과 그 연계성을 설명합니다.

## **1. 리눅스 커널**
리눅스 커널은 컨테이너 기술의 가장 하위 계층으로, 모든 기능의 기반이 됩니다. 커널은 시스템 자원을 관리하고, 네임스페이스와 cgroups와 같은 기능을 통해 프로세스의 격리와 자원 관리를 지원합니다. 이러한 기능들은 컨테이너가 독립적으로 실행될 수 있도록 하는 핵심 요소입니다[1][2].

## **2. 네임스페이스 (Namespaces)**
네임스페이스는 리눅스 커널의 기능으로, 각 컨테이너가 독립적인 환경에서 실행될 수 있도록 합니다. 이 기능은 다음과 같은 격리 기능을 제공합니다:

- **프로세스 격리**: 각 컨테이너는 독립적인 프로세스 ID 공간을 가지며, 서로의 프로세스에 접근할 수 없습니다.
  
- **네트워크 격리**: 각 컨테이너는 독립적인 네트워크 스택을 가지며, IP 주소와 포트 번호가 격리됩니다[1][6].

이러한 네임스페이스 기능은 컨테이너가 서로 영향을 미치지 않고 독립적으로 동작할 수 있게 해줍니다.

## **3. 컨트롤 그룹 (cgroups)**
cgroups는 리눅스 커널의 기능으로, 프로세스 그룹의 자원 사용을 제한하고 모니터링합니다. 이를 통해 다음과 같은 기능을 제공합니다:

- **자원 제한**: CPU, 메모리, 네트워크 대역폭 등을 제한하여 각 컨테이너가 할당된 자원 내에서만 동작하도록 합니다.
  
- **안정성**: 특정 컨테이너가 자원을 과다 사용하지 않도록 제어하여 전체 시스템의 안정성을 유지합니다[1][6].

cgroups는 컨테이너의 자원 사용을 관리하여, 시스템의 성능을 최적화하고 안정성을 높이는 데 기여합니다.

## **4. 컨테이너 이미지 (Container Image)**
컨테이너 이미지는 애플리케이션 실행에 필요한 모든 파일과 설정을 포함하는 불변의 파일 시스템입니다. 이 이미지는 다음과 같은 역할을 합니다:

- **이식성**: 동일한 이미지를 사용하여 다양한 환경에서 애플리케이션을 실행할 수 있습니다.
  
- **표준화**: 컨테이너 실행 환경을 표준화하여 개발과 배포의 일관성을 제공합니다[1][2].

컨테이너 이미지는 네임스페이스와 cgroups의 기능을 활용하여 독립적인 실행 환경을 제공하며, 이를 통해 개발자는 다양한 환경에서 일관된 애플리케이션 실행을 보장할 수 있습니다.

## **5. 컨테이너 런타임 (Container Runtime)**
컨테이너 런타임은 컨테이너 이미지를 기반으로 실제 컨테이너를 생성하고 실행하는 소프트웨어입니다. 이 런타임은 다음과 같은 기능을 수행합니다:

- **컨테이너 실행**: 컨테이너를 생성하고 실행하는 기능을 제공합니다.
  
- **자원 관리**: cgroups와 네임스페이스를 설정하여 컨테이너의 자원 할당 및 생명 주기를 관리합니다[1][2].

컨테이너 런타임은 네임스페이스와 cgroups의 기능을 조합하여 실제 컨테이너를 실행하며, 이를 통해 애플리케이션의 성능과 안정성을 보장합니다.

## **6. 보안 기술 (SELinux 등)**
보안 기술은 컨테이너 간의 보안 격리를 강화합니다. SELinux와 같은 보안 모듈은 다음과 같은 기능을 제공합니다:

- **정책 기반 보안**: 역할 기반 접근 제어(RBAC)와 MAC 정책을 통해 컨테이너의 안전한 실행 환경을 보장합니다.
  
- **위험 감소**: 컨테이너 간의 상호작용을 제한하여 보안 위험을 줄입니다[1][2].

보안 기술은 컨테이너 이미지와 런타임의 안전성을 높여, 전체 시스템의 보안을 강화하는 데 중요한 역할을 합니다.

## **결론**
리눅스 컨테이너 기술은 리눅스 커널을 중심으로 한 계층 구조로 구성되어 있으며, 각 요소는 서로 연결되어 동작합니다. 네임스페이스와 cgroups는 프로세스의 격리와 자원 관리를 담당하고, 컨테이너 이미지는 이식성과 표준화를 제공합니다. 컨테이너 런타임은 이러한 요소들을 조합하여 실제 컨테이너를 실행하며, 보안 기술은 전체 시스템의 안전성을 강화합니다. 이러한 계층적 구조는 리눅스 컨테이너가 효율적이고 안전하게 동작할 수 있도록 지원합니다.
[1] https://computing-jhson.tistory.com/98
[2] https://www.44bits.io/ko/keyword/linux-container
[3] http://www.opennaru.com/cloud/container/
[4] https://terianp.tistory.com/153
[5] https://www.redhat.com/ko/topics/containers
[6] https://velog.io/@bunny/docker-cgroup%EA%B3%BC-%EB%84%A4%EC%9E%84%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4
[7] https://www.redhat.com/ko/topics/containers/whats-a-linux-container
[8] https://dd-developer.tistory.com/64
[9] https://m.blog.naver.com/shakey7/221600166205
[10] https://www.redhat.com/ko/topics/containers/what-is-docker
[11] https://gggggeun.tistory.com/60
[12] https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/
[13] https://jiheon95.tistory.com/56
[14] https://heewon0704.tistory.com/30
[15] https://hero-space.tistory.com/entry/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B8%B0%EC%88%A0%EC%9D%84-%EC%9C%84%ED%95%9C-namespace-cgroup-3%EB%B6%84%EC%9A%94%EC%95%BD
[16] https://anweh.tistory.com/67
[17] https://trylhc.tistory.com/entry/docker-and-container-runtime
[18] https://blog.naver.com/hpe_hrd/221827042082?viewType=pc
[19] https://www.reddit.com/r/devops/comments/a67g3z/can_someone_please_explain_me_what_is_the/?tl=ko
[20] https://adimkim900.tistory.com/41
[21] https://blog.naver.com/brainzsquare/223411921857
[22] https://blog.naver.com/alice_k106/221179849700?viewType=pc
[23] https://aws.github.io/aws-eks-best-practices/ko/security/docs/image/
[24] https://rockplace.tistory.com/2
[25] https://velog.io/@karldev/Docker%EC%9D%98-Image%EC%99%80-Container%EB%A5%BC-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90
[26] https://csj000714.tistory.com/655
[27] https://bbang-jun.tistory.com/18
[28] https://loosie.tistory.com/881
[29] https://puzzle-puzzle.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88LXC
[30] https://cwal.tistory.com/31
[31] https://dev-chicken.tistory.com/16
[32] https://velog.io/@becooq81/Docker%EC%97%90-%EC%82%AC%EC%9A%A9%EB%90%98%EB%8A%94-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B0%9C%EB%85%90%EA%B3%BC-Docker-%EA%B8%B0%EB%B3%B8-%EC%82%AC%EC%9A%A9%EB%B2%95
[33] https://aws.amazon.com/ko/compare/the-difference-between-docker-images-and-containers/
[34] https://devbksheen.tistory.com/entry/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%9D%98-%EC%9D%B4%ED%95%B433-%EB%8F%84%EC%BB%A4%EC%99%80-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EC%9D%98-%EA%B4%80%EA%B3%84
[35] https://binux.tistory.com/174
[36] https://www.samsungsds.com/kr/insights/docker.html
[37] https://osckorea.tistory.com/189
[38] https://ubuntu2304.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-docker-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B0%80%EC%83%81%ED%99%94-%EA%B8%B0%EC%88%A0%EB%A1%9C-%EC%97%B4%EC%96%B4%EA%B0%80%EB%8A%94-%EB%AF%B8%EB%9E%98
[39] https://hoon93.tistory.com/48
[40] https://www.reddit.com/r/docker/comments/97utrp/creating_a_docker_image_with_a_custom_linux_kernel/?tl=ko
[41] https://www.redhat.com/ko/topics/security/container-security
[42] https://www.illumio.com/ko/cybersecurity-101/container-security
[43] https://www.ahnlab.com/ko/contents/content-center/30533
[44] https://www.wiz.io/ko-kr/academy/what-is-container-security
[45] https://byline.network/2022/07/25-217/
[46] http://www.boannews.com/media/view.asp?idx=112972
[47] https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART002866501
[48] https://guide.ncloud-docs.com/docs/container-ncr-1-5
[49] https://cloud.ibm.com/docs/Registry?topic=Registry-registry_images_&locale=ko
[50] https://themapisto.tistory.com/192
[51] https://dev-chicken.tistory.com/19
[52] https://iwanhae.tistory.com/3
[53] https://kin3303.tistory.com/14
[54] https://conanglog.tistory.com/69
[55] https://m.blog.naver.com/hj_kim97/222870522243
