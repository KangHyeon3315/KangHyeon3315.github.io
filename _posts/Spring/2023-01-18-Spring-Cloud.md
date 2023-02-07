---
title:  "Spring Cloud 알아보기"
excerpt: ""

categories:

- Spring
tags:
- [Cloud]

toc: true
toc_sticky: true

date: 2023-01-18
last_modified_at: 2023-01-18
comments: true
---

나무소리님의 [Microservices with Spring Boot and Spring Cloud](https://www.youtube.com/playlist?list=PLOSNUO27qFbv95vD0Cc5Vwtro4vcMZGiy) 강좌를 듣고 제가 이해한대로 요약한 내용입니다.

# 모놀리스 아키텍처와 마이크로서비스 아키텍처

## 모놀리스(Monolith) 아키텍처란?

모놀리스 아키텍처는 모든 업무 로직을 하나의 어플리케이션 형태로 묶어 서비스하는 아키텍처입니다.

### 장점

- 실행 파일 또는 디렉토리가 하나여서 배포가 더 쉽습니다.
- 하나의 코드 베이스로 애플리케이션을 구축하여 개발이 더 쉽습니다.
- 모놀리식 애플리케이션은 하나의 중앙 집중식 장치이므로 분산된 애플리케이션보다 엔드투엔드 테스트를 더 빠르게 수행할 수 있습니다.
- 모든 코드가 한 곳에 있으므로 요청을 따라가서 문제를 찾기가 더 쉽습니다.

### 단점

- 구성 요소에 기능을 추가하거나 수정하는 것은 매우 큰 비용을 초래합니다.
- 규모가 커질수록 개발 및 유지보수, 배포에 이르기까지 많은 문제점을 갖습니다.
- 개발팀의 분리부터 그에 대한 관리, 프로덕션 배포까지 시간이 흐를 수록 복잡도가 증가합니다.
- 전체 코드에 대한 이해도가 떨어집니다.
- 하위 기능으로 내려갈 수록 코드의 복잡성은 더욱 증가합니다.
- 새로운 기술을 적용하면 전체 어플리케이션이 영향을 받기 때문에 반영하기 어렵습니다.

 <br/>

## 마이크로서비스(Micro Service) 아키텍처란?

마이크로서비스 아키텍처는 큰 소프트웨어를 작은 소프트웨어 조각으로 나눠 서비스하는 아키텍처입니다.

이때 각 작은 조각은 하나의 비즈니스 목표를 가지고 분리한 후 개발하고 배포합니다.

### 장점

- 작은 코드 베이스를 가지므로 관리가 용이합니다.
- 개별적인 컴포넌트로 스케일 관리가 쉽습니다.
- 기술에 대한 다양성을 가질 수 있습니다.
- 결함을 고립하여 하나의 시스템이 다운되더라도 전체 시스템이 다운되는 것을 방지할 수 있습니다.
- 동시에 작업하는 더 작은 개발팀을 구성할 수 있습니다.
- 독립적인 배포가 가능합니다.

### 단점

- 서비스간의 강한 일관성을 달성하기 어렵습니다.
- 분산 환경이므로 디버깅과 추적이 어렵습니다.
- End-to-end 테스트의 필요성이 증가합니다.
- 애플리케이션 개발과 배포방법(CI/CD)이 중요해집니다.
- 서비스간 호출에 따른 커뮤니케이션 비용이 증가합니다.


# Spring Cloud
Spring cloud는 분산 시스템에서 일부 공동 패턴을 빠르게 만들 수 있는 툴을 제공하는 프레임워크입니다. 

주요 기능은 다음과 같습니다.
- 분산되고 버전화된 Configuration
- 서비스의 등록 및 검색
- 라우팅
- 서비스간 호출
- 부하 분산
- 결함 허용
- 글로벌 락
- 분산 시스템의 리더 선택과 클러스터 상태
- 분산 메시징


# Spring Cloud의 주요 프로젝트

- [Spring Cloud Config](../Spring-Cloud-Config/)

	Git Repository에서 중앙화된 Configuration을 관리하는 기능을 제공합니다. Configuration flthtmsms 스프링 환경에 직접 매핑되고, 스프링이 아닌 어플리케이션에도 Configuration을 사용할 수 있도록 구현할 수 있습니다
	
- Spring Cloud Netflix

	다양한 Netflix OSS 구성 요소(Eureka, Hystrix, Zuul, Archaius 등)와 통합된 프로젝트입니다.

	- Eureka : Discovery Server

		각각의 서비스 인스턴스들이 동적으로 확장 및 축소 되더라도 인스턴스의 상태를 하나의 서비스로 관리할 수 있는 서비스

	- Hystrix : Circuit Breaker

		특정 서비스가 과부하가 걸려 서비스 장애를 전파하는 특성을 해결하려는 기능

	- Zuul : API Gateway

		각각의 마이크로 서비스의 종단점을 연결하는 리버스 프록시

	- Archaius : Spring Boot external config + Spring cloud config

		다양한 소스에서 config 프로퍼티를 수집하고 안전하게 액세스하는 기능을 제공하는 기능



- Spring Cloud Bus

	서비스 및 서비스 인스턴스를 분산 메시징과 함께 연결하기 위한 이벤트 버스입니다. 
	
	클러스터 전체에 상태 변경을 전파하는데 유용한 프로젝트입니다.

- Spring Cloud Cloudfoundry

	애플리케이션을 Pivotal Cloud Foundry와 통합합니다. 
	
	서비스 검색 구현을 제공하고 SSO 및 OAuth2를 쉽게 구현할 수 있습니다.

- Spring Cloud Open Service Broker

	Open Service Broker API를 구현하는 서비스 브로커 구축을 위한 시작점을 제공합니다.

	서비스 브로커(Service Brokers) : 서비스들을 다른 앱들에서 활용하도록 돕는 어플리케이션입니다.

- Spring Cloud Cluster

	Zookeeper, Redis, Hazelcast, Consul에 대한 추상화 및 구현을 통한 분산 시스템의 리더 선택과 공통 상태 패턴에 관한 프로젝트입니다.

- Spring Cloud Consul

	Hashicorp Consul을 통한 서비스 검색 및 구성 관리 기능을 제공하는 프로젝트입니다.

- Spring Cloud Security

	Zuul 프록시에서 부하 분산된 OAuth2 Rest 클라이언트 및 인증 헤더 릴레이에 대한 지원을 제공합니다.

- Spring Cloud Sleuth

	Zipkin, HTrace 및 로그 기반(ELK) 추적과 호환되는 Spring Cloud 애플리케이션용 분산 추적 프로젝트입니다.

- Spring Cloud Data Flow

	최신 런타임에서 구성 가능한 마이크로서비스 애플리케이션을 위한 클라우드 네이티브 오케스트레이션 서비스입니다. 
	
	사용하기 쉬운 DSL, drag-and-drop GUI 및 REST-API가 함께 마이크로서비스 기반 데이터 파이프라인의 전반적인 오케스트레이션을 단순화합니다.

- Spring Cloud Stream

	외부 시스템에 연결할 수 있는 애플리케이션을 빠르게 빌드하기 위한 경량 이벤트 기반 마이크로서비스 프레임워크입니다. 
	
	Spring Boot 앱 간에 Apache Kafka 또는 RabbitMQ를 사용하여 메시지를 보내고 받는 간단한 선언적 모델입니다.

- Spring Cloud Stream Applications

	Spring Cloud Stream 애플리케이션은 Spring Cloud Stream의 바인더 추상화를 사용하여 Apache Kafka, RabbitMQ 등과 같은 외부 미들웨어 시스템과의 통합을 제공하는 즉시 사용 가능한 Spring Boot 애플리케이션입니다.

- Spring Cloud Task

	제한된 양의 데이터 처리를 수행하는 애플리케이션을 빠르게 구축하기 위한 수명이 짧은 마이크로서비스 프레임워크입니다. 

- Spring Cloud Task App Starters

	Spring Cloud Task 앱 스타터는 영원히 실행되지 않고 한정된 데이터 처리 기간 후에 종료/중지되는 Spring Batch 작업을 포함한 모든 프로세스일 수 있는 Spring Boot 애플리케이션입니다.

- Spring Cloud Zookeeper

	Apache Zookeeper를 사용한 서비스 검색 및 구성 관리기능을 제공하는 프로젝트입니다.

- Spring Cloud Connectors

	다양한 플랫폼의 PaaS 애플리케이션이 데이터베이스 및 메시지 브로커와 같은 백엔드 서비스에 쉽게 연결할 수 있도록 하는 프로젝트입니다.

- Spring Cloud CLI

	Groovy에서 빠르게 Spring Cloud 구성 요소 애플리케이션을 생성하기 위한 Spring Boot CLI 플러그인입니다.

- Spring Cloud Contract

	Spring Cloud Contract는 사용자가 소비자 중심 계약 접근 방식을 성공적으로 구현하는 데 도움이 되는 솔루션을 보유한 우산 프로젝트입니다.

- Spring Cloud Gateway

	Spring Cloud Gateway는 Project Reactor를 기반으로 하는 지능적이고 프로그래밍 가능한 라우터입니다.

- Spring Cloud OpenFeign

	Spring Cloud OpenFeign은 Spring 환경 및 기타 Spring 프로그래밍 모델 관용구에 대한 자동 구성 및 바인딩을 통해 Spring Boot 앱에 대한 통합을 제공합니다.

- Spring Cloud Pipelines

	Spring Cloud Pipelines는 중단 시간이 없는 방식으로 애플리케이션을 배포하고 문제가 발생한 경우 쉽게 롤백할 수 있도록 하는 단계와 함께 독자적인 배포 파이프라인을 제공하는 프로젝트입니다.

- Spring Cloud Function

	Spring Cloud Function은 함수를 통해 비즈니스 로직 구현을 촉진합니다. 서버리스 공급자 전체에서 균일한 프로그래밍 모델을 지원하고 독립 실행형(로컬 또는 PaaS에서) 실행 기능을 지원합니다.


# 출처

- [Microservices with Spring Boot and Spring Cloud](https://www.youtube.com/playlist?list=PLOSNUO27qFbv95vD0Cc5Vwtro4vcMZGiy)
- [마이크로서비스와 모놀리식 아키텍처 비교](https://www.atlassian.com/ko/microservices/microservices-architecture/microservices-vs-monolith)
- [Spring Cloud Docs](https://spring.io/projects/spring-cloud)