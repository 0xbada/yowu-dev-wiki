---
title: VPC 환경에서 Elastic Beanstalk 을 사용해야할 때 알아둬야할 것들
description: Public Subnet 이라고 해서 무조건 외부 인터넷과 통신이 가능한 것이 아니다.
published: true
date: 2022-11-25T22:02:21.645Z
tags: aws, elasticbeanstalk, elasticip, vpc
editor: markdown
dateCreated: 2022-11-25T20:37:51.253Z
---


> AWS 에서 프로젝트를 빌드하면서 Custom VPC 를 생성
> 하지만 VPC 에서 ElasticBeanstalk 환경을 구축하는데 외부 인터넷 연결 문제로 고생을 했다.
> 관련되어 체크해야할 VPC 설정들을 정리함

# VPC 설정

## DNS 설정

- DNS 확인 활성화 :white_check_mark:
- DNS 호스트 이름 활성화 :white_check_mark:

# Subnet 설정

> 아래의 구성은 모든 케이스가 Internet <-> Public Subnet <-> Private Subnet 로만 통신된다.{.is-info}

## Public Subnet의 경우

- **라우팅 테이블** (Routing Table)
  - 서브넷 상위 VPC 의 CIDR 이 등록되어 있는지 확인
  - `0.0.0.0/0` 이 인터넷 게이트웨이로 등록되어 있는지 확인
- **네트워크 ACL** (Network ACL)
  - 모든 트래픽(유형)이 `0.0.0.0/0`(소스) `Allow` 로 등록되어 있는지 확인
  
### Public Subnet 추가 설정 :fire::fire:

> VPC 생성할 때 Public Subnet 으로 생성하더라도 아래 설정이 기본 값으로 설정되지 않는다.
> 그로 인해 ElasticBeanstalk 에서 신규 EC2 인스턴스를 생성할 때 Public IP 할당이 일어나지 않아서, EC2가 ElasticBeanstalk 와 통신이 실패하는 문제가 발생한다.
> 
> > **The EC2 instances failed to communicate with AWS Elastic Beanstalk**, either because of configuration problems with the VPC or a failed EC2 instance. Check your VPC configuration and try launching the environment again.{.is-danger}
> 
> 해당 이슈 때문에 가장 많은 시간을 소요함 :rage:
> 아래의 **자동 할당 IP 설정** 을 체크하면 ElasticBeanstalk 에서 EC2 인스턴스를 생성할 때 정상적으로 Public IP <small>(Elastic IP)</small>가 인스턴스에 할당된다.


- 자동 할당 IP 설정
  - 퍼블릭 IPv4 주소 자동 할당 활성화 :white_check_mark:

## Private Subnet의 경우

> Private Subnet은 외부 인터넷에 연결하지 않고 오직 Public Subnet과 통신한다.
> NAT을 설정하면 필요한 경우 Private Subnet 의 리소스가 외부에 접근이 가능한데, 여기서는 설정하지 않는다. (그리고 NAT은 매우 비싸다)
{.is-info}

- **라우팅 테이블** (Routing Table)
  - 서브넷 상위 VPC 의 CIDR 이 등록되어 있는지 확인
  - (S3와 같은 특정 AWS 리소스에 접근이 필요하다면) VPC 엔드포인트(vpce)가 설정되어 있는지 확인
  
> **유지보수 꼼수**
> - 라우팅 테이블 설정 시 특정 IP를 인터넷 게이트웨이에 연결해두고, EC2와 같은 인스턴스에 Public IP를 설정해두면, 특정 IP로만 Private Subnet 의 리소스에 접근할 수 있다.
  
- **네트워크 ACL** (Network ACL)
  - 모든 트래픽(유형)이 `0.0.0.0/0`(소스) `Allow` 로 등록되어 있는지 확인

# Security Group 설정

> 사용하는 AWS 리소스에 따라 적절하게 구성한다. 나는 EC2, RDS 전용 Security Group을 구성했다.
> 인바운드 규칙은 AWS 리소스가 사용하는 포트에 따라 다르게 구성하고, 아웃바운드 규칙은 모두 `0.0.0.0/0` 전체 TCP 트래픽 허용으로 동일하다.
{.is-info}

- `ec2-sg` inbound (webserver 전용)
  - 32 PORT (HTTP) 를 `0.0.0.0/0` 으로 허용
  - 443 PORT (HTTPS) 를 `0.0.0.0/0` 으로 허용
- `rds-sg` inbound (database 전용)
  - 3306 PORT (MySQL/MariaDB의 경우) 를 `ec2-sg` 허용

> **유지보수 꼼수**
> - 라우팅 테이블 때와 마찬가지로, 특정 IP로만 subnetmask 32로 전체 트래픽 허용하게 설정할 수 있다.

---

- https://notes.webutvikling.org/elastic-beanstalk-in-a-vpc/
{.links-list}
