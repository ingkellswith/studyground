SAA 문제 풀이 정리
===
#1. S3 transfer fee
- 721p
- s3로의 ingress fee : $0
- s3 to cloudfront : $0
- s3 to internet, cloudfront to internet : 기가바이트당 $0.1수준이고 cloudfront를 통한 egress가 더 싸다.
- s3 transfer acceleration : $0.04 ~ $0.08 per GB
- s3 cross region replication : $0.02 per GB
- 결론 : **S3 to internet**이 cost가 꽤 된다.
- 주의 : **storage cost(per GB per month) : 저장 용량 비용, retrieval cost**가 존재한다.

#2. S3 versioning
- Different versions of a single object can have different retention modes and periods : 다른 버전의 오브젝트는 보유 기간을 다르게 설정할 수 있다.
- object version에 명시적으로 Retain Until Date설정 가능, (default 기능 아님)
- 버켓에서 versioning을 한 번 설정하면 다시 unversioned될 수 없지만 version을 suspend할 수 있음

#3. Resilient to periodic spikes in request rates : 급격한 요청 증가에 대한 회복 탄력성이 존재하는가
- Cloudfront
  - **regional failover**을 지원한다는 점에서 resilient하다.
- Aurora
  - read replica는 최대 15개까지 multi az in a region이 가능하다
  - read replica가 write instance가 다운되면 write instance가 될 수 있다는 점에서 resilient하다.

#4. Storage Cost
- EFS : $0.30 per GB
- gp2(SSD) : $0.10 per GB
- s3 : $0.023 per GB
- s3 < gp2 < EFS 순으로 비싸지만 gp2는 provision해야 해서 사용된 만큼 지불되는 것이 아니다.

#5. EBS volumes
- io2 Block Express : 256,000IOPS
- io2 : max 64,000IOPS, size는 4GiB to 16TiB
- io1/2만 multi-attach가능 gp는 불가능

#6. SQS Batch
- SQS FIFO Queue는 초당 300개까지 처리가능한데 batch mode로 돌리면 최대 3000개까지 처리가능하다.
- operation하나당 최대 300개까지 처리가능하므로 4 messages per operation이면 최대 1200개 처리가능하다.

#7. Placement Group
- Cluster Placement Group : 모여 있어서 네트워크적으로 이점 존재
- Partition Placement Group : partition간은 독립적이나 partition내부적으로는 네트워크적으로 이점 존재
- Spread Placement Group : high availability

#8. own custom DNS service
- own custom DNS service를 사용하면 Route 53을 사용할 필요가 없다.

#9. UDP protocol, fast regional failover if an AWS Region goes down
- Global Accelerator improves performance for a wide range of applications over **TCP or UDP** by proxying packets at the edge to applications running in one or more AWS Regions. 
- Global Accelerator is a good fit for **non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP, as well as for HTTP** use cases that specifically require static IP addresses or deterministic, **fast regional failover**. 

#10. ASG
- 패치 전략
  - ReplaceUnhealthy를 suspend해서 패치완료 될 때까지 인스턴스를 replace하지 않도록 함
  - standby state를 사용해서 패치완료함. standby state의 인스턴스는 asg안에 포함되나 트래픽을 받지 않음

#11. ASG policy
- scaling policy
- cpu 50%를 타겟으로 인스턴스의 개수를 유지하고 싶다면 target tracking policy
- 최저점, 최고점 기준으로 인스턴스의 개수를 유지하고 싶다면 step, simple policy
- step과 simple policy의 차이점
  - simple은 인스턴스가 생성되거나 제거되는 과정 중에서 policy를 실행해 추가적인 작업을 발동시키지 않는다.
  - step은 인스턴스가 생성되거나 제거되는 과정 중에서도 policy를 실행해 추가적인 작업을 발동시킨다.

#12. 유용한 Architecture
- Kinesis Firehose는 DynamoDB에 데이터를 전달할 수 없다.
- SQS Standard > Lambda Batch > DynamoDB  
- **Sudden Traffic Spike** 감당가능한 서비스(실전 테스트1 - 36)
  - API GATEWAY(Token Buffer로 request의 개수 제한 가능) > SQS(Buffer 역할 가능) > KINESIS(Buffer 역할 가능)
- **Major traffic spikes** 감당 가능, store the processed updates in a highly available database, minimize the management overhead(서버리스에 가까운)
  - Kinesis Data Streams(order 보장, 많은 데이터 소스로부터 데이터 수집가능) > Lambda function > DynamoDB

#13. HDD는 Boot Volume이 될 수 없다.

#14. Aurora Read Replica failover priority
- tier가 낮은 것 우선(ex. 1이 15보다 우선순위가 높음)
- tier가 같다면 용량이 높은 것 우선
- tier와 용량이 같다면 arbitrary로 배정

#15. Instance Store : fleet of instances에서 instance하나가 다운되어도 다른 인스턴스가 대체가능하다는 점에서 resilient architecture이다.
- fleet of instances가 있을 때 instance store를 사용하면 instance간에 instance store의 데이터를 공유 가능
- 따라서 인스턴스 하나가 다운되어도 다른 인스턴스가 다운된 인스턴스가 사용하던 데이터에 접근 가능
- **임시 스토리지** 
- 호스트 컴퓨터에 물리적으로 부착됨
- load-balanced pool of web servers : best practice
- Instance store volumes are included as part of the instance's usage cost : instance처럼 취급되어 instance 비용에 청구됨

#16. ALB는 사설 IP, Instance, lambda를 대상으로 하고 **공인 IP로 라우팅 하지 않는다.**

#17. Direct Connect
- 실전 테스트1 - 27번, 다시 보는 것 추천
- Site-to-site VPN cannot provide low latency and high throughput connection : Internet-based connectivity를 사용하는 VPN은 속도가 빠르지 않다.
- Site-to-site VPN : 하지만 Immediate need에 대응가능하다.
- Site-to-site VPN : IPSec to establish encrypted network connectivity between your intranet and Amazon VPC over the Internet > Internet-based connectivity인 것을 보여줌. **encrypted network이자만 public network인 internet을 사용한다.**
- AWS Direct Connect by itself cannot provide an encrypted connection between a data center and AWS Cloud : Direct Connect 자체는 encrypted connection과 관련이 없다. **encrypted network와 관련이 없지만, private network를 사용한다.**
- Site-to-site VPN + Direct Connect : IPsec-encrypted(by VPN) private connection(by DC) that also reduces network costs(인터넷에서 라우팅하는 것보다 private network를 사용해 라우팅하면 더 빠르기 때문에), increases bandwidth throughput(DC를 사용하면 Increase bandwidth throughput이 가능함)

#18. Transit Gateway
- 모든 aws 서비스 중에서 유일하게 multicast지원

#19. Kinesis Data Steams
- SNS+SQS Fan Out보다 **multiple applications to consume the same stream하는데에 유리함**
- 대표적 Use case 3가지
  - Routing related records to the same record processor : 연관된 레코드를 같은 프로세서에 전달
  - Ordering : immediate이든 a few hours later이든 데이터의 순서보장가능
  - Ability for multiple applications to consume the same stream concurrently. For example, you have one application that updates a real-time dashboard and another that archives data to Amazon Redshift. You want both applications to consume data from the same stream concurrently and independently. : Kinesis Data Steams은 **multiple applications to consume the same stream in real-time에 최적화됨**
- **partition key부분 강의자료 다시 볼 것**

#20. Route53 : Host-based Routing
- You can route a client request based on the **Host field** of the **HTTP header allowing you to route to multiple domains** from the same load balancer. : 서로 다른 도메인 간 라우팅도 가능하다.

#21. FSx for Windows File Server(실전테스트1 - 41)
- support **DFS(Distributed File System), SMB Protocol** 사용
- **S3 objects as files and does not allow** you to write changed data back to S3.
- **user quotas, end-user file restore**
- AWS Managed Microsoft AD, FSx for Lustre는 DFS를 support하지 않는다.

#22. AMI는 ebs snapshot이 베이스이다.
- 따라서 리전 간 ami copy시 복사할 리전에는 AMI, EBS Snapshot 2개가 생성된다.

#23 s3는 prefix당 3,500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD이 가능하고 prefix의 개수는 무제한이다.
- 따라서 customer-specific custom prefixes을 사용하면 사실상 request에 제한이 없다.

#24 Kinesis
- Kinesis Data Streams는 firehose처럼 intermediary Lambda function을 사용할 수 없다.
- Kinesis Data Analytics는 streams와 달리 다양한 소스로부터 데이터를 전달 받을 수 없고, 보통 data streams 또는 data firehose로부터 전달받는다.

#25 S3에서 standard를 제외하면 minimum storage duration이 최소 30일으로 그 이후에 data transition이 가능하다.

#26 Lambda는 default로 동시에 최대 1000개까지 실행하고 그 이상을 원한다면 aws support에 문의해야 한다.

#27 WAF - Geo match conditions
- configure a whitelist that allows only viewers in those countries. 
- configure a blacklist so that end-users from those countries are blocked from downloading their software.
- WAF를 사용하면 cloudfront가 edge레벨(doesn't belong to VPC)에서 geo restriction하는 것과 달리 VPC - ALB레벨에서 geo match 화이트리스트, 블랙리스트를 사용할 수 있다.

#28 FSx For Lustre(실전테스트1 - 52)
- HPC, fast storage
- FSx for Lustre provides the ability to both process the 'hot data' in a parallel and distributed fashion as well as easily store the 'cold data' on Amazon S3
- **FSx for Lustre integrates with Amazon S3**

#29 인스턴스 레벨의 액세스 제한
- VPC security groups
- IAM policy
- EFS Access Points
  - When an NFS client mounts an EFS file system without using an access point, the user ID and group ID provided by the client is trusted. 
  - You can use EFS access points to override user ID and group IDs used by the NFS client. 
  - When users attempt to access files and directories, Amazon EFS checks their user IDs and group IDs to verify that each user has permission to access the objects
  - 즉 access point를 사용하지 않으면 권한 체크를 하지 않는데 access point를 사용하면 권한 체크를 한다.

#30 ECS Cost
- ECS with EC2 launch type is charged based on **EC2 instances and EBS volumes** used. 
- ECS with Fargate launch type is charged based on **vCPU and memory resources** that the containerized application requests

#31 EFS로의 region간 접근
- The spreadsheet on the EFS file system can be accessed in other AWS regions by using an inter-region VPC peering connection