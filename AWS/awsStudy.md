AWS Certified Solutions Architect Associate Certification SAA-C02 스터디
===========
#1 보안 자격 증명 
- Access Key는 루트 계정에 만들면 cli나 sdk로 루트 권한에 접근 하는것이므로 만들지 않는 것이 좋다.  
   
#2 IAM Roles 
- IAM Roles are like an user but they are intended to be used not by physical people,
- but instead they will be by AWS services.

#3 보안 관리
- Credential report : csv로 유저별 access 히스토리 제공
- Access Advisor : 권한 중에서 last accessed로 어떤 서비스가 필요없는 지 알 수 있음 
  
#4 IAM Policies
- JSON documents that define a set of permissions for making requests to AWS services, and can be used by IAM Users, User Groups, and IAM Roles
  
#5 IAM User Groups can contain IAM Users and other User Groups. << False
- IAM User Groups can contain only IAM Users. << True


#6 EC2 INSTANCE TYPE(33강)
- general : t class
- compute optimized(high performance) : c class
- memory optimized : r class
- storage optimized : I, D, H class

#7 보안 그룹
- 한 보안 그룹이 여러 개의 인스턴스에 부착 가능
- 한 인스턴스가 여러 보안 그룹을 가질 수도 있다. 
- it's good to maintain one separate security group for ssh access

#8 classic port
- 22 : ssh
- 21 : ftp
- 22 : sftp(ssh를 사용한 ftp)
- 80 : http
- 443 : https
- 3389 : rdp(remote desktop protocol) - log into a windows instance

#9 보안그룹을 인바운드룰에 추가하면 보안그룹이 부착된 인스턴스가 허용된다.  
따라서 ip나 port를 몰라도 인바운드가 허용될 수 있다.

#10 chmod 0400 << ssh 로그인 pem키 읽기 전용으로 권한 변경
맥에서 이걸 해야 ssh -i로 로그인 가능

#11 never do aws configure at ec2 instance
- 왜냐하면 aws configure로 ec2에 iam의 권한을 줘버리면 ec2에 접근하는 다른 권한을 가진 iam도 그 권한을 획득하기 때문이다.  
  
#12 
- instance ssh 접근 : pem key
- aws cli 설정 : access key id, secret access key
  
#13 aws ec2 purchasing option

#14 ec2 spot instances requests
- spot instance used for batch jobs, data analysis, or workloads **that are resilieent to failures**
- resilient to failures means that '실패해도 괜찮은 것' 정도?
  
#15 how to terminate spot instances
- 44강 참고

#16 Spot Fleets
- set of spot instances(optionaaly on-demand instance can be here)
- spot fleets allow us to automatically request spot instances with the lowest price

#17 machines connect to www using **NAT** + **internet gateway(proxy)**
    
#18 placement groups : 인스턴스들에 대해 물리적으로 그룹핑하는 것
- cluster
- spread
- partition >> hadoop, kafka에 사용 
- 대상 그룹을 만들고 인스턴스 생성 시 적용가능하다.
  
#19 elastic network interfaces
- logical component in VPC
- it represents a virtual network card
- it gives **ec2 instances** access to the network 
- also used out of ec2 instance(인스턴스 밖에서도 사용가능, but 나중에 다룸)
- 속성
  - 한 ec2가 primary private ipv4, one or more secondary ipv4를 가질 수 있음
  - each eni can have one elastic ip per private ipv4 or public ipv4
  - **one or more security groups 할당가능** 
  - a MAC address is attached
  - failover를 위해 eni를 **다른 ec2에 부착가능** 즉 eni는 이동가능하다는 것
  - **bound to** a specific availability zone(AZ)

#20 elastic network interface 설정
- description
- subnet >> **서울 availability zone에서 a,b,c,d 선택**
- ipv4할당 auto assign
- **security group 설정**

#21 EC2 Nitro
- it is next generation EC2 instance
- new virtualization technology
- higher speed EBS (MAX 64000 IOPS, whereas 32000 IOPS on none nitro / IOPS : io operation per second)
- better security 
- Virtualized와 Bare metal 모두 지원

#22 optimizing cpu option
- core나 스레드(vCPU)를 billing 때문에 줄일 수 있다.  

#23 spot block instance
- 예를 들면 batch 작업중 1~6시간 정도되는 시간에는 interrupt를 막을 수 있음

#24 EBS Volumes
- they can only be mounted to one instance at a time
- **bound to** a specific availability zone
- network drive이기 떄문에 io에 use network한다. 따라서 a bit of latency가 생긴다.
- snapshot은 az에 상관없이 옮길 수 있음
- can increase capacity of drive over time

#25 AMI
- creating an AMI will also create EBS snapshots
- like ebs, ami is built for a specific region and also can be copied across region

#26 EC2 hibernate
- EC2 instance root volume type must be an ebs volume

#27 EC2 Instance Store
- 물리적으로 ec2와 붙어있기 때문에 high io performance 기대 가능
- 하지만 instance를 stop하기만 해도 볼륨이 다 삭제됨
- 캐시, 버퍼, 일시적 데이터 저장에 사용
- It is ephemeral drive
- 256,000 IOPS 이상 기대가능 
- 물론 백업은 데브옵스에게 달려 있음

#28 EBS Volume Type
- gp2,3 : ssd / gp2 >> max 16,000 IOPS
- io1,2 : 성능 더좋은 ssd, great for **databases workloads** / 
  - io1 >> max 64,000 IOPS
  - io2 block express drive >> max 256,000 IOPS
  - 32000이상의 iops를 원한다면 nitro ec2의 io volume을 사용할 것
  - io1,2의 경우에는 multi-attach가 가능해서 여러 인스턴스가 한 볼륨을 사용하게 할 수도 있음
  - Using EBS Multi-Attach, you can attach the same EBS volume to multiple EC2 instances in the same AZ. 
  - Each EC2 instance has full read/write permissions.
  - **즉 EBS의 속성은 same az이어야 한다는 것이 적용된다.**
  - 그럴 경우 동시 write를 manage해야함
  - 그럴 경우 achieve **high application availability**
  - 그럴 경우 cluster aware한 file system을 써야 함(XFS, EX4는 안됨)
- st1, sc1 : hdd, cannot be a boot volume

#29 EBS Encryption
- EBS Encryption을 enable하면 모든 부분에서 암호화가 적용됨 모든 데이터, 볼륨, 스냅샷, 데이터 전송 시에도 암호화가 적용됨
- volume을 unencrypted로 생성, 또 이 volume에 대한 snapshot을 unencrypted로 생성하면
- snapshot copy본을 encrypted하게 만들고 이 encrypted된 snapshot으로부터 암호화된 volume을 생성가능
- 또는 unencrypted snapshot에서 바로 encrypted된 볼륨을 생성 가능
  
#30 EFS(Elastic File System)
- Managed NFS(network file system) that can be mounted on many EC2
- highly available, expensive, pay per use **반대로 EBS는 PROVISIONED된 만큼 지불**
- EFS works with EC2 instance in multi AZ **반대로 EBS는 bound to specific region**
- EFS에 대해 보안 그룹 설정이 필요함
- Attributes
  - uses security group to control access to EFS
  - Use cases : content management, web serving, data sharing, Wordpress
  - **Only compatible with Linux based AMI** not windows (POSIX 파일 시스템을 사용하기 때문)
  - scales automatically 
- Performance & Storage Classes
  - Performance mode
  - Throughput mode
  - Storage Tiers
  - 66강 필독
- EFS-IA(Infrequent Access)에는 기본 옵션으로 30일 동안 접근되지 않은 파일은 IA영역으로 옮겨져 관리됨(Storage Tier) to save some costs

#31 AMI
- AMIs are built for a specific AWS Region, they're unique for each AWS Region. 
- You can't launch an EC2 instance using an AMI in another AWS Region, 
- but you can copy the AMI to the target AWS Region and then use it to create your EC2 instances.
- 그러니까 us-east-1로 만든 AMI를 다른 지역에서 사용 불가능하니까 다른 지역에서도 사용가능하도록 AMI를 copy한 후 써야 한다는 뜻

#32 Scalability & High Availability
- Horizontal Scaling : Auto Scaling Group, Load Balancer
- High Availability : Auto Scaling Group Multi AZ, Load Balancer Multi AZ

#33 Application Load Balancer(v2)
- 클라이언트의 요청은 로드 밸런서를 통해 들어오므로 인스턴스가 원본 ip from을 모르므로 X-Forwarded-For(client ip), X-Forwarded-Port, X-Forwarded-Proto를 적어주어야 요청한 ip를 알 수 있다.
- ALB는 multiple한 target group을 가질 수 있지만 **하나의 포트에는 하나의 target group만 할당할 수 있다.**

#34 Target Group
- Instance
- Ip address(**Must be private IPs**)
- Lambda functions
- Load balancer

#35 Network Load Balancer
- NLB has one static IP per AZ, and supports assigning Elastic IP
- Performance is better than ALB
- 로드 밸런서 생성 시 AZ에 IP할당할 때, ALB는 assigned by AWS이지만
- NLB는 각 AZ당 elastic ip를 할당할 수 있음  
- **NLB는 ALB와 달리 보안 그룹을 설정하지 않는다.** 
- 따라서 HTTP를 이용해 대상 그룹 내부의 **인스턴스들의 응답을 받으려면 인스턴스들의 보안 그룹을 HTTP, 80포트를 from anywhere로 설정**해야 한다.  
- 즉 NLB는 TCP,TLS,UDP를 이용해 바로 대상 그룹에 전달하는 역할을 하는 것이다.  
- 겉에서 보기에는 로드 밸런서를 통해서 오는 것처럼 보이지 않고, 외부 클라이언트의 요청에서 오는 것처럼 보인다.

#36 Gateway Load Balancer
- in **IP protocol (layer 3 Network layer)**
- it is **transparent network gateway** becauase it has single entry and single exit
- 요청이 보안그룹같은 조건에 부합하지 않으면 drop
- 78강 다이어그램 참고
- it uses GENEVE protocol on port 6081

#37 Cross-Zone Load Balancing
- Classic Load Balancer
  - disabled by default
  - no charges for inter AZ
- Application Load Balancer
  - always on
  - no charges for inter AZ
- Network Load Balancer
  - disabled by default
  - pay charges for inter AZ

#38 SNI(Server Name Indication)
- CLB 미지원
- SNI로 인해 ALB,NLB는 multiple한 ssl인증서를 로드밸런서에 장착해 다른 도메인의 대상그룹으로 보낼 수 있다.

#39 Connection Draining
- CLB : Connection Draining
- ALB & NLB : Deregistration Delay
- 이걸 설정하면 새 요청은 거절하고 기존 요청은 안전하게 처리한 후 인스턴스를 종료할 수 있음

#40 Auto Scaling Group
- work with Load Balancer
- Automatic Scaling
  - dynamic scaling policy
    - target tracking policy : cpu에 사용량에 기반해 인스턴스를 늘릴거나 줄일 수 있음
    - step policy
    - simple policy
  - predicted scaling policy
    - 이전 사용 기록에 기반한 오토 스케일링, 머신러닝을 이용
  - scheduled actions
- Good Mertics To Scale On
  - CPUUtilization: Average CPU utilization across your instances
  - RequestCountPerTarget: to make sure the number of requests per EC2 instances is stable
  - Average Network In / Out (if you’re application is network bound)
  - Any custom metric (that you push using CloudWatch)
- After a scaling activity happens, you are in the cooldown period (default 300 seconds)
  - During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilize)
  - Advice: Use a ready-to-use AMI to reduce configuration time in order to be serving request fasters and reduce the cooldown period
  - 즉 scaliing할 시 cooldown 기간이 생겨 그 기간중 또 다른 scaling action이 생기면 무시한다는 의미
- ASG Default Termination Policy 
  - ASG tries the balance the number of instances across AZ by default
  - 즉 인스턴스가 많은 AZ부터 인스턴스를 제거하는 것이 기본 정책
- ASG lifecycle hooks
  - auto scaling group에서 인스턴스가 inService나 Terminated한 상태가 되기 전에 Wait, Proceed하는 기간에 스크립트를 실행하거나 요청을 처리할 수 있다.
- Launch Configuration (legacy) VS Launch Template (newer)
  - Launch Configuration (legacy)
    - Must be re-created every time
  - Launch Template (newer)
    - Can have multiple versions
    - Create parameters subsets (partial configuration for re-use and inheritance)
    - **Provision using both On-Demand and Spot instances (or a mix)**
    - Can use T2 unlimited burst feature
    - Recommended by AWS going forward
- 참고로 RequestCountPerTarget은 CPUUtilization과 달리 생성 기본 옵션에 없기 때문에 이걸 Scaling의 지표로 사용하려면 CloudWatch를 사용해서 custom metric을 만든 후 CloudWatch알람을 사용해야 한다.

#41 RDS
- can't ssh to an instance
- **RDS Backups**
  - 자동 백업 존재
  - 매일 모든 데이터가 백업됨
  - 백업데이터는 7일 동안 저장되고 35일까지 연장가능
  - 매 5분마다 트랜잭션 로그가 쌓임
- **DB Snapshot**
  - 유저가 직접 백업함
  - 보유 기간 무제한
- **Storage Auto Scaling**
  - Helps you increase storage on your RDS DB instance dynamically
  - When RDS detects you are running out of free database storage, it scales automatically
  - You have to set Maximum Storage Threshold 
  - Useful for applications with unpredictable workloads
- **Read Replicas**
  - up to 5 read replicas
  - within az, cross az, cross region
  - network cost : async replication to different az but same region no fee
  - network cost : async replication to different region will fee
  - replication is async
- **RDS Multi AZ (Disaster Recovery)**
  - **SYNC replication**
  - One DNS name – failover to standby(dns를 사용해서 rds인스턴스가 사용 불가능하면 자동 교체하기 때문에 사용자는 dns만 알면 됨)
  - Increase availability
  - Not used for scaling
  - **Important** : The Read Replicas be setup as Multi AZ for Disaster Recovery (DR)
- **From Single-AZ to Multi-AZ**
  - Zero downtime operation (no need to stop the DB)
  - Just click on “modify” for the database
- Multi AZ와 Read Replica의 큰 차이점은 sync와 async라는 특징

#42 RDS Security
- 마스터가 암호화되어 있지 않으면, 레플리카는 암호화할 수 없다.
- rds에서의 enforcec ssl
  - postgre : parameter group 사용
  - mysql : sql command 사용(grant user ... require ssl)
- 91강 summary 읽어보기

#43 Aurora
- aurora 구조 92강
- aurora hands on 93강 : replica사용 안해도 storage는 replica는 3개의 az에 저장된다. that's guarantee.

#44 Aurora Replicas - Auto Scaling

#45 Section 9 : quiz 6 - 5번, 8번, 11번

#46 Route 53
- Multi Value policy와 Simple policy의 차이점은 Multi Value는 health check가 가능하다는 것
- 반면 simple policy는 여러 값들을 리턴하고 클라이언트는 그 중 랜덤한 값을 받아서 사용한다.
- Health Check 3가지
  - Monitor on EndPoint
  - Calculated Health Check
  - Cloud Watch Alarm을 모니터링하는 Health Check / Private Endpoint같은 곳은 health check이 불가능하므로 cloudwatch metric을 사용해 연동

#47 S3
- s3 versioning : delete marker가 존재해서 restore가 가능함
- 135강 s3 CORS 이론 읽어보기
- Explicit DENY in an IAM Policy will take precedence over an S3 bucket policy.
- IAM Policy가 s3 bucket policy에 우선하게 하려면 explicit deny를 iam policy에 적용하면 된다.

#48 S3 Storage Classes : One Zone IA를 제외한 모든 클래스는 3개 이상의 az를 가진다. 
- S3 Standard
- S3 Intelligent Tiering : S3 Intelligent-Tiering은 액세스 패턴이 변경될 때 두 액세스 티어(Frequent Access 및 Infrequent Access) 간에 데이터를 이동시켜 비용을 자동으로 절약해 주는 최초의 클라우드 객체 스토리지 클래스로서, 액세스 패턴을 알 수 없거나 액세스 패턴이 변경되는 데이터에 적합
- S3 Standard IA
- S3 One Zone IA : 1개의 az를 사용하기 때문에 다른 클래스에 비해서 낮은 availability를 가짐
- S3 Glacier : 빠르면 분단위에도 retrieve할 수 있지만, standard는 3 to 5 hours
- S3 Glacier Deep Archive : bulk옵션 선택 시 retrieve에 최대 48시간 걸림, standard는 12 hours

#49 S3 Lifecycle Rules
- S3 시나리오를 주고 라이프 사이클을 어떻게 구성할 것인지 물어보는 문제 자주 출제 p.311

#50 S3 Analytics
- S3 분석을 제공해서 이 옵션을 활성화하면 분석에 근거해서 라이프 사이클을 며칠을 기준으로 해야할 지 결정할 수 있다.
- Standard to Standard_IA로만 가능하다.
- Onezone_IA, Glacier는 제공하지 않는다.

#51 S3 Select & Glacier Select
- Retrieve less data using SQL by performing server side filtering
- 간단한 파일 타입 정도 ex).csv를 쿼리 가능하면 복잡한 쿼리는 s3 serverless인 athena에서 다룬다.

#52 S3 Baseline Performance
- Your application can achieve at least 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix in a bucket. 
- 즉 4개의 prefix를 사용한다면 22,000 requests per second for GET and HEAD의 성능을 가진다고 볼 수 있다.
- Example (object path => prefix):
  - bucket/folder1/sub1/file => /folder1/sub1/
  - bucket/folder1/sub2/file => /folder1/sub2/
  - bucket/1/file => /1/
  - bucket/2/file => /2/

#52 S3 Event Notifications
- 예시로 동영상을 s3에 업로드하면 sns, sqs, lambda에 알림을 주는 용도로 사용할 수 있다.
- 이벤트 발생 시점은 오브젝트 업로드나, 삭제, 복제 등으로 설정가능하다.
- 언제 notification이 발생하는 지, rule을 커스텀할 수 있다.
