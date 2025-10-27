이기종 정찰위성 통합 운용을 위한 확장형 지상국 소프트웨어 아키텍처 설계
Design of a Scalable Ground Station Software Architecture for Integrated Operation of Heterogeneous Reconnaissance Satellites
이상훈*,1) ․ 김응백1)
Sanghun Lee*,1) ․ Eungbaek Kim1)


[ 초      록 ]
 본 연구는 EO·IR·SAR 등 이기종 정찰위성을 단일 지상국에서 효율적으로 운용하기 위한 MSA 기반 확장형 지상국 소프트웨어 아키텍처를 제안한다. 기존 시스템의 한계를 분석하고 서비스 분리와 플러그인 구조를 적용하여 신규 위성 추가 시 최소한의 개발로 통합 운용이 가능함을 보였으며 메시지 기반 통신으로 결합도를 낮추고 확장성을 확보하였다.


[ ABSTRACT ]
 This study proposes a microservices architecture (MSA)-based extensible ground station software architecture for the efficient operation of heterogeneous reconnaissance satellites—such as EO, IR, and SAR—from a single ground station. By analyzing the limitations of existing systems and applying service separation and a plugin-based structure, the architecture enables integrated operations with minimal development effort when adding new satellites. Furthermore, message-based communication is employed to reduce coupling and enhance scalability.


Key Words : Heterogeneous Reconnaissance Satellites(이기종 정찰위성), Microservice Architecture(마이크로서비스 아키텍처), Scalable Ground Station Software(확장형 지상국 소프트웨어), Message-Oriented Middleware(메시지 기반 통신)

1. 서 론


1) 한화시스템 우주연구소
(Space Research Institute, Hanwha Systems, Korea)
*Corresponding author, E-mail: shlee124@hanwha.com
Copyright ⓒ The Korean Institute of Defense Technology
received :                    Revised :
Accepted :

  정찰위성은 군사 및 안보 분야에서 핵심적인 정보 자산으로 활용되며 관측 대상의 상황에 따라 전자광학(EO), 적외선(IR), 합성개구레이더(SAR) 등 다양한 센서로부터 정보를 수집한다. 그러나 지금까지 대부분의 위성 지상국 소프트웨어는 특정 위성 또는 동종 센서군에 맞추어 개별 개발 및 운용되어 왔다. 이로 인해 이기종 정찰위성 통합 운용 시 시스템 간 연계가 부족하고 운용 인력은 각기 다른 시스템을 병행 모니터링해야 하는 비효율이 발생한다. 기존의 모놀리식(monolithic) 지상국 시스템은 여러 개의 독립 애플리케이션으로 구성되어 있으나, 각 기능 간 결합도가 높고 규모가 커 전체적으로는 단일 시스템처럼 동작한다. 이로 인해 부분 기능의 수정이나 신규 위성 운용 기능 추가 시 전체 시스템의 재배포가 필요했다[1].
이로 인해 유지보수 비용이 증가하고 다중 위성 통합이나 임무 변경의 유연성이 떨어지는 한계가 있었다[2].

  본 논문의 목적은 EO·IR·SAR와 같이 상이한 정찰 센서를 탑재한 위성들을 하나의 지상국 소프트웨어 플랫폼에서 통합 운영하기 위한 확장형 소프트웨어 아키텍처를 제안하는 것이다. 제안하는 아키텍처는 마이크로서비스 기반으로 구성하여 기존 지상국 시스템의 한계를 극복하고자 하였다. 본론에서는 제안한 아키텍처의 설계와 지상국 소프트웨어 적용 예시를 기술하였다.


2. 기존 지상국 시스템 구성 및  문제점
 
2.1 기존 지상국의 일반적인 구성
  기존 대부분의 지상국은 단일 위성의 임무 수행을 위한 필수 기능들을 체계적으로 구분하여 구성되어 있으나 설계 초점이 서비스 지향보다는 각 기능의 역할 수행에 중점을 둔 구조로 되어 있다. 일반적으로 지상국은 그림 1과 같이 크게 관제시스템(Mission Control System, MCS)과 영상수신처리시스템(Image Reception and Processing System, IRPS)으로 구성되고 각 시스템은 기능 역할에 따라 하위 서브시스템으로 구성되어 있다.


그림 1. 기존 지상국 구조
Fig. 1. Legacy ground station architecture

2.1.1 관제시스템
  관제시스템(Mission Control System, MCS)은 위성 임무 수행과 운용 제어의 핵심 역할을 담당하는 시스템으로 임무계획, 비행역학, 위성운용 세 개의 서브시스템으로 구성된다.

 1) 임무계획 서브시스템
 임무계획 서브시스템(Mission Planning Subsystem, MPS)은 촬영계획, 수신계획, 기동계획 등 외부 또는 내부에서 생성된 다양한 임무계획을 수신하여 검증 및 스케줄링을 수행하고 이를 위성 운용이 가능한 명령 시퀀스로 변환한다. 검증된 임무계획은 위성운용 서브시스템으로 전달되어 실제 명령 전송 절차를 거쳐 위성에 업로드 된다.
 2) 비행역학 서브시스템
 비행역학 서브시스템(Flight Dynamics Subsystem, FDS)은 위성의 궤도 결정, 예측, 충돌 회피 분석 등을 수행하며 궤도 유지 및 궤도 기동에 필요한 기동 명령을 산출한다. 연료 사용량, 통신 링크 가시성 등 운용 제약조건을 고려하여 효율적인 궤도 유지 전략을 수립한다. 또한 생성된 기동계획은 임무계획 서브시스템으로 전달되어 명령 검증 및 실행 스케줄에 반영된다.
 3) 위성운용 서브시스템 
 위성운용 서브시스템(Satellite Operation Subsystem, SOS)은 위성과의 원격측정 수신 및 원격명령 송신을 담당하며 위성 상태 감시 및 명령 수행 결과를 실시간으로 관리한다. 실시간 운용, 자동 스케줄 운용, 이상 상태 감시 및 복구 절차 등을 지원한다.

2.1.2 영상수신처리시스템
  영상수신처리시스템(Image Reception and Processing System, IRPS)는 위성으로부터 전송된 관측 데이터를 수신하고 이를 영상 및 분석 가능한 정보로 변환하는 시스템으로 직저장, 제품관리, 영상수집계획 세 개의 서브시스템으로 구성된다.

 1) 직저장 서브시스템
 직저장 서브시스템(Direct Ingestion Subsystem, DIS)은 안테나 시스템을 통해 수신된 원시 데이터를 실시간으로 수신 및 처리하여 L0F 파일을 생성한다. 대용량 데이터 수신 환경에 대응하기 위해 버퍼 기반 병렬처리 구조를 적용하여 데이터 무결성과 안정성을 확보한다.
 2) 제품관리 서브시스템
 제품관리 서브시스템(Product Management Subsystem, PMS)은 직저장 서브시스템으로부터 L0F 파일을 입력으로 전달받아 데이터 압축 해제, 방사보정 및 기하보정 등을 수행하여 영상을 생성한다. 또한 영상 생성에 대한 작업 모니터링 및 생성된 영상에 대한 관리 및 배포를 수행한다.
 3) 영상수집계획 서브시스템
 영상수집계획 서브시스템(Image Collection Planning Subsystem, ICPS)은 위성 궤도, 지상국 가시구간, 탑재체 제약조건, 위성 자원 상태 등을 종합적으로 고려하여 촬영계획 및 수신계획을 통합적으로 수립한다. 수립된 계획은 MCS의 임무계획 서브시스템으로 전달되어 검증 및 명령 변환 절차를 거친다.

2.2 기존 구조의 문제점
  현재 운용 중인 대부분의 위성 지상국 소프트웨어는 특정 위성 또는 동종 센서군(예: EO 전용, SAR 전용)을 대상으로 독립적으로 개발되어 왔다. 이러한 구조는 초기 구축 시에는 단일 위성의 임무 수행에 최적화되어 효율적이지만 이기종 위성의 통합 운용이나 신규 위성의 추가 운용이 필요한 시점에서는 심각한 한계를 드러낸다.

  예를 들어, 기존 EO 위성을 운용하던 지상국에서 새로운 SAR 위성을 추가하려 할 경우 SAR 위성의 특수한 명령 포맷, 비행역학 모델, 데이터 전송 프로토콜을 기존 시스템에 반영해야 한다. 그러나 모놀리식 구조로 설계된 지상국 소프트웨어는 핵심 기능이 하나의 응용 프로그램 내부에 강하게 결합되어 있어 특정 기능만 수정하거나 추가하기 어렵다. 그 결과 새로운 위성 추가를 위해서는 시스템 재개발이 불가피하며 개발 기간과 비용이 크게 증가한다.

  또한 이러한 구조에서는 운용 인력의 비효율도 발생한다. 위성별로 서로 다른 GUI, 명령어 체계, 모니터링 화면을 제공하기 때문에 운용자는 여러 개의 프로그램 상태를 수동으로 확인해야 한다. 예컨대 EO 위성의 촬영 명령을 전송한 직후 SAR 위성의 수신 계획을 확인하려면 두 개의 별도 시스템을 전환해야 하며 데이터 관리 또한 개별 저장소에 분산되어 있어 통합 운용이 어렵다.

  이와 같은 문제는 지상국 소프트웨어의 확장성과 재사용성 부족에서 기인한다. 위성별로 독립된 시스템을 구축하는 방식은 단기적으로는 신속한 개발이 가능하지만 장기적으로는 운영 복잡도 증가, 유지보수 비용 상승, 그리고 임무 확장성 저하로 이어진다. 따라서 향후 다양한 위성 플랫폼이 병행 운용되는 환경에서는 모듈화와 서비스 분리를 통해 확장 가능한 아키텍처로의 전환이 필수적이다.


3. 지상국 아키텍처 설계 제안

  본 연구에서는 마이크로서비스 아키텍처, 컨테이너 기반 배포 환경, 메시지 기반 통신 구조를 적용한 확장형 지상국 소프트웨어 아키텍처를 제안한다.

3.1 마이크로서비스 아키텍처


 그림 2. 모놀리식, 마이크로서비스 아키텍처 비교
Fig. 2. Comparison of monolithic and MSA

  마이크로서비스 아키텍처(Microservice Architecture, MSA)는 그림 2와 같이 시스템을 여러 개의 독립적인 서비스 단위로 분리하여 각 서비스가 독립적으로 개발, 배포, 확장될 수 있도록 하는 구조이다. 기존의 모놀리식 구조는 일부 기능 변경에도 전체 시스템 재배포가 필요하고 장애가 전체로 확산되는 문제가 있었다. 반면 MSA는 기능 별 서비스를 분리하여 결합도를 낮추고 기능 단위의 확장성·유지보수성·복원력을 확보할 수 있다[3].

  정찰위성 지상국 시스템들은 기본적으로 동일한 데이터 흐름과 관리 로직을 공유하지만 위성의 센서 특성에 따라 세부 기능이 달라진다. 따라서 제안하는 구조에서는 공통 기능은 유지하고 위성 별 특화 기능만 플러그인 형태로 분리하여 추가하도록 설계할 수 있다.

 1) PMS 적용 예시
 공통적으로 영상 처리 작업에 대한 스케줄링, 처리된 영상에 대한 저장 및 조회, 처리 상태 모니터링 등의 기능을 수행한다. 그러나 위성 별, 센서 별로 처리 알고리즘(Processor)은 다르다. 이 경우 공통 서비스는 동일하게 유지하되 상이한 처리 알고리즘은 플러그인 모듈 형태로 업로드하여 적용한다. 운용자는 신규 위성 추가 시 해당 위성의 처리 플러그인만 Docker 이미지 형태로 등록하기만 하면 된다.
 2) MPS 적용 예시
 임무계획 데이터를 수신하여 검증하고 이를 명령 시퀀스로 변환한 뒤 SOS에 전달하는 프로세스는 공통적이지만 위성 별로 파일 포맷, 유효성 검증 방식, 명령 시퀀스의 변환 규칙이 다르다. 공통 임무계획 서비스는 동일하게 유지하고 위성 별 스펙과 제약조건을 반영한 검증 및 변환 모듈을 플러그인 형태로 추가만 하면 되는 방식으로 개발이 가능하다.
 3) SOS 적용 예시
 명령 송수신 기능과 상태 모니터링 기능은 공통이며 위성 별 명령 포맷이나 텔레메트리 항목만 다르다. SOS는 위성 ICD(Interface Control Document)를 기반으로 플러그인 명령 파서 및 텔레메트리 디코더를 플러그인 형태로 등록하여 간단히 추가할 수 있다.

  이러한 방식으로 지상국 소프트웨어의 구조적 복잡성을 단순화하고 기능 간 결합도를 최소화함으로써 확장성과 유연성을 극대화할 수 있다. 각 서비스는 독립적인 단위로 설계되어 위성 별로 상이한 알고리즘이나 운용 로직을 플러그인 형태로 손쉽게 추가할 수 있으며 시스템 전체를 재구성하지 않고도 새로운 위성 통합을 가능하게 한다. 대표적인 오픈소스 및 기술 스택은 표 1과 같다.

표 1. 마이크로서비스 아키텍처 오픈소스 및 기술 스택
Table 1. Microservice architecture: open-source and technology stack

구분
기술명
주요 기능
MSA 프레임워크
Spring Cloud, NestJS, FastAPI
REST/gRPC 기반 마이크로서비스 구현
API Gateway
Traefik, Nginx, Kong
통합 라우팅, 인증 및 보안 관리
설정 관리
Spring Cloud Config, etcd
서비스 환경 설정 중앙 집중 관리


3.2 컨테이너 기반 배포 환경
  Docker는 응용 프로그램과 실행 환경을 하나의 경량 컨테이너로 패키징하여 어디서나 동일하게 실행할 수 있게 하는 오픈소스 플랫폼이다. 가상머신(VM) 대비 빠른 시작 속도와 낮은 리소스 사용량을 가지며 마이크로서비스 아키텍처 구조에서 각 서비스를 독립적으로 배포 및 관리할 수 있도록 지원한다. 컨테이너 기술은 일관된 배포 환경, 신속한 복구, 확장 자동화 측면에서 지상국 시스템의 안정적인 운용에 적합하다.

  예를 들어 서비스별로 이기종 언어와 하드웨어를 사용하는 복잡한 구조를 가져, 비행역학은 Python 기반, 위성운용은 Java 기반, 영상처리는 GPU 연산이 필요한 C++ 기반으로 개발될 수 있다. 이러한 이질적인 환경을 통합하기 위해 각 서비스를 Docker 컨테이너로 패키징하고 Kubernetes(K8s)를 통해 오케스트레이션한다. 시스템 부하가 증가할 경우 Kubernetes의 자동 스케일링 기능을 통해 영상처리 컨테이너만 즉시 확장하여 자원 효율을 극대화한다. 또한, 장애가 발생한 서비스 컨테이너는 자동으로 재시작되어 무중단 운용을 실현할 수도 있다. 대표적인 오픈소스 및 기술 스택은 표 2와 같다.

표 2. 컨테이너 기반 배포 환경 오픈소스 및 기술 스택
Table 2. Containerized deployment environment: open-source and technology stack

구분
기술명
주요 기능
컨테이너 런타임
Docker, Podman
경량화된 컨테이너 실행 환경 제공
오케스트레이션
Kubernetes(K8s), Docker Swarm
서비스 자동 배포, 스케일링, 복구
이미지 저장소
Docker Hub, Harbor
컨테이너 이미지 관리 및 버전 관리
모니터링
Prometheus, Grafana
자원 사용량 및 상태 시각화
CI/CD
Jenkins, ArgoCD
자동 빌드 및 지속적 배포 파이프라인


3.3 메시지 기반 통신 구조

그림 3. 메시지 기반 통신 구조
Fig. 3. Message-driven architecture

  메시지 기반 통신 구조(Message-driven Architecture)는 그림 3과 같이 서비스 간 직접 호출 대신 메시지 브로커(Message Broker)를 매개로 하여 비동기적으로 이벤트를 전달하는 구조이다[4]. 각 서비스는 이벤트를 발행(Publish)하거나 구독(Subscribe)하는 방식으로 동작하며 이를 통해 서비스 간 결합도를 최소화할 수 있다. 직접 호출 방식에서는 서비스 간 인터페이스 변경이 전체 시스템에 영향을 미칠 수 있으나 메시지 기반 구조에서는 이벤트의 형식만 유지되면 신규 서비스 추가나 수정이 기존 서비스에 영향을 주지 않는다. 따라서 각 서비스는 독립적으로 개발 및 운용되면서도 메시지 구독 설정만으로 손쉽게 상호 연동이 가능하다.

  예를 들어, 지상국 소프트웨어는 다음과 같은 이벤트 중심의 데이터 흐름으로 구성할 수 있다.
 1) 임무계획 서비스(MPS)가 촬영계획 완료 후 “MissionPlan Completed” 이벤트를 발행한다.
 2) 비행역학 서비스(FDS)는 해당 이벤트를 구독하여 궤도 예측을 수행하고 결과를 “OrbitUpdated” 이벤트로 발행한다.
 3) 임무운용 서비스(SOS)는 궤도 갱신 이벤트를 수신하여 명령을 생성·송신한다.
 4) 직저장 서비스(DIS)는 “DownlinkStart” 이벤트를 구독하여 데이터 수신을 개시한다.
 5) 영상처리 서비스(PMS)는 “DataReceived” 이벤트를 받아 영상 처리 절차를 자동으로 수행한다.
  이로써 각 시스템 간의 메시지 발행과 구독으로 직접 호출 없이 결합도를 낮추며 연동이 가능하다. 메시지 기반 통신의 대표적인 오픈소스 및 기술 스택은 표 3과 같다.

표 3. 메시지 기반 통신 오픈소스 및 기술 스택
Table 3. Message-based communication architecture: open-source and technology stack

구분
기술명
주요 기능
메시지 브로커
RabbitMQ, Apache Kafka, NATS
비동기 이벤트 처리 및 스트리밍 지원
워크플로우 엔진
Apache Airflow, Temporal
이벤트 기반 프로세스 제어 및 스케줄링
메시지 포맷
Protocol Buffers, Avro, JSON Schema
서비스 간 표준화된 데이터 구조 정의



4. 통합 지상국 아키텍처 구성

  그림 4는 본 연구에서 제안하는 이기종 정찰위성 통합 지상국 소프트웨어 아키텍처의 전체 구성을 나타낸다.
그림 4. 통합 지상국 아키텍처
Fig. 3. Integrated ground segment architecture

 1) API 서비스
 먼저 API 서비스는 PMS, MPS, SOS 등 각 시스템의 주요 기능을 수행하는 핵심 서비스이다. 각 서비스는 공통적으로 필요한 기능을 포함하고 위성의 탑재체 특성이나 운용 방식에 따라 달라지는 기능은 위성 특화된 플러그인 형태로 모듈화되어 있다. 이러한 구조를 통해 이기종 정찰위성의 운용 로직이나 데이터 처리 알고리즘 등을 서비스 전체를 재배포하지 않고도 손쉽게 추가 및 교체할 수 있다. 또한 운용 클라이언트는 이러한 동적으로 변동되는 서비스들을 API Gateway를 통해 일관된 방식으로 접근할 수 있다.
 2) 컨테이너 배포 환경
 모든 서비스는 Docker와 Kubernetes를 기반으로 한 컨테이너 오케스트레이션 환경으로 구성된다. 각 마이크로서비스는 독립된 컨테이너로 패키징되어 배포된다. 일관된 배포 환경을 통해 시스템 전반의 운용 안정성을 확보하고 위성 추가나 서비스 교체 시 환경 간 불일치 문제를 제거한다.
 3) 이벤트 기반 통신 
 메시지 브로커 중심의 이벤트 기반 통신을 수행한다. 모든 서비스는 직접 호출 방식이 아닌 Publish/Subscribe 모델을 통해 상호 연동되며 이벤트 단위의 데이터 흐름을 통해 임무계획–비행역학–위성운용–영상처리 간 절차가 자동으로 연결된다. 새로운 서비스 추가 시 기존 구조를 변경하지 않고 이벤트 구독만 등록하면 쉽게 통합할 수 있다.


5. 결 론

  본 연구에서는 EO·IR·SAR 등 이기종 정찰위성을 단일 지상국에서 효율적으로 통합 운용하기 위한 확장형 지상국 소프트웨어 아키텍처를 제안하였다. 제안된 구조는 마이크로서비스 아키텍처를 기반으로 하여 각 기능을 독립된 서비스 단위로 분리하고 컨테이너 기반 배포 환경과 메시지 기반 통신 구조를 적용함으로써 기존 모놀리식 구조의 한계를 극복하였다.

  제안된 구조는 다양한 위성 플랫폼이 혼재된 환경에서 단일 통합 운용 체계 구축을 가능하게 하고 향후 위성 수 증가나 새로운 임무 요구사항 변화에도 지속적으로 확장 가능한 기반 구조로 활용될 수 있다. 나아가 본 연구의 아키텍처는 위성 운영뿐 아니라 타 감시 및 정찰 체계나 지상 인프라 통합 시스템에도 활용될 수 있을 것이다.




References

[1] Y. Wang, W. Han and Z. Nian, “Design of satellite ground management system based on microservices,” Proceedings of the 3rd International Conference on Computer Science and Software Engineering, pp. 119–123, 2020.
[2] W. Lu, Q. Xu, C. Lan, L. Lyu, Y. Zhou, Q. Shi and Y. Zhao, “Microservice‐Based Platform for Space Situational Awareness Data Analytics,” Int. J. Aerospace Engineering, Vol. 2020, No. 1, pp. 8149034, 2020.
[3] A. Chavan, “Exploring event-driven architecture in microservices: Patterns, pitfalls, and best practices,” Int. J. Software and Research Analysis, 2021. [Online]. Available: https://ijsra.net/content/exploring-event-driven-architecture-microservices-patterns-pitfalls-and-best-practices
[4] A. Ghosh, “Event-Driven Architectures for Microservices: A Framework for Scalable and Resilient Rearchitecting of Monolithic Systems,” Int. J. Science and Technology (IJSAT), 2025.
