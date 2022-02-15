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
- io2 : max 64,000IOPS for nitro(nitro가 아닌 것은 최대 32,000IOPS), size는 4GiB to 16TiB
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
- HPC, fast storage : FSx For Lustre는 고성능 파일 시스템이다.
- FSx for Lustre provides the ability to both process the 'hot data' in a parallel and distributed fashion as well as easily store the 'cold data' on Amazon S3
- **FSx for Lustre integrates with Amazon S3**
- 다른 서비스(ex. EFS에 비해 비싸다)
- 컴퓨팅 파워만큼 빠른 스토리지 성능을 위해서 사용

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

#32 ec2 user data
- ec2 user data는 최초 부팅 시에만 실행됨
- ec2 user data는 default로 root 권한을 가짐

#33 By default, an S3 object is owned by the AWS account that uploaded it. So the S3 bucket owner will not implicitly have access to the objects written by Redshift cluster : a계정이 b계정의 s3버켓에 파일을 업롣드하면 버켓을 소유한 b계정은 기본적으로 그 파일에 접근할 수 없다. 파일을 올린 계정이 아니기 때문이다.

#34 IAM permission boundary. They can only be applied to roles or users, **not IAM groups**

#35 ASG가 unhealthy instance를 terminate하지 않을 때
- The health check grace period for the instance has not expired : grace period라고 terminate하지 않는 유예 기간이 있다.
- The instance maybe in Impaired status - 손상된 status이면 recover할 시간을 주기 때문에 terminate하지 않는다.
- The instance has failed the ELB health check status - By default, Amazon EC2 Auto Scaling doesn't use the results of ELB health checks to determine an instance's health status when the group's health check configuration is set to EC2 : 즉 asg는 기본적으로 elb health check보다 ec2 health check을 우선순위로 두기 때문에 elb health check이 fail하더라도 ec2 health는 정상일 수 있다는 것이다.

#36 cognito user pool vs cognito identity pool
- 둘 다 소셜 로그인, SAML을 지원한다.
- **Cognito user pool**
  - 애플리케이션에 회원가입, 로그인, 소셜 로그인 기능을 부착가능. 즉 앱을 사용할 때 인증을 위해서 사용한다.
- **Cognito identity pool**
  - Amazon S3 및 DynamoDB같은 aws 서비스에 액세스하기 위해 임시 자격 증명을 얻을 때 사용한다.

#37 Use Cognito Authentication via Cognito User Pools for your Application Load Balancer : true
- Use Cognito Authentication via Cognito User Pools for your CloudFront distribution : You cannot directly integrate Cognito User Pools with CloudFront distribution as you have to **create a separate Lambda@Edge** function to accomplish the authentication via Cognito User Pools. 

#38 Kinesis
- Kinesis firehose는 fully managed인데 비해 Kinesis Data Streams는 shard를 provision해야 한다.

#39 Spot Fleet Request는 spot instance들을 요청하는 것이지 asg처럼 유동적으로 인스턴스를 terminate하고 create하는 것이 불가능하다.

#40 video는 rds가 아닌 s3에 적합

#41 **MAX I/O performance mode in EFS**
- 지연 시간이 늘어나지만, 병렬화된 애플리케이션에 특화되고, 더 많은 throughput 지원

#42 Kinesis firehose의 source를 kinesis data streams로 사용중이라면, kinesis agent는 kinesis firehose에 direct하게 데이터를 전달할 수 없다.
- 따라서 이 경우 kinesis agent는 데이터를 kinesis data streams에 추가해야 한다.
- 기본적으로 kinesis agent는 kinesis firehose, kinesis data streams에 데이터를 전달할 수 있다.

#43 **can't move data directly from Snowball into a Glacier Vault or a Glacier Deep Archive Vault. You need to go through S3 first**
- 스노우볼에서 glacier로 데이터를 옮기려면 먼저 s3로 옮긴 후 lifecycle로 옮기는 것이 일반적이다.

#44 You can only use a **launch template** to provision capacity across multiple instance types using both On-Demand Instances and Spot Instances to achieve the desired scale, performance, and cost
- launch template : versioning가능, provision capacity across multiple instance types using both On-Demand Instances and Spot Instances가능
- launch configuration : 위 2가지 불가능

#45 Create an encrypted snapshot of the database, share the snapshot, and allow access to the **AWS Key Management Service (AWS KMS) encryption key**
- You can share the AWS Key Management Service (AWS KMS) customer master key (CMK) that was used to encrypt the snapshot with any accounts, that you want to be able to access the snapshot. You can share AWS KMS CMKs with another AWS account by adding the other account to the AWS KMS key policy.
- CMK에 대한 액세스는 여러 계정 간에 공유하는 것도 가능하다.
- Making an encrypted snapshot of the database by CMK will give the auditor a copy of the database 왜냐하면 AWS KMS key policy를 바꿔서 키에 대한 접근 권한을 공유했기 때문이다.

#46 "aws:RequestedRegion": "eu-west-1"은 api call이 만들어진 곳 기준이 아니라, **instance가 어느 리전에 존재하는지**를 기준으로 한다.

- 아래는 policy이다.
```text
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Mystery Policy",
      "Action": [
        "ec2:RunInstances"
      ],
      "Effect": "Allow",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "eu-west-1"
        }
      }
    }
  ]
}
```

#47 블루 그린 배포를 하는데 DNS캐싱이 발생할 경우, Global Accelerator가 dns 캐싱을 해결해 줄 수 있다.
- Use AWS Global Accelerator(**multi-Region solution**) to distribute a portion of traffic to a particular deployment
- "AWS Global Accelerator를 사용하면 클라이언트 디바이스와 인터넷 리졸버에서 DNS 캐싱에 종속되지 않고 트래픽을 점진적으로 또는 모두 한 번에 이동할 수 있으며, 트래픽 다이얼 및 끝점 가중치 변경은 몇 초 내에 적용됩니다."

#48 Secrets manager는 변수 자동 로테이션 기능을 지원하지만, SSM Paramter Store는 자동 로테이션 기능을 지원하지 않고, 수동으로 돌려야 한다.

#49 
- A developer needs to implement a Lambda function in AWS account A that accesses an Amazon S3 bucket in AWS account B.
- 위 상황에 필요한 두 가지 설정은 아래와 같다.
- S3 버킷에 액세스할 수 있는 람다 기능에 대한 IAM 역할을 만든다. 
- **IAM 역할을 람다 기능의 실행 역할**로 설정합니다. 
- **버킷 정책이 람다 함수의 실행 역할에 대한 액세스 권한도 부여**해야 한다.

#50 Instance가 종료된 후에도 EBS Volume을 유지하는 방법
- Set the DeleteOnTermination attribute to false
- 위와 달리 **ec2 hibernate는 in-memory state를 유지하게 한다.** : hibernate를 사용하면 in-memory의 내용을 ebs에 저장하기 때문에 가능한 것이다.

#51 SQS에서 group id를 사용하지 않으면 consumer는 only one이다.

#52 Kinesis Data Streams에 비해 SQS FIFO는 consumers를 늘리기 효율적이다(한 SQS에 최대 100개의 consumer).

#53 **IAM** Account level, User level 액세스 권한
- https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html(IAM 역할을 사용한 AWS 계정 간 액세스 권한 위임) 참고
- 위 문서를 읽어보면 계정 간 권한 위임을 IAM Policy를 사용한다.(AssumeRole, Security Token Service)

#54 **S3** Account level, User level 액세스 권한

|TYPE|Account Level|User Level|
|------|---|---|
|IAM Policies|No|Yes|
|ACLs|Yes|No|
|Bucket Policies|Yes|Yes|

- **Bucket Polices는 ip를 기준**으로도 제한 가능

#55 Elastic Load Balancing does not work across regions : ELB는 한 리전에 국한된 서비스이다.

#56
- Does S3 bucket policy override IAM policy? : S3 bucket policy가 iam policy를 무시하고 덮어씌우는 것이 가능한가?
- Yes it can indeed override the policy, but only where it uses a Deny. If it includes an Allow but the IAM policy includes a Deny this will not evaluate as Allow. : deny에 한해서 가능하다. iam policy가 deny인데 bucket policy가 allow한다고해서 override되지 않는다.

#57 storage gateway는 on-premise및 cloud의 하이브리드 환경을 사용하는 사내에 **데이터를 캐싱**하는 기능도 한다.

#58 PostgreSQL의 기본 포트는 5432이다.

#59 Batch job은 spot instance가 비용면에서 최적화되어 있다.

#60 Partition placement group은 Hadoop, 카산드라, 카프카 같은 대규모 데이터 분산 작업에 사용된다.

#61 ASG Default Termination Policy
- 1순위로 제일 먼저 terminate되는 대상 : 오래된 launch configuration
- 2순위로 terminate되는 대상 : 오래된 launch template
- 3순위로 terminate되는 대상 : closest to next billing hour - 이는 시간 단위로 청구되는 linux, ubuntu ec2 usage cost를 줄여준다.

#62 CloudFormation은 리소스를 프로비저닝하는 데 시간이 걸리기 때문에 특정 사용 사례에 최소량의 다운타임이 필요한 경우에는 적절한 솔루션이 아니다.

#63 Reserved Instance vs Spot Instance
- Reserved Instance는 지속적인 사용에 효율적
- Spot Instance는 monthly work에 효율적
- Spot Instances are well-suited for **data analysis, batch jobs, background processing, and optional tasks**
- Amazon EC2 needs the capacity back일 때, Spot instance가 종료될 수 있는데 이 때 공식문서에서는, "Amazon EC2 automatically resubmits a persistent Spot Instance request after the Spot Instance associated with the request is terminated"라고 말하며 다른 spot instance를 자동으로 요청한다.
- 즉 **Amazon EC2 needs the capacity back 또는 Spot price exceeds the maximum price for your request일 때 spot instance는 terminate**된다.

#64 Dedicated Host는 Dedicated Instance에 비해 cost가 많이 들어 cost-effective하지 않다.

#65 실전 테스트 2 - 63번 bucket policy 문제 있음

#66 Shared Service VPC
- 실전 테스트 2 - 65번
- 한 회사가 AWS 계정을 여러 개 운영하고 있으며 AWS Transit Gateway를 사용하여 허브 앤 스포크 방식으로 이들 계정을 상호 연결했습니다. 네트워크 분리를 용이하게 하기 위해 이러한 AWS 계정 전체에 VPC가 프로비저닝되었습니다. VPC의 워크로드에 필요한 서비스에 대한 공유 액세스를 제공하면서 **관리 오버헤드와 비용**을 모두 줄일 수 있는 솔루션은 무엇입니까?
- Sharing resources from a central location instead of building them in each VPC may reduce administrative overhead and cost : 각 vpc에서 자원을 만들어 공유하는 것이 아니라 Shared Service VPC를 만들어 사용한다.
- https://aws.amazon.com/ko/blogs/architecture/reduce-cost-and-increase-security-with-amazon-vpc-endpoints/(Amazon VPC 엔드포인트로 비용 절감 및 보안 강화)

#67 When you publish a high-resolution metric, CloudWatch stores it with a resolution of 1 second, and you can read and retrieve it with a period of 1 second, 5 seconds, 10 seconds, 30 seconds, or any multiple of 60 seconds : high-resolution metric을 사용하면 1초 간격으로 쌓고, 1~60초 간격으로 retreive할 수 있다.

#68 EC2 Detailed Monitoring은 1분 간격으로 metric 모니터링 가능

#69 Endpoint on Route 53 Resolver
- Create an inbound endpoint on Route 53 Resolver and then DNS resolvers on the on-premises network can forward DNS queries to Route 53 Resolver via this endpoint : Route 53 Resolver의 인바운드 엔드포인트는 온프레미스의 DNS resolver가 Route 53 Resolver에 쿼리를 요청할 수 있게 한다.
- Create an outbound endpoint on Route 53 Resolver and then Route 53 Resolver can conditionally forward queries to resolvers on the on-premises network via this endpoint : Route 53 Resolver의 아웃바운드 엔드포인트는 조건적으로 온프레미스 네트워크의 DNS resolver에 쿼리를 요청할 수 있게 한다.
- inbound endpoint 방향 : on-premises DNS resolvers > Route 53 Resolver
- outbound endpoint 방향 : Route 53 Resolver > on-premises DNS resolvers
- 참고로, dns resolver가 dns server로 쿼리를 보내 응답을 요청하는데 dns resolver > local dns server > (root dns server, tld dns server, sld dns server) 이 순서로 요청이 이루어진다고 보면 된다.

#70 Aurora Global Database
- Short Recovery Time(RTO)에 특화
- Managed planned failover – 자동으로 failover를 실행
- Unplanned failover - 직접 failover실행하기 때문에 RTO가 길어질 수 있음

#71 AWS Elastic Beanstalk
- **full control** over the AWS resources powering your application and **can access the underlying resources** at any time

#72 Dedicated Hosts enable you to use your existing server-bound software licenses like **Windows Server and address corporate compliance and regulatory requirements** : dedicated host는 dedicated instance와 달리 온프레미스 서버의 소프트웨어 라이센스를 dedicated host에 똑같이 적용시킬 수 있다.

#73 Configure an Amazon CloudWatch alarm that triggers the recovery of the EC2 instance, in case the instance fails. **The instance can be only configured with EBS volume** - The recover action is supported only on instances that have EBS volumes configured on them, instance store volumes are not supported for automatic recovery by CloudWatch alarms. : ebs가 아닌 instance store를 사용하는 경우 자동 복구를 지원하지 않는다.

#74 ALB와 EC2 Instances를 사용하는데 너무 많은 ALB사용으로 구조가 복잡해졌을 경우
- The architecture has now become complex with too many ALBs in multiple AWS Regions. Security updates, firewall configurations, and traffic routing logic have become complex with **too many IP addresses and configurations**.
- 해결책 : Launch AWS Global Accelerator and create endpoints for all the Regions. Register the ALBs of each Region to the corresponding endpoints