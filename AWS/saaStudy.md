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
- Managed **NFS(network file system)** that can be mounted on many EC2
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
- EBS Multi-attach는 single-az의 인스턴스만 지원하지만 EFS는 multi-az의 인스턴스와 통합 가능하다. 
- Uses **security group** to control access to EFS
- Compatible with Linux based AMI(not Windows) : 윈도우 ami는 지원하지 않는다.

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
- **NLB has one static IP per AZ, and supports assigning Elastic IP** : 네트워크 로드밸런서는 az당 하나의 고정 IP를 갖고 elastic ip 할당을 지원한다.
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
- it uses **GENEVE protocol on port 6081**

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
- Use a **ready-to-use AMI** to reduce configuration time in order to be serving request fasters and reduce the cooldown period : **ready-to-use ami**를 사용하면 설정 시간을 줄여 cooldown period를 줄일 수 있다.
- ASG tries the balance the number of instances across AZ by default : AZ간 인스턴스 수에 균형을 맞추는 것이 default이다.

#41 RDS
- **can't ssh** to an instance
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
  - When RDS detects you are running out of free database storage, **it scales automatically**
  - **You have to set Maximum Storage Threshold** 
  - Useful for applications with unpredictable workloads
- **Read Replicas**
  - up to 5 read replicas : 최대 5개의 replica 생성 가능
  - within az, cross az, cross region
  - network cost : async replication to **different az but same region** no fee
  - network cost : async replication to **different region** will fee
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
- **Multi-AZ replication is free**
- **Encryption has to be defined at launch time**
- **IAM-based authentication** can be used to login into RDS MySQL & PostgreSQL
- RDS - IAM Authentication으로 얻을 수 있는 장점
  - 참고 : aws rds generate-db-auth-token << 이런 명령어를 사용함
  - Network in/out must be encrypted using **SSL**
  - IAM to centrally manage users instead of DB : rds 서비스로부터 토큰 발급을 통해 IAM policy으로 만들어진 rds접근 권한을 가지므로 db접근 권한을 db가 아닌 IAM이 관리해 관리를 중앙화할 수 있다.
  - Can leverage IAM Roles and EC2 Instance profiles for easy integration

#42 RDS Security
- 마스터가 암호화되어 있지 않으면, 레플리카는 암호화할 수 없다.
- rds에서의 enforce ssl
  - postgre : parameter group 사용
  - mysql : sql command 사용(grant user ... require ssl)
- 91강 summary 읽어보기

#43 Aurora
- aurora 구조 92강
- aurora hands on 93강 : replica사용 안해도 storage는 replica는 3개의 az에 저장된다. that's guarantee.
- aurora의 **shared storage volume**은 master, read replica가 공유하고, 10GB to 64TB까지 자동 확장된다.
- One Aurora Instance takes writes(master)
- Automated failover for master in **less than 30 seconds**
- Master + up to 15 Aurora Read Replicas serve reads : 마스터 1개 + read replica 최대 15개 = 총 16개
- Support for **Cross Region Replication**
- 6 copies of your data across 3 AZ : 약간의 인스턴스가 fail해도 상관없다는 것을 보여줌
  - 4 copies out of 6 needed for writes : 쓰기에 6개 중 최소 4개의 카피본이 필요
  - 3 copies out of 6 need for reads : 읽기에 6개 중 최소 3개의 카피본이 필요
- **Aurora Security**
  - **Possibility to authenticate using IAM token (same method as RDS)**
- **Aurora Cross Region Read Replicas**
  - Useful for disaster recovery : 재해 복구에 유용하다.
- **Aurora Global Database (recommended)** : 재해 복구에 **Aurora Cross Region Read Replicas**보다 더 유용하다.
  - 1 Primary Region (read / write) 
  - Up to **5 secondary(read-only) regions***, replication lag is less than 1 second
  - Up to **16 Read Replicas per secondary region**
  - **Helps for decreasing latency** : 모든 리전에서 빠른 접근 가능
  - Promoting another region (for disaster recovery) has an RTO of < 1 minute 

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
- If uploading more than **5GB**, must use “multi-part upload” : s3에 업로드하는 오브젝트가 5GB 이상이면 multi-part upload를 사용해야 한다. 
- **Cloudfront는 1GB 미만인 static 자원을 캐시하기 적합하다. 1GB 이상인 자원에 대해서는 S3 Transfer Acceleration(Cloudfront와 마찬가지로 글로벌한 서비스이기 때문에 글로벌한 애플리케이션에 적합)을 사용하는 것이 좋다.**
- s3 versioning : delete marker가 존재해서 restore가 가능함
- 135강 s3 CORS 이론 읽어보기
- Explicit DENY in an IAM Policy will take precedence over an S3 bucket policy.
- IAM Policy가 s3 bucket policy에 우선하게 하려면 explicit deny를 iam policy에 적용하면 된다.
- 99.999999999% durability / 99.99% availability : durability는 추후 언젠가는 데이터를 받을 수 있음을 의미하고, availabilty는 즉시 데이터를 받을 수 있음을 의미한다.

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

#51 S3 Baseline Performance
- Your application can achieve at least 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix in a bucket. 
- 즉 4개의 prefix를 사용한다면 22,000 requests per second for GET and HEAD의 성능을 가진다고 볼 수 있다.
- 예시 : object path == prefix:
  - bucket/folder1/sub1/file => /folder1/sub1/
  - bucket/folder1/sub2/file => /folder1/sub2/
  - bucket/1/file => /1/
  - bucket/2/file => /2/
- 위에서 볼 수 있듯이 위 4개는 전부 다른 prefix이다.

#52 S3 Performance
- p.315, p.317 그림 참고
- Multi-Part upload
  - **recommended for files > 100MB**
  - **must use for files > 5GB**
  - **Can help parallelize uploads** (speed up transfers) : 파일을 여러 부분을 잘라 나눈 후 동시에 보낸다.
-  S3 Transfer Acceleration
  - Increase transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region  
  - 가까운 region의 edge location에 보내고 aws가 알아서 private경로를 통해서 목표 region의 bucket에 보낸다는 것이다.
  - 따라서 업로드 속도 향상을 기대할 수 있다.
  - **Compatible with multi-part upload**
  - edge location은 cloudfront에서 다시 다룰 듯하다.
- S3 Performance – S3 Byte-Range Fetches
  - Parallelize GETs by requesting specific byte ranges
  - Better resilience in case of failures
  - 파일을 바이트 기준으로 부분으로 나누어 보내거나, 파일에서 첫 몇 바이트만 가져올 수 있다.

#53 S3 Select & Glacier Select
- Retrieve less data using SQL by performing server side filtering
- 간단한 파일 타입 정도 ex).csv를 쿼리 가능하면 복잡한 쿼리는 s3 serverless인 athena에서 다룬다.

#54 S3 Event Notifications
- 예시로 동영상을 s3에 업로드하면 sns, sqs, lambda에 알림을 주는 용도로 사용할 수 있다.
- 이벤트 발생 시점은 오브젝트 업로드나, 삭제, 복제 등으로 설정가능하다.
- 언제 notification이 발생하는 지, rule을 커스텀할 수 있다.

#55 S3 Request Pays
- s3에 파일을 요청하는 사람이 네트워크 비용을 지불하게 할 수 있다.
- aws에 인증된 사람만 가능(must not be anonymous)

#56 Amazon Athena
- **Serverless query service to perform analytics** against S3 objects
- 유형 문제 : analyze data in S3 using serverless SQL, use Athena
- 서버리스이기 때문에 db를 직접 만들고 관리할 필요 없이, rdb를 GUI로 만들 수 있고 sql을 사용가능하다.
- 여러 정보가 저장되기 때문에 s3 접근 시간, http status등 다양한 액세스 정보를 저장하고 조회할 수 있다.

#57 S3 Object Lock
- WORM (Write Once Read Many) model 구현
- **versioning이 활성화된 상태에서만 동작**
- Object retention(보유 기간)
  - Retention Period: specifies a fixed period
  - Legal Hold: same protection, no expiry date
- Mode:
  - Governance mode: 루트 계정은 모드 변경 가능, 파일 변경 가능
  - Compliance mode: 한 번 설정하면 루트 계정이라고 모드 변경 불가, 파일 변경 불가

#58 CloudFront vs S3 Cross Region Replication
- CloudFront를 사용해야 할 곳
  - Great for **static content** that must be available everywhere
- Cross Region Replication을 사용해야 할 곳
  - great for **dynamic content** that needs to be available at low-latency in few regions
  - 단점으로는 CloudFront와 달리 region마다 일일이 세팅해줘야 함

#59 CloudFront Signed URL / Signed Cookies
- 단일 자원에 액세스하기 위한 특정 회원만을 위한 URL
- Multiple 자원에 액세스하는 특정 회원만을 위한 URL(one signed cookie for many files)

#60 CloudFront – Origin Groups
- high availability를 위해 origin을 여러 개 둘 수 있음

#61 CloudFront – Field Level Encryption

#62 Unicast IP vs Anycast IP
- AWS Global Accelerator는 Anycast IP를 사용해서 같은 ip중 geo location이 더 가까운 쪽으로 라우팅되는 방식을 채택

#63 CloudFront vs Global Accelerator
- CloudFront uses Edge Locations to cache content while Global Accelerator uses Edge Locations to find an optimal pathway to the nearest regional endpoint. 
- CloudFront is designed to handle HTTP protocol meanwhile Global Accelerator is best used for both HTTP and non-HTTP protocols such as TCP and UDP.

#64 AWS Snow Family
- 몇십 테라바이트부터 시작하는 데이터들을 빠르게 옮기고 싶을 때 사용하는 것으로 물리적 장치를 사용해 속도를 높인다.
- 즉, 네트워크를 통해 s3같은 저장소에 옮기는 것이 아니라 직접 aws에 하드웨어적으로 갈아끼우는 것이라고 생각하면 편하다.
- 보통 데이터 마이그레이션에 일주일 이상이 걸릴 때 사용하면 좋다.
- Snowcone - 수십 테라바이트 마이그레이션에 사용
- Snowball Edge - 수십 페타바이트 마이그레이션에 사용
- Snowmobile - 수십 엑사바이트 마이그레이션에 사용

#64 Snow Family Edge Computing
- 오지에서 클라우드 컴퓨팅을 사용하기 위해서 snow family를 사용할 수 있고, snowcone같은 물리적 장치에 접근할 때 OpsHub를 사용한다.

#64 Snowball into Glacier
- snowball에서 바로 glacier로 옮길 수 없고, s3로 옮긴 후 s3 lifecycle을 이용해서 glacier로 옮기는 것이 제일 좋은 방법이다.

#65 SQS - Standard Queue
- FIFO가 아닌 Standard버전의 특성
- At least once delivery : 한 번만 receive하는 것을 보장한다.
- Consumers receive and process messages in parallel : 병렬의 consumer가 존재할 때 한 메시지는 한 consumer에게만 전달된다.(한 번만 받는 것 보장)
- Consumers delete messages after processing them
- We can scale consumers horizontally to improve throughput of processing
- FIFO Queue와 달리 throughput에 제한이 없다.
- 낮은 지연시간(publish, receivee0하는데에 10ms밖에 걸리지 않는다.)
- 각 메시지는 256KB로 제한
- 메시지는 기본 4일에서 최대 14일까지 보관가능

#66 SQS - Message Visibility Timeout
- 387p. consumer그룹 내에서 한 consumer가 메시지를 consume하면 다른 consumer는 visibility timeout기간 동안 메시지를 볼 수 없다.
- 즉 한 consumer가 소비하는 동안 visibility timeout기간(사용자 설정가능)동안 다른 consumer의 접근을 막는 것이다.
- visibility timeout기간 이후 message가 delete되지 않았을 때 다시 메시지를 consume할 consumer결정 방식은 polling중인 consumer중 랜덤으로 결정되는 것 같다.

#67 SQS - Dead Letter Queue
- sqs를 두 개 만들고 하나는 dead letter queue전용 큐로 만들 수 있다.
- 여기서 dead letter이라 함은, 한 message에 대해서 threshold(사용자 설정 가능)만큼 consumer가 receive했음에도 불구하고, message가 정상적으로 process되지 않은 것을 말하고 여기에서의 message가 dead letter이다.

#68 SQS - Long Polling
- Long Polling **decreases the number of API calls** made to SQS while **increasing the efficiency and latency** of your application.
- Long Polling은 메시지가 수신될 때까지 길게 기다리겠다는 의미(보통 20초로 설정)이다. 
- 메시지가 없다면 몇 번이고 계속 요청하는 상황이 발생할 수 있는데 Long Polling으로 이를 방지할 수 있다.
- Increasing the efficiency and latency : 오래 기다릴 수 있기 때문에 당연하게도 효율성이나 지연시간이 늘어날 수 있다.

#69 SQS - Request-Response Systems
- 391p. Request-Response Systems을 구현하는 클라이언트가 존재한다.
- SQS Temporary Queue Client라는 클라이언트(자바로 구현)를 사용하면 Request-Response Systems를 구현할 수 있고 내부적으로 이 시스템을 구현하는 가상의 큐들이 만들어진다.  

#70 SQS - FIFO Queue
- Standard Queue가 최소 한 번 이상의 delivery를 보장하는 것과 달리, FIFO Queue는 정확히 한 번의 delivery를 보장한다.(by removing duplicates)
- throughput이 300msg per second(batching을 사용하면 3000msg per second)로 standard에 비해 적다.

#71 SNS - Simple Notification Service
- SQS와 달리 pub/sub모델을 사용해서 topic을 발행하면 여러 구독자가 읽는 방식이다. topic은 SNS가 발행하는 메시지라고 보면 된다.
- 최대 100,000개의 토픽과 10,000,000개의 구독자를 감당할 수 있다.
- 구독자는 SQS, HTTP/HTTPS를 사용한 애플리케이션, Lambda 등이 될 수 있다.
- AWS Service중에서도 SNS로 데이터를 보낼 수 있는 서비스들이 다수 존재한다.
  - CloudWatch for alarm, Auto Scaling Group Notifications, S3 Bucket Events 등
- Message Filtering이 가능해서 구독자별로 받고 싶은 메시지를 선별해서 받을 수 있다.(ex. 특정 문자가 들어간 메시지만 받고 싶은 경우)

#72 SNS + SQS: Fan Out
- Push once in SNS, receive in all SQS queues that are subscribers : SNS에 메시지를 푸시하면 SQS 구독자들이 메시지를 받는 형태이다.
- 기존 SQS만 사용할 때는 하나의 메시지를 하나의 consumer(SQS)가 가져갔지만, sns를 사용함으로써 하나의 메시지를 여러 구독자들이 소비하는 형태가 가능해졌다.
- 마찬가지로 S3 Event(ex. object creation)를 여러 SQS에 보내고 싶다면 fan out 패턴을 사용해, SNS에서 메시지를 받고 SNS에서 여러 구독자에게 보낼 수 있다.

#73 SNS – FIFO Topic
- SNS역시 SQS와 마찬가지로 Standard형태와 FIFO형태가 있고, 거의 비슷한 특징을 지닌다.
- 다만 FIFO형태의 SNS는 FIFO형태의 SQS만 구독자로 가질 수 있다. 왜냐하면 FIFO Queue방식으로 전달되는 메시지를 FIFO로 처리하려면 FIFO형태의 SQS가 사용될 수 밖에 없기 때문이다. 

#74 Kinesis
- Makes it easy to collect, process, and analyze streaming data in real-time : 데이터 수집, 처리, 분석을 실시간으로 가능하게 해주는 서비스

#75 Kinesis Data Streams
- capture, process, and store data streams : 데이터 캡쳐, 처리, 저장을 지원하는 서비스
- Kinesis Data Streams에는 Record를 받는 shard가 존재하고, shard들을 모아놓은 Stream이 있다.
- Producer가 만드는 Record는 1. Partition key, 2. Data Blob으로 구성된다.(Kinesis Data Streams에 전달)
- Kinesis Data Streams가 만드는 Record는 1. Partition key, 2. Data Blob 3. Sequence no으로 구성된다.(Consumers에게 전달)
- Partition key가 존재하는 이유 참고 : [Kafka 사용시 주의점](https://allover3773.gitbook.io/study/kafka/kafkapoint)
- throughput은 1MB/sec 또는 1000msg/sec이 가능하고, 이 throughput은 per shard이기 때문에 30개의 shard는 30MB/sec의 throughput을 가진다.
- Retention between 1 day (default) to 365 days : 데이터가 Kinesis Data Streams에 최소 1일 이상 쌓이기 때문에 쌓인 데이터를 대상으로 replay data가 가능하다.
- Data that shares the same partition goes to the same shard (ordering) : Partition key를 이용해 순서보장이 필요한 데이터는 순서보장 지원
- Once data is inserted in Kinesis, it can’t be deleted (immutability) : 데이터가 한 번 쌓이면 retention기간 동안 지울 수 없다.
- Real-time : ~200ms의 실시간 처리가 가능하다.
- **Destination**
  - 1. SDK를 사용한 애플리케이션(real time처리가 가능하기 때문)
  - 2. Lambda
  - 3. Kinesis Data Firehose
  - 4. Kinesis Data Analytics
- **Shard를 사용자가 직접 확장하거나 축소해야 하기 때문에 fully managed service는 아니다**.

#76 Kinesis Data Firehose
- Fully Managed Service, no administration, automatic scaling, serverless : Kinesis Data Streams와 달리 fully managed service이다.
- 배치 처리를 하기 때문에 처리에 시간이 걸린다. 따라서 Near Real Time이라고 부른다.
  - 60 seconds latency minimum for non full batches
  - Or minimum 32 MB of data at a time
- Kinesis Data Streams와 달리 데이터를 가공한 후 batch 처리해서 consumers에게 전달한다.(이 가공 과정과, batch처리 기간이 있기 때문에 real time으로 사용이 불가능하다.)
- **Destination**
  - 1. datadog같은 모니터링 3rd-party destination 
  - 2. S3, Redshift(페타바이트 이상의 대규모 데이터 저장소 및 분석 서비스), ElasticSearch와 같은 aws destination
  - 3. http endpoint
- 데이터를 저장하지 않으므로 Kinesis Data Streams와 달리 data replay가 불가능하다.

#77 Kinesis Data Analytics
- Kinesis Data Streams또는 Kinesis Data Firehose로부터 전달받은 데이터를 바탕으로 SQL을 실행할 수 있는 서비스
- Fully Managed Service이다.
- Real-time이기 때문에 실시간 데이터 분석이 가능하다.
- Kinesis Data Streams또는 Kinesis Data Firehose를 source로 하고 이 데이터를 대상으로 SQL을 실행하고 다시 Kinesis Data Streams또는 Kinesis Data Firehose에 전달하는 구조이다.
- 크게 두 가지 구조를 사용한다.
  - SQL실행 후 다시 Kinesis Data Streams에 전달해 Kinesis Data Streams가 APP이나 Lambda로 전달하는 방식
  - SQL실행 후 다시 Kinesis Data Firehose에 전달해 Kinesis Data Firehose가 S3나 Redshift같은 저장소로 전달해 저장하는 방식
  
#78 Ordering data into Kinesis
- Partition key를 kinesis에 보내면 hash해서 알맞은 shard로 배치한다.
- The same key will always go to the same shard : 예를 들어 1번 partition key가 1번 샤드에 배치되고, 2번 partition key가 2번 샤드에 배치되었다면, 그 이후에도 1번 partition key는 1번 샤드에 배치되고, 2번 partition key는 2번 샤드에 배치되는 시스템이다.

#79 Ordering data into SQS
- kinesis와 정렬 매커니즘이 다르다.
- For SQS FIFO, if you don’t use a Group ID, messages are consumed in the order they are sent, with only one consumer : sqs는 consumer가 메시지를 consume한 후 그 메시지를 지우는 것이 일반적이고, 한 consumer에 의해서 이 작업이 이루어진다. group id가 없다면 메시지는 보낸 순서대로 읽혀질 것이다.
- 413p 참고

#80 SQS VS SNS VS KINESIS
- SQS
  - Data is deleted after being consumed
  - No need to provision throughput
- SNS
  - Data is not persisted (lost if not delivered) : 데이터는 어떤 이유로 전달되지 못했을 경우 저장되지 않기 때문에 데이터가 사라질 수 있다.
  - No need to provision throughput
  - Integrates with SQS for fan- out architecture pattern : 한 메시지를 여러 consumer가 소비하는 것이 가능함.
- KINESIS
  - **Throughput**
    - Standard: pull data - 2 MB per shard
    - Enhanced-fan out: push data - 2 MB per shard per consumer
  - Possibility to replay data : Kinesis Data Streams의 경우 데이터가 저장되므로 저장된 데이터를 대상으로 작업 가능

#81 Amazon MQ
- Amazon MQ = managed Apache ActiveMQ
- on-premise mq가 MQTT, AMQP같은 기존에 존재하던 프로토콜을 사용하고 있는데 클라우드로 이전하고 싶다면 AMAZON MQ를 사용하면 된다.(SNS, SQS로의 이전이 아니라는 점 주의)
- High Availability를 지원한다.

#82 Docker
- Docker is a software development platform to deploy apps : 소프트웨어 배포를 위한 플랫폼, 애플리케이션을 컨테이너 안에 담을 수 있게 해주는 플랫폼

#83 Docker vs Virtual Machines
- Docker is ”sort of” a virtualization technology, but not exactly : 도커는 일종의 가상화 기술이지만, 가상화 그 자체는 아니다.
- 421p. Resources are shared with the host => many containers on one server : 한 서버에 여러 컨테이너가 올라가 있다면, 저장소나 네트워크를 도커 컨테이너 간에, 혹은 호스트와 공유할 수 있다.
- 반면 개별 Virtual Machine(Guest OS)은 호스트 OS와 자원을 공유하지 않는다.

#84 ECS
- 2개의 launch type
  - 1. ec2 launch type : 직접 ec2를 확장하거나 축소시킨다.
  - 2. fargate : ec2 관여 필요 없이 serverless로 동작한다.
- **EC2 launch type** : ECS Cluster 내부에 여러 개의 Container Instance가 있고, 개별 Container Instance 내부에는 ECS Agent가 있고, 이 ECS Agent가 Container Instance 내부의 task들을 외부에 노출시키는 역할을 한다. a task는 한 개의 실행 중인 도커 컨테이너라고 생각하면 편하다.
- **Fargate** : ECS Cluster 내부에서, 한 개의 task에 한 개의 ENI가 붙는다. 따라서 많은 task가 존재한다면 그 만큼의 ip를 감당해야 하므로 네트워크가 충분히 커야 한다.
- **EC2 Instance Profile** : ECS agent가 가지는 권한이다. 427p 참고
- **ECS Task Role** : 개별 태스크가 가지는 권한(role)이다. 이 role에 해당하는 권한만 가지고 다른 aws 서비스에 접근한다.
- ECS에서 CloudWatch에 의한 Scaling작업을 할 때, 두 단계의 Scaling이 이루어진다.
  - 1. task의 개수를 늘려 서비스 컨테이너의 수를 늘리는 작업 : 컨테이너가 인스턴스의 capacity를 초과해서 생성될 수는 없다.
  - 2. Scale ECS Capacity Providers : 따라서 인스턴스를 수평적으로 늘리는 작업을 ECS Capacity Providers가 도와준다.
  - 이 작업이 가능하려면 fargate를 사용하지 않는 ec2 launch type이어야 하고, 당연하게도 auto scaling group을 사용해야 한다.
- **ECS Rolling Update** 
  - 컨테이너 업데이트를 실행하기 위해 사용할 수 있는 하나의 방법으로, 태스크를 업데이트하는 방법이라고 생각하면 편하다.
  - 컨테이너 업데이트를 위해서는 컨테이너가 remove되고, 새로운 컨테이너로 실행되어야 한다.
  - 초기 태스크 수 기준으로, 퍼센트 단위로 태스크의 실행 개수를 정할 수 있다.

#85 EKS
- ECS와 마찬가지로 아래의 launch type을 지원한다.
  - 1. ec2 launch type : 직접 ec2를 확장하거나 축소시킨다.
  - 2. fargate : ec2 관여 필요 없이 serverless로 동작한다.
- 자세한 구조 441p 참고
- ECS에서는 **task**라고 부르던 컨테이너가, EKS에서는 **pod**라고 불리고, 한 개의 EC2 Instance를 **EKS node**라고 부른다.

#86 Lambda
- Virtual functions – no servers to manage! : 요청 횟수 또는 GB-seconds로 요금이 계산되기 때문에 요청 횟수를 예상할 수 없거나, 요청 횟수가 적을 때 효과적이다.

#87 Lambda@Edge
- Lambda를 CloudFront처럼 쓸 수 있게 해주는 서비스

#88 DynamoDB
- Fully managed, highly available with replication across multiple AZs : 서버 리스 NoSQL 데이터베이스 with multi AZ
- **Provisioned Mode(default)**
  - Pay for provisioned Read Capacity Units (RCU) & Write Capacity Units (WCU) : 미리 capacity를 정해서 예상된 트래픽에 대응한다.
- **On-Demand Mode**
  - Read/writes automatically scale up/down with your workloads : 트래픽을 예상할 수 없을 때 자동으로 scale해주는 모드, 당연히 default보다 더 비싸다.

#89 DynamoDB Accelerator (DAX)
- Fully-managed, highly available, seamless in memory cache for DynamoDB : DynamoDB전용 캐시
- **DynamoDB Accelerator(DAX) vs ElastiCache**
  - DynamoDB에 **DynamoDB Accelerator(DAX)**를 사용하면 DynamoDB의 개별 object, 쿼리 결과를 캐시할 수 있다.
  - **ElastiCache**도 같이 사용 가능한데, ElastiCache는 Application이 DynamoDB 또는 DynamoDB Accelerator(DAX)으로부터 데이터를 전달받고 이 데이터를 가공한 후 이 데이터를 캐시하고 싶을 때 사용한다. 이를 Store Aggregation Result라고 부른다.
  - 위 케이스에서 둘의 차이점은 데이터 가공여부라고 볼 수 있다.

#90 DynamoDB Streams
- Ordered stream of item-level modifications (create/update/delete) in a table
- DynamoDB의 object를 create/update/delete할 때 이벤트가 발생해 DynamoDB Streams가 발생하게 할 수 있다.
- 구조 464p 참고

#91 DynamoDB Features
- **DynamoDB Global Tables**
  - active-active replication의 multi region방식을 지원한다.(region은 2개 이상 가능)
  - Applications can READ and WRITE to the table in any region : 아무데나 READ, WRITE해도 active-active replication이기 때문에 상관 없고, multi region이기 때문에 low latency라는 장점이 있다.
  - Must enable DynamoDB Streams as a pre-requisite : DynamoDB Streams을 필수적으로 활성화해야 함
- **Time To Live**
  - expire time을 지정해서 1.item이 너무 많아지거나 2.세션 관리를 해야할 경우, item을 삭제하는 용도로 사용할 수 있다.
- **Indexes**
  - DynamoDB에서 쿼리는 Primary key를 대상으로만 가능한데 인덱스를 설정하면 attributes를 대상으로 쿼리가 가능하다.
- **Transactions**
  - A Transaction is written to both tables, or none! : rds로 예를 들면 외래키를 설정했을 때 자식 테이블과 부모 테이블의 데이터를 동기화하기 위해서 부모 테이블의 외래키 update가 cascade되는 경우가 있다. 이를 DynamoDB에서는 transaction이라는 개념으로 동시에 쓰거나 동시에 쓰지 않는 방식으로 구현했다고 보면 된다.(데이터를 참조하는 경우 동일 데이터 보장)

#92 AWS API Gateway
- **Edge-Optimized(default) mode: For global clients**
  - Requests are routed through the CloudFront Edge locations (improves latency) : 요청을 받을 때 CloudFront를 사용해 latency를 줄이고, 실제로는 한 region에만 api gateway를 두는 방식이다.
- **Regional mode**
  - Edge-Optimized(default)와 비슷하게 CloudFront를 사용하는 점까진 같은데, 지역 한정으로 동작하므로 캐시를 다룰 때 더욱 세부적으로 다룰 수 있다고 한다.
- **Private mode** 
  - Can only be accessed from your VPC : private VPC에서 ENI를 사용해서 접근한다.

#93 API Gateway – Security
- **IAM Permissions**
  - Sig v4라는 일종의 서명키를 요청 헤더 또는 쿼리 문자열에 추가한다.
  - 이 서명키를 API Gateway에 보내면 API Gateway가 IAM service에게 IAM policy check을 요청한다. 
  - 인증과 인가 둘 다 가능
- **Lambda Authorizer (formerly Custom Authorizers)**
  - Token을 요청 헤더에 담아 API Gateway에 보내면 API Gateway가 Lambda Authorizer에게 token check을 요청한다.
  - 토큰이 유효하다면 IAM Permission과 동일하게 IAM policy를 리턴한다.
  - IAM Permissions의 Sig v4라는 AWS전용 서비스가 아니라 OAuth같은 third party 인증타입을 지원하는 것이 주요 기능이다.
  - 인증과 인가 둘 다 가능
- **Cognito User Pools**
  - Cognito fully manages user lifecycle : serverless 유저 인증 데이터베이스(모바일 특화)
  - 1. Cognito User Pools에 인증 요청을 하고 토큰을 받는다.
  - 2. API Gateway에 토큰과 요청을 함께 보낸다.
  - 3. API Gateway는 Cognito User Pools에 이 토큰이 있는 지 확인한다.
  - IAM Permissions, Lambda Authorizer와 다른 점은 바로 API Gateway에 요청하는 것이 아니라, 그 이전에 Cognito User Pools으로부터 발급받은 토큰이 필요하기 때문에 Cognito User Pools에 먼저 인증요청을 보내야 한다.
  - 인증만 가능

#94 AWS Cognito
- **Cognito User Pools**
  - Sign in functionality for app users : 모바일 앱을 위한 인증 제공
  - Sends back a JSON Web Tokens (JWT) : (모바일 App을 위해서) 로그인 시 jwt를 제공해준다.(앱은 쿠키 세션 관리가 힘들다고 알고 있음 - 학습 필요)
  - Integrate with API Gateway와 함께 동작
- **Cognito Identity Pools (Federated Identity)**
  - AWS resources에 직접 접근하기 위한 AWS credentials을 제공한다. 
  - **identity provider로 Cognito User Pools를 사용**한다.
  - 1. 클라이언트가 identity provider에 로그인 요청을 보낸다.
  - 2. identity provider는 응답으로 토큰을 반환한다.
  - 3. 토큰을 Federated Identity로 보낸다.
  - 4. Federated Identity는 identity provider에게 토큰을 검증한다.
  - 5. 토큰이 유효하다면 STS에서 credential를 획득해 temp credential을 client에게 리턴한다.
  - 6. client는 temp credential을 이용해서 aws service에 직접 접근이 가능하다.
  - 사용 사례 : 페이스북 로그인으로 s3에 직접 쓰기 권한을 얻고 싶을 때
- **Cognito Sync == Appsync**
  - Requires Federated Identity Pool in Cognito (not User Pool) : **Federated Identity Pool이 요구조건**이다.
  - Store preferences, configuration, state of app : app의 상태, 설정을 저장한다.
  - Cross device synchronization : 멀티 디바이스 간 싱크 가능
  - Offline capability : 오프라인 상태여도 온라인되면 싱크 가능
  - Store data in datasets : dataset에 data를 저장하고 dataset은 최대 20개까지 가능

#95 AWS SAM - Serverless Application Model
- Framework for developing and deploying serverless application : 이런 프레임워크도 있다는 것만 알아두기

#96 AWS Database Types
- RDBMS : RDS, Aurora – 자유로운 join
- NoSQL : DynamoDB(JSON), ElasticCache(Key, Value), Neptune(graphs) - 불완전한 쿼리, 조인
- Object Store : S3, Glacier - 파일 저장
- Data Warehouse : Redshift(OLAP), Athena - 데이터 분석을 위한 데이터 웨어하우스
- Search : ElasticSearch (JSON)
- Graph : Neptune – displays relationships between data : 데이터 간 관계를 보여주는 그래프 형식

#97 RDS Features
- Managed PostgreSQL / MySQL / MariaDB / Oracle / SQL Server
- (참고로 현재 RDS 생성 페이지에서 Amazon Aurora가 managed db중에서 제일 먼저 선택되는 기본옵션이다. 이것만 봐도 아마존이 aurora를 장려하고 있다는 것을 알 수 있다. (2022-01-19))
- DB instance & EBS Volume type and size를 사전에 provision해야 한다.
- **Read Replicas & Multi AZ** 지원
- OLTP - Online Transaction Processing(트랜잭션 처리 가능)
- 동작 방식
  - **small downtime when failover happens** : failover가 발생할 때 약간의 다운 타임이 있다.(오로라는 30초 이내)
  - 즉 maintenance happens, scaling in read replicas / ec2 instance / restore EBS이 발생할 때 다운 타임이 존재한다는 것이다.
  - Performance: **depends on EC2 instance type, EBS volume type**, ability to add Read Replicas. Storage auto-scaling & manual scaling of instance : 인스턴스 타입, 볼륨 타입에 따라 성능이 다르고, read replica를 추가해 읽기 성능을 향상시킬 수 있다는 점, 저장소 오토-스케일링 & manual 스케일링이 가능하다는 점으로 인해 성능은 천차만별이다.
- Backups are automatically enabled in RDS : 자동 백업, 백업본 default 유지 기간 7일(최대 35일), 트랜잭션 로그가 5분마다 백업됨
- DB Snapshot : 유저가 직접 스냅샷 캡처하는 것
- Maximum Storage Threshold : rds의 storage는 auto-scaling되지만 auto-scaling의 최대치를 필수적으로 정해주어야 한다.
- Read Replica는 최대 5개(multi AZ or multi Region)
- Replication은 **비동기** 작업이다. 즉 마스터 인스턴스에만 쓰고 읽기만 가능한 read replica에 비동기적으로 복제하는 방법을 택한다.
- **For RDS Read Replicas within the same region, you don’t pay that fee** : same region은 그 대가로 돈을 지불하지 않는다, **cross region read replication역시 가능하지만 cost가 든다.**
- RDS Multi AZ는 **동기** 작업이다. - **Zero downtime operation**, **no cost**, 가용성 증가(Increase availability), scaling이 아닌 Disaster Recovery을 위해 사용
- 173p Multi-AZ 동작 과정 필히 참고
- **Security**
  - **No SSH access** : ssh 접속 안 된다. 필히 기억.
  - If the master is not encrypted, the read replicas cannot be encrypted : 마스터 인스턴스가 encrypt되지 않으면 replica도 encrypt되지 않는다.
  - Possibility to encrypt the master & read replicas with AWS KMS - **AES-256 encryption** : 저장된 데이터 암호화를 at rest encryption이라고 부른다.
  - **In-flight encryption** : SSL통신을 하기 위해서는 MySQL같은 DB에 GRANT USAGE ON *.* TO 'mysqluser'@'%' **REQUIRE SSL;** 같이 설정을 해줘야 한다.
  - **Network Security** : security group can communicate with RDS (ec2의 보안 그룹과 같은 개념)
  - RDS에 IAM Authentication을 적용시키면 ec2에서 iam role이 iam으로부터 토큰을 받아와 rds에 ssl로 접속한다.

#98 Amazon Aurora Features
- Define EC2 instance type for aurora instances
- **Data is held in 6 replicas, across 3 AZ**
- 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS : 성능이 좋다.
- Aurora storage automatically grows in increments of 10GB, up to 128 TB : rds와 마찬가지로 오토 스케일링을 지원한다.
- Aurora can have 15 replicas while MySQL has 5, and the replication process is faster
- Failover in Aurora is instantaneous. It’s HA (High Availability) native.
- **Support for Cross Region Replication**
- **Shared Storage Volume**
  - 한 개의 마스터만 데이터를 써도 모든 인스턴스가 볼륨을 공유하므로 데이터를 읽어낼 수 있다.
  - **Shared Storage Volume**은 **Replication** + **Self Healing** + **Auto Expanding**의 특성을 지닌다.
- **Aurora DB Cluster** : 181p 참고
- Aurora Multi-Master : 멀티 마스터 즉각적인 failover을 가능하게 해준다. 30초도 길다고 생각한다면 멀티 마스터 사용가능.
- **Aurora Global Database (recommended)**
  - disaster recovery가 1분 안에 가능
  - **decreasing latency**
  - 1개의 마스터 인스턴스, 최대 16개의 read-only인스턴스 in 최대 5개의 region, region간의 replication lag은 1초 미만이다.
- Aurora Serverless – for unpredictable / intermittent workloads

#99 ElastiCache Features
- **Managed** Redis / Memcached (similar offering as RDS, but for caches)
- Must provision an EC2 instance type
- Key/Value store, Frequent reads : 세션 관리 최적화
- **Clustering(Redis)**, Multi AZ, Read Replicas
- **Redis**
  - Multi AZ with auto-failover, Read Replicas, **Backup and restore features**
  - **Read Replicas to scale reads and have high availability**
  - AOF(Append Only File) : redis는 in-memory db이므로 인스턴스 종료 시 데이터 유실을 방지하기 위해서 명령문이 실행될 때마다 1초 정도 단위로 파일에 저장한다.
- **MEMCACHED**
  - Multi-node for partitioning of data(sharding) : 샤딩이란 테이블의 수평 분할을 의미한다. 데이터가 많아질 경우를 대비하여 한 테이블의 데이터를 여러 데이터베이스에 나누어 저장함을 말한다. 한 레코드를 여러 디비에 저장하는 read replica가 아니라, 두 레코드를 두 개의 db에 나누어 저장하는 것이 샤딩이다.
  - **No high availability(replication), No backup and restore** : 데이터가 유실되면 그대로 유실된다.
  - Multi-threaded architecture
  - Supports SASL(Simple Authentication and Security Layer)-based authentication : 애플리케이션 프로토콜들로 부터 인증 메커니즘을 분리한 것이 SASL이다. 즉 https같은 프로토콜을 사용하지 않고, 다른 보안 레이어를 사용한 것이다.
- **Cache Security**
  - **Do not support IAM authentication** : **RDS, AURORA와 다르게 ElastiCache**는 IAM Authentication을 지원하지 않는다. IAM Authentication을 사용하면 권한을 가진 aws유저가 redis에 어떤 값이든지 넣을 수 있기 때문에 그런 것이 아닌가 추측해본다. redis는 세션을 관리하는 데에 주로 쓰이기 때문에 보안이 중요할 수 밖에 없다.
  - **IAM policies on ElastiCache are only used for AWS API-level security** : API-level security 정도만 iam policy로 관리할 수 있다.
  - **Redis AUTH**
    - IAM policies on ElastiCache are only used for AWS API-level security : IAM Policy는 그냥 elasticache api를 호출가능한 iam인지만 확인하고, elasticache에 접근해서 데이터를 조작할 수 있는 권한을 부여하지는 않는다는 뜻읻다.
    - You can set a **password/token** when you create a Redis cluster
    - This is an extra level of security for your cache 
    - Support SSL in flight encryption
- Redis Sorted sets guarantee both **uniqueness** and **element ordering**

#100 DynamoDB Features
- AWS proprietary(전용, 독점 정도의 뜻) technology, managed NoSQL database
- Serverless, **provisioned capacity**, auto scaling, on demand capacity
- Can replace **ElastiCache as a key/value store** (storing session data for example)
- **Multi AZ by default**
- **Backup / Restore feature**
- Read and Writes are decoupled : 읽기와 쓰기 분리
- **DAX for read cache** : elasticache의 대체재가 될 수 있는 이유 중 하나인 것 같다.
- Security, authentication and authorization is done **through IAM**
- **DynamoDB Streams** : 보통 kinesis 또는 lambda로 데이터를 보낼 수 있다.
- **Global Table feature** : 읽기와 쓰기를 어떤 region에도 할 수 있다는 것이 global table이다. 이는 active-active replication이 있기 때문에 가능하다. 여러 지역 간에 양방향의 replication이 가능하다는 것이다.
- **Can only query on primary key, sort key, or indexes**

#101 Athena Features
- Fully **Serverless database** with SQL capabilities
- Used to query data in S3, Pay per query, Output results back to S3
- Secured through **IAM**

#102 Redshift
- Redshift is based on PostgreSQL, but it’s not used for OLTP(Online transaction processing)
- It’s OLAP – online analytical processing (analytics and **data warehousing**) : 즉 쓰기를 위한 db가아니라 분석을 위한 db이다.
- **Columnar storage** of data (instead of row based)
- **Massively Parallel Query Execution (MPP)** : 분석을 위한 쿼리를 많이 하므로 병렬 쿼리를 지원한다.
- BI(Business Intelligence : 데이터를 분석하여 효과적인 의사결정 도모) tools such as AWS Quicksight or Tableau integrate with it
- Data is loaded from S3, DynamoDB, DMS, other DBs…
- From 1 node to 128 nodes, up to 128 TB of space per node
- **Redshift Cluster**
  - **Leader node**: for query planning, results aggregation
  - **Compute node**: for performing the queries, send results to leader
- **Redshift Spectrum**: perform queries directly against S3 (no need to load), S3를 대상으로 로딩없이 쿼리 가능
  - Redshift Cluster가 필수적으로 enable되어야 한다.
- **Backup & Restore을 지원하지만 Multi AZ를 지원하지 않는다.**
- 따라서 disaster recovery를 snapshot를 캡처해서 직접 설정해줘야 한다. 스냅샷을 캡처해 멀티 리전에 백업하는 과정은 자동화(aws 공식 지원)될 수 있다. 531p 참고 
- disaster recovery를 어찌 되었든 자동화가능하기 때문에 **auto healing features, cross-region snapshot copy**의 특성을 가진다.
- vs Athena : 아테나도 역시 s3를 대상으로 쿼리하는데 아테나에 비해 faster queries/joins/aggregations thanks to **indexes**의 장점을 가진다.(Redshift는 인덱스를 가지는 특성이 있다.)

#103 Glue, Neptune
- 535p 참고

#104 ElasticSearch
- Example: In DynamoDB, you can only find by primary key or indexes
- With ElasticSearch, **you can search any field, even partially matches** : ElasticSearch는 기본적으로 NoSQL database라고 한다. 검색 엔진이기도 한데 NoSQL database라.. 더 조사가 필요할 것 같다.
- 따라서 DynamoDB같은 데이터베이스에 데이터를 store하고 ElasticSearch로 데이터를 서치하는 작업을 할 수 있다고 한다.
- **Cognito를 지원한다.**

#105 CloudWatch Metrics
- Metric이란? : **Metric is a variable to monitor** (CPUUtilization, NetworkIn…)
- Metrics belong to namespaces : 한 메트릭은 한 네임스페이스에 속한다.
- Metrics have timestamps : metric은 시간 속성을 필수적으로 가진다. 이는 모니터링하는 metric간에 같은 timestamp를 공유해야 하기 때문에 중요하다. dashboard로 여러 metric들을 관찰할 때 같은 시간대인 것이 보장되어야 편하기 때문이다.
- **Dimension** is an attribute of a metric (instance id, environment, etc…) : Up to 10 dimensions per metric
- **EC2 Detailed monitoring** : 5분마다 metric 갱신이 default, Detailed monitoring을 사용하면 1분마다 갱신 가능
  - **EC2 Memory usage is by default not pushed** : 메모리 사용량은 default지표가 아니고, unified agent를 ec2 instance내부에 설치 후 메모리 사용량을 custom metric으로 사용해야 한다.
- **Custom Metrics**
  - Use API call **PutMetricData**
  - **StorageResolution** : 1분마다 metric 갱신이 default, High Resolution을 사용하면 1/5/10/30초마다 갱신 가능
  - Accepts metric data points **two weeks in the past and two hours in the future** (make sure to configure your EC2 instance time correctly) : Cloudwatch로 보내지는 metric은 과거 2주, 미래 2시간 사이의 metric만 받는다. 따라서 metric을 보낼 때 timestamp를 잘 설정해야 한다.
- CloudWatch Dashboards
  - Great way to setup custom dashboards for **quick access to key metrics** and alarms : 계속 관찰해야 할 key metric에 빠르게 접근 가능
  - Dashboards are global, Dashboards can include graphs from different AWS accounts and regions : 한 대시보드에 여러 region의 metric을 넣을 수 있다. 예를 들어 한국 region metric과 미국 region metric이 한 대시보드에 공존하는 것이다.
  - Dashboards can be shared with people who don’t have an AWS account : aws계정이 없어도 다른 사람들과 공유가능, aws계정이 없는 다른 팀과의 협업에 좋을 듯?
  - You can setup automatic refresh : 대시보드를 시간마다 자동 갱신가능

#106 CloudWatch Logs
- **Log groups**: arbitrary name, **usually representing an application** : 예를 들어 lambda에서 로그가 쌓인 것이라면 lambda가 prefix나 postfix로 붙은 naming을 사용한다. 물론 커스텀 로그는 사용자가 직접 이름을 설정한다.
- **Log stream**: instances within application / **log files** / containers - 로그 파일의 용량이 어느 정도 커지면 로그 파일을 보낼수도 있고, 로그가 생길 때마다 보낼 수도 있는 것 같다.
  - Log group안에 Log stream이 로우처럼 쌓이는 방식이다.
- Can define log expiration policies (never expire, 30 days, etc..) : never expire로 설정할 시 일종의 데이터베이스로 사용가능하다.
- CloudWatch Log에 metric filter를 사용하면, custom metric을 만들어낼 수 있다.
- **CloudWatch Insights**
  - CloudWatch Logs Insights can be used to query logs and add queries to CloudWatch Dashboards : CloudWatch Logs를 대상으로 쿼리할 수 있고, 쿼리 결과를 dashboard에 쌓을 수도 있는 것 같다.
- **S3 Export**
  - API NAME : **CreateExportTask**을 사용해서 s3로 로그를 export할 수 있다.
  - 하지만 최대 12시간이 걸리기 때문에 Not near-real time or real-time이기 때문에 실시간으로 전송하려면 Logs Subscriptions을 사용해야 한다.
- **CloudWatch Logs Subscriptions**
  - CloudWatch Subscription Filter을 사용하면 Lambda혹은 Kinesis에 데이터를 보내 real-time에 가깝게 사용할 수 있다. 549p 참고
- **CloudWatch Logs Agent & Unified Agent**
  - By default, no logs from your EC2 machine will go to CloudWatch : 따라서 agent를 ec2혹은 on-premise server에 설치해 원하는 로그를 보낼 수 있다.
  - Unified Agent는 최신 버전의 Logs Agent이고 더 많은 기능을 지원한다.
  - **Unified Agent**
    - Collect additional system-level metrics : **CPU, Disk metrics, RAM, Netstat**, Processes, Swap Space

#107 CloudWatch Alarms
- **Period**
  - metirc을 몇 초마다 평가할 것인가를 정한다.
  - High resolution custom metrics: 10, 30, 60초 단위로 설정 가능
- **Alarm States**
  - OK : 문제 없는 상태
  - ALARM : 문제 있는 상태
  - INSUFFICIENT_DATA : metric을 보고 판단하기에 데이터가 불충분한 경우
- **Alarm Targets**
  - Stop, Terminate, Reboot, or Recover an EC2 Instance
  - Trigger Auto Scaling Action
  - Send notification to SNS

#108 CloudWatch Events
- Event Pattern: Intercept events from AWS services : aws service에서 발생하는 이벤트를 가져올 수 있다.
  - Example sources: EC2 Instance Start, CodeBuild Failure
  - Can intercept any API call with **CloudTrail integration**
- Schedule or Cron 
  - 예시로 특정 이벤트를 매 시간마다 발생하게 만들 수 있다.
- A JSON payload is created from the event and passed to a target : 기본적으로 json 형식으로 이벤트가 대상에게 전달된다.

#109 Amazon EventBridge
- EventBridge is the next evolution of CloudWatch Events : EventBridge는 CloudWatch Events의 최신 버전이다.
- Amazon EventBridge builds upon and **extends** CloudWatch Events
- Over time, the CloudWatch Events name will be replaced with EventBridge
- Default event bus: generated by AWS services (CloudWatch Events) > EC2 Instance Start, CodeBuild Failure같은 aws specific한 이벤트가 발생할 경우 사용
- **EventBridge specific한 기능**
  - Partner event bus: receive events from SaaS service or applications (Zendesk, DataDog, Segment, Auth0…)
  - 예를 들어, 데이터독에서 모니터링하다 문제 있을 경우 이벤트를 발생시켜 EventBridge로 전달시킬 수 있다는 것이다.
  - Custom Event buses: for your own applications
  - **The Schema Registry**
    - The Schema Registry allows you to generate code **for your application**, that will know in advance how data is structured in the event bus
    - 스키마 레지스트리는 event가 발생할 경우 전달될 json을 어떤 구조로 설계할 것인지를 미리 정해놓기 위해 사용하는 서비스이다.
    - EventBridge can **analyze the events in your bus** and infer the schema
- **Event buses can be accessed by other AWS accounts** : 다른 계정에 의해 event bus가 access가능한 것은 Partner event bus 때문에 권한을 열어줘야해서 그런 것이 아닌가 추측해본다.

#110 AWS CloudTrail
- CloudTrail is enabled by default!
- 참고
  - IAM Credential Report : 한 계정 내부의 IAM user들의 자격 증명의 상태를 관리
  - IAM Access Advisor : last-access를 파악해 권한 관리 가능
- Provides governance(관리), compliance(규정 준수) and audit(감사) for your AWS Account
- Get an history of events / API calls made within your AWS Account by:
  - Console
  - SDK
  - CLI
  - AWS Services
- **Events are stored for 90 days in CloudTrail**
  - 장기 보관을 할 것이라면 s3에 보내면 된다.
- **Can put logs from CloudTrail into CloudWatch Logs or S3**
- **A trail can be applied to All Regions (default) or a single Region.** : 모든 Region의 access를 파악할 수 있다.
- **If a resource is deleted in AWS, investigate CloudTrail first!** : 자원을 누가 삭제했는지 보고 싶다면 cloudtrail을 사용하면 된다.
- **CloudTrail Events**
  - **Management Events**
    - Operations that are performed on resources in your AWS account : 예를 들어 ec2에 iam role을 설정해주는 것. 웬만한 aws service는 api call가 호환되므로, api call이 가능한 모든 서비스는 cloudtrail로 추적이 가능하다고 보면 된다.
    - Read Eventsd와 Write Events를 분리할 수 있다.
  - **Data Events**
    - **By default, data events are not logged (because high volume operations)** : 데이터 이벤트는 기본적으로 쌓이지 않는다.
    - 이 옵션을 활성화하면 getObject, putObject 등의 s3 event가 발생할 때마다 로그를 쌓을 수 있다.
  - **CloudTrail Insights Events**
    - CloudTrail Insights : CloudTrail Insights to detect unusual activity **in your account** > 계정 레벨에서 계정 내부 유저의 비정상적인 활동을 감지할 수 있다. 예시 565p 참고
    - Continuously analyzes write events to detect unusual patterns : CloudTrail은 지속적으로 Management Events를 분석하면 이상이 없는지 파악하고 이상 감지 후 이벤트를 CloudTrail Console 또는 s3 또는 eventbridge에 보낸다.

#111 AWS Config
- Helps with **auditing and recording compliance(규정 준수)** of your AWS resources : 규정 준수라고 함은 aws를 사용하면서 자신이 생각한대로 각종 서비스들을 잘 사용하고 있나 정도로 생각하면 된다.
- Helps record configurations and changes over time : 아래에서 compliance를 audit하는 예시를 들 수 있다.
  - Is there unrestricted SSH access to my security groups?
  - Do my buckets have any public access?
  - How has my ALB **configuration changed over time**?
- You can receive alerts (SNS notifications) for any changes
- AWS Config is a **per-region service**
- Can be aggregated **across regions and accounts** : CloudTrail, CloudWatch Dashboard와 다르게 per region 서비스이지만 cross region하게 만들 수 있다.
- Possibility of storing the configuration data into S3 (analyzed by Athena) : CloudWatch, CloudTrail과 동일

#112 AWS Config Rules
- **Managed config rules** (over 75)
- **Make custom config rules** (must be defined in AWS Lambda)
  - Example: evaluate if each EBS disk is of type gp2
  - Example: evaluate if each EC2 instance is t2.micro
- Rules can be evaluated / triggered
  - For each config change 
  - At regular time intervals : triggered
- **AWS Config Rules does not prevent actions from happening(no deny)** : 예시로 ec2타입을 t2로 aws config로 설정하고 r계열의 인스턴스를 생성할 수도 있다는 뜻.
- **AWS Config Resource**
  - View compliance of a resource over time
  - View configuration of a resource over time
  - **View CloudTrail API calls of a resource over time** : compliance, configuration는 compliance(규정 준수) check을 위해서 당연히 볼 수 있는 것인데 Cloudtrail API calls까지 볼 수 있는 것도 생각해야 한다.
- **Config Rules Remediations** : Non-compliant resource에 대한 회복
  - Non-compliant한 resource에 대해 자동으로 **Auto-Remediation Action**(SSM Document: AWSConfigRemediationRevokeUnusedIAMUserCredentials)을 실행할 수 있다.
  - SSM Automation을 사용하지 않고, Custom Automation을 사용해서 lambda function을 사용할 수도 있다.
  - Remediation실행 후에도 회복이 안되면 5번을 dafault로 시도
- **Config Rules Notifications**
  - aws config는 eventbridge에 event를 전달할 수 있다.
  - aws config에서 발생하는 모든 이벤트를 eventbridge에 전달하고 eventbridge는 sns에 전달할 수 있는데 이 과정에서 특정 이벤트만 원한다면 sns filter를 사용하면 된다.
  - CloudWatch vs CloudTrail vs Config 571p 필히 참고.

#113 AWS STS – Security Token Service
- Allows to grant limited and temporary access to AWS resources : aws 자원에 대한 일시적 접근을 원할 때 사용
- **Token is valid for up to one hour (must be refreshed)** : 최대 1시간까지 유효한 토큰을 사용
- AssumeRole : 아래 두 가지 경우에 사용된다.
  - 1. Within your own account: 강화된 보안을 위해서
  - 2. Cross Account Access: 예를 들어, development계정의 유저가 배포를 위해서 production계정의 역할이 필요한 경우 production계정을 주는 게 아니라 배포에 필요한 role만 잠깐 sts로 줄 수 있다.
  - https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html 참고
- Assume Role 사용법
  - 1. Define an IAM Role within your account or cross-account
  - 2. Define which principals(**a person or application that can make a request** for an action or operation on an AWS resource) can access this IAM Role : 누가 접근할 수 있는지 설정
  - 3. Use AWS STS (Security Token Service) to retrieve credentials and impersonate(모방) the IAM Role you have access to (AssumeRole API) : 576p 참고
  - 간단하게 생각하면 된다. 예를 들어 development계정의 유저가 production s3 bucket에 접근해야할 일이 생긴다면 미리 production s3 bucket에 접근할 수 있는 IAM Role을 만들어놓고, sts token을 development계정에 부여해서 production s3 bucket에 일시적으로 접근하는 것이다.

#114 Identity Federation in AWS
- Federation lets users outside of AWS to assume temporary role for accessing AWS resources. : 외부 서비스 로그인으로 aws 자원에 직접 접근할 때 사용하는데에 최적화되어 있다.
- Using federation, you don’t need to create IAM users (user management is outside of AWS) : 예를 들어 백만명의 사용자가 aws자원에 접근해야 할 때 일일이 계정을 생성해주기는 어렵다.
- Identity Federation은 외부에서 어떤 인증 방법(SAML 2.0, Web Identity Federation, Custom Identity Broker)을 사용하든지 매커니즘은 비슷하다. **외부 서비스에 로그인하고, aws에 액세스 요청 후, sts로 토큰을 받아 aws자원에 직접 접근하는 방식이다.**
- SAML2.0(Security Assertion Markup Language, 다른 애플리케이션에 인증 토큰을 전달할 수 있게 해주는 로그인 표준)은 deprecated이고, Amazon Single Sign On (SSO)을 권장한다.
- **Custom Identity Broker Application** : 이것은 SAML 2.0을 지원하지 않는 곳에 사용하는데, identity broker가 sts token을 받아온다는 점 빼고는 기존 identity federation 동작 방식과 비슷하다.(AssumeRole or GetFederationToken API사용)
  - The identity broker must determine the appropriate IAM policy : The identity broker가 IAM policy를 정의한다.(당연하지만 어떤 역할을 원하는지 aws에 말해줘야 하므로)
- AssumeRoleWithWebIdentity : deprecated이고 Cognito로 대체 가능하다.
- **Federated Identity Pools** using **Cognito**
  - 중요한 세 가지 과정만 기억하자.
  - 1. Identity Provider(Cognito User Pool, 페이스북 로그인, 구글 로그인, SAML 로그인 등)에 로그인 후 토큰을 받는다.
  - 2. 토큰을 Cognito Federated Identity에 넘긴 후 Cognito Federated Identity는 Identity Provider토큰이 유효한 지 검증한다.
  - 3. sts로 temporary credential 리턴

#115 Microsoft Active Directory
- Overview는 간략하게 짚고 넘어간다. **AWS Directory Services 3가지만 외우고 넘어가면 된다.**
- Active Directory helps you organize your company's users, computer and more : Database of objects, 회사 내의 유저 계정, 컴퓨터, 프린터 등을 데이터로 저장
- Centralized security로 domain controller에서 모든 권한을 가지고 하위 오브젝트들을 관리한다.
- Lightweight Directory Access Protocol(LDAP) 사용 : TCP/IP 위에서 디렉터리 서비스를 조회하고 수정하는 응용 프로토콜
- **AWS Directory Services**
  - **AWS Managed Microsoft AD**
    - on-premise AD 사용
    - **MFA** 지원
    - Establish **“trust” connections** with your on-premise AD
  - **AD Connector**
    - on-premise AD 사용
    - **프록시** 사용
  - **Simple AD**
    - on-premise AD를 사용할 수 없을 때 사용
    - ec2 instance사용가능

#116 AWS Organizations
- Global service
- Allows to manage multiple AWS accounts : 1개의 master account와 수 많은 member account로 이루어진다.
- 한 member account는 한 organization에만 속할 수 있다. 
- API is available to automate AWS account creation
- **Multi Account Strategies** : aws organization을 활용할 다양한 방법이 존재한다.
  - Create accounts per department, 
  - per cost center, per dev / test / prod, based on regulatory restrictions (using SCP : Security Control Policy)
  - for better **resource isolation (ex: VPC)**, to have separate per-account service limits, **isolated account for logging**
- **Organizational Units(OU)** : OU는 하나의 그룹이라고 생각하면 된다. OU안에 그룹을 만들면 그룹 안의 계정들을 OU의 속성을 공유한다.
- **Security Control Policy** : **Whitelist or blacklist** IAM actions - allow하거나 deny
  - Applied at the OU or Account level : OU(사실상 Account를 묶은 그룹에 가까움)와 account모두에 적용가능하고 개별 권한 필요한 경우 account에 적용
  - Does not apply to the Master Account : **master account에는 scp를 적용해도 적용되지 않는다.**
  - SCP is applied to all the Users and Roles of the Account, including Root user : 계정에서 iam user를 만들든 루트 계정을 사용하든 상관 없이 scp는 한 계정에 적용하면 그 계정에 **전역적**으로 적용된다.
  - **기본적으로 deny가 default로 적용되고 deny, allow가 공존하는 경우 deny를 우선적용한다.**
- **Moving Accounts**
  - 한 조직은 멤버를 다른 조직으로 옮기려면 조직에서 멤버를 삭제해야 가능
  - 한 조직의 master account를 다른 조직으로 옮기는 것 역시 위와 같이 기존 조직에서 완전히 벗어나야 가능

#117 IAM Roles vs Resource Based Policies
- When you assume a role (user, application or service), you give up your original permissions and take the permissions assigned to the role : assume role은 자신의 권한을 포기하고 다른 사람에게 넘기는 것
- When using a resource based policy, the principal doesn’t have to give up his permissions : 자신의 권한을 포기 안해도 됨

#118 IAM Permission Boundaries
- IAM Permission Boundaries are supported **for users and roles** (not groups)
- Advanced feature to use a managed policy to set the **maximum permissions an IAM entity can get** : IAM 엔티티(유저 혹은 role)가 가질 수 있는 권한의 최대치를 설정할 수 있다.
- 예를 들어 administrator권한을 부여해도 Permission Boundary가 s3한정이면 s3에 한정된 권한을 갖게 된다.

#119 Organization SCP & Permissions Boundary & Identity-based policy
- Organization SCP & Permissions Boundary & Identity-based policy 3가지를 동시에 활용해서 보안 강화 가능
- Identity-based policy : 어떤 identity를 가진 계정 혹은 role에 부여하는, identity를 대상으로 권한을 부여하는 policy
- Resource-based policy : 리소스에 부여하는, 리소스를 대상으로 특정 identity만 허용하겠다는 policy
- https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html : 공식 문서 참고

#120 IAM Policy Evaluation Logic
- 1. Deny Evaluation : 무언가를 deny하는 effect가 있다? 그렇다면 무조건 allow보다 우선적용되어 무조건 deny된다.
- 2. Organization SCP 
- 3. Resource-based policies
- 4. IAM permissions boundaries
- 5. Session policies
- 6. Identity-based policies\

#121 AWS Resource Access Manager(RAM)
- **Share AWS resources that you own with other AWS accounts** : 한 aws계정이 가진 리소스를 다른 aws계정과 쉐어한다.
- Avoid resource duplication : 리소스 복제가 아니라 공유를 지향한다.
- **VPC Subnets**의 경우 RAM이 자주 사용되므로 이 케이스는 꼭 기억하도록 하자. 
- 다이어그램 604p를 참고하면 **private subnet내에서 private ip로 리소스간 접근이 가능해** 보안 그룹끼리만 잘 설정해주면 다른 계정에 있는 서로 다른 리소스 간에 접근이 쉽게 가능하다.

#122 AWS Single Sign-On (SSO)
- Centrally manage Single Sign-On to access multiple accounts and 3rd-party business applications. : On-premise서버 혹은 **Identity Store SAML 2.0 Compatible**과 연동하여 sso에 로그인하면 aws console, business cloud application(slack, dropbox), custom SAML application 모두를 한 로그인으로 사용할 수 있다. 말 그대로 Single Sign On이다.
- **Integrated with AWS Organizations**
- Supports SAML 2.0 markup
- Integration with on-premise Active Directory
- Centralized permission management
- 607p 참고

#123 SSO vs AssumeRoleWithSAML
- **AssumeRoleWithSAML**는 3rd party앱에 로그인하고 sts에 credential요청 후 토큰을 받아 aws resource에 접근한다.
- 반면 **SSO**는 SSO Login Portal(Identity Store SAML 2.0 Compatible과 연동된)에만 로그인하면 credentail을 받아 aws resource에 바로 접근할 수 있고 aws에서 AssumeRoleWithSAML에 비해서 권장하는 방식이다.

#124 AWS KMS(Key Management Service)
- Fully integrated with IAM for authorization
- server side encryption에 효과적
- **Symmetric(AES-256 keys)**
  - Necessary for **envelope encryption** : envelope encryption이란 한 키로 암호화한 것을 다른 키로 한 번 더 암호화하는 것을 말한다. (envelope encryption은 developer exam범위이고 solutions architect범위는 아니다.)
  - You never get access to the Key unencrypted (must call KMS API to use)
- **Asymmetric(RSA & ECC key pairs)**
  - The public key is downloadable, but you can’t access the Private Key unencrypted
  - Use case: Sign/Verify operations, encryption outside of AWS by users who can’t call the KMS API : public key가 downloadable이므로 KMS API를 사용할 수 없는 상황이라면 고려 가능
- Symmetric key와 Asymmetric의 private key는 공통적으로 unencrypted상태로 볼 수 없게 설계되었다.
- Three types of **Customer Master Keys(CMK) : KMS와 동의어이고 KMS가 최신 버전**
  - AWS Managed Service Default CMK: free > 예를 들어 EBS사용할 때 at rest encryption으로 default kms를 사용한다면 요금이 free라는 것이다.
  - 나머지는 커스텀키를 생성하거나 import하는 것인데 개당 $1/month이다.
- aws에서 관리하는 KMS내부의 키는 볼 수 없다. 따라서 모든 보안은 aws에서 맡기는 것이고, rotate key로 보안을 강화할 수 있다. 
- KMS can only help in encrypting up to 4KB of data per call
- If data > 4 KB, use envelope encryption : KMS에서 4KB이상의 데이터는 envelope encryption을 사용한다.
- **To give access to KMS to someone** : IAM Policy뿐만 아니라 Key Policy도 설정해야 한다는 것을 기억하자.
  - Make sure the **Key Policy** allows the user : Key policy 설정 필요
  - Make sure the IAM Policy allows the API calls : IAM Policy 설정 필요

#125 KMS Key Policies
- Control access to KMS keys, “similar” to S3 bucket policies : s3 bucket policy처럼 접근을 제한하는 policy를 말한다.
- **Default KMS Key Policy**
  - policy를 따로 설정하지 않았을 경우 적용되는 policy
  - Complete access to the key to the root user = entire AWS account : 루트 유저를 포함한 IAM의 모든 계정이 이 키에 접근 가능
  - 따라서 이 경우는 IAM policy만 잘 설정해주면 Key Policy가 default로 적용되는 것이다.
- **Custom KMS Key Policy**
  - 커스텀으로 어떤 user, role이 이 키를 사용할 수 있는지 정의한다. 당연히 policy에 따라서 다른 계정도 이 키에 접근할 수 있다.

#126 KMS Automatic Key Rotation
- **For Customer-managed CMK(not AWS managed CMK)** : AWS Managed Key는 3년마다 rotate된다고 한다. Customer Managed Key는 AWS Managed Key보다 보안이 취약할 수 있다고 판단한 것 같다.
- If enabled: automatic key rotation happens every 1 year
- Previous key is **kept active** so you can decrypt old data : 이전 키는 이전 데이터들을 위해서 절대 삭제되면 안된다.
- New Key has the same CMK ID (only the backing key is changed)
- Automatic Key Rotation이 발생하면 CMK ID는 변경되지 않고 내부의 (Backing Key)실제 키 값만 변경된다. 물론 Old Backing Key도 save된다.

#127 KMS Manual Key Rotation
- **90일, 180일** 등의 간격으로 key를 rotate할 수 있다.
- **New Key has a different CMK ID** : 당연히 backing key도 바뀐다. alias만 유지 가능
- Better to use **aliases** in this case (to hide the change of key for the application) : new key를 생성해도 애플리케이션에서 api변경 없이 사용가능할 수 있도록 해주는 것이 alias이다.
- Good solution to rotate CMK that are not eligible for automatic rotation (like asymmetric CMK) : **비대칭키는 automatic rotation을 지원하지 않는다.** 개인적인 의견으로는 비대칭키는 public, private key 2개가 존재해서 자동 관리가 좀 부담스러운 것 같다.

#128 SSM Parameter Store
- Secure storage for configuration and secrets : 설정에 필요한 여러 환경 변수들을 저장하기에 좋은 서비스
- **Serverless**, scalable, durable, **easy SDK**
- Version tracking of configurations / secrets : 버저닝 지원
- Notifications with CloudWatch Events : 시크릿에 만료 기한을 설정하면 나중에 만료 며칠 전인지 event를 발생시킬 수 있다.
- **SSM Parameter Store Hierarchy**
  - /my-department/my-app/dev/db-url : GetParameters로 이 경로에 저장된 키를 받아오는 것이 가능
  - /my-department/my-app/dev/amzn2-ami-hvm-x86_64-gp2 : GetParametersByPath로 PATH자체를 받아오는 것이 가능

#129 AWS Secrets Manager
- Newer service, meant for storing secrets : 시크릿 키 저장에 특화된 Secrets Manager
- Capability to force rotation of secrets every X days : rotation을 x-day마다 돌릴 수 있다.
- Integration with Amazon RDS (MySQL, PostgreSQL, Aurora) : RDS, redshift, documentdb 등의 **데이터베이스**를 사용할 때 권장되는 조합이다.
- 그냥 AWS Secrets Manager 생성 시 유저 이름, 패스워드를 입력하고 원하는 데이터베이스를 고르면 키 관리 + 키 rotation을 알아서 해준다고 보면 된다.

#130 AWS Shield
- AWS Shield Standard
  - Free service : 기본으로 세팅되면 무료
  - Provides protection from attacks such as **SYN/UDP Floods**, **Reflection attacks** and other **layer 3/layer 4 attacks**
  - 참고로 **SYN/UDP Floods**, **Reflection attacks**이 무엇인지는 설명하지 않았다. 출제범위가 아니기 때문이다.
- AWS Shield Advanced
  - $3,000 per month per organization로 굉장히 비싸다.
  - DDoS 방어
  - Protect against higher fees during usage spikes due to DDoS : 디도스로 인해 트래픽이 급격히 늘어나도 요금이 늘어나지 않는다.

#131 CloudHSM
- KMS => AWS manages the software for encryption : KMS가 소프트웨어 측면에서, aws가 직접 키를 관리한다면
- CloudHSM => AWS provisions encryption hardware : CloudHSM는 하드웨어만 제공하고, 유저가 직접 키를 관리한다.
- Dedicated Hardware : 전용 하드웨어
- Supports both symmetric and asymmetric encryption (SSL/TLS keys)
- No free tier available
- **Must use the CloudHSM Client Software**
- **Good option to use with SSE-C(고객 제공 암호화 키) encryption**
- CloudHSM Software가 key와 user를 manage한다. 현재 공부한 바로는 KMS와의 차이점은 KMS는 키 로테이션을 AWS에서 해주지만 CloudHSM에서는 유저가 직접 해야 된다 이 정도 같다.
- **CloudHSM – High Availability**
  - Multi AZ
  - 너무나 당연하게도 이렇게 중요한 보안키는 Highly available해야 하고 High durability를 가져야 하고, 역시 그런 특성을 지니고 있다.
- CloudHSM vs KMS : 631p 필히 참고.

#132 AWS WAF – Web Application Firewall
- Protects your web applications from common web exploits(웹 취약점 공격 - 웹 설계 취약점 등)
- Layer 7(HTTP) 에서 작동
- **Application Load Balancer, API Gateway, CloudFront** : Layer 7에서 작동하므로 이 세가지에만 적용된다. 필히 기억.
- **Web Access Control List** : 어떤 요청을 막을 것인지 rule을 설정하는 것이다. 아래는 rule을 설정해서 막을 수 있는 것들이다.
  - Rules can include: IP addresses, HTTP headers, HTTP body, or URI strings
  - Protects from common attack - **SQL injection and Cross-Site Scripting(XSS)**
  - **geo-match(block countries)**
  - Rate-based rules (to count occurrences of events) – **for DDoS protection**
- **AWS Firewall Manager**
  - Manage rules in all accounts of an AWS Organization : 한 organization의 waf rule, shield advanced, security group for ec2 & eni등을 관리한다.

#133 Amazon GuardDuty
- Intelligent Threat discovery to Protect AWS Account : 머신러닝을 사용하는 서비스로, 계정 내의 이상 징후를 감지한다.
- GuardDuty로의 인풋이 가능한 것들 > 보통 비정상적인 것을 감지한다고 보면 된다.
  - **CloudTrail Logs**: unusual API calls, unauthorized deployments 
  - **VPC Flow Logs**: unusual internal traffic, unusual IP address
  - **DNS Logs**: compromised(위협이 되는) EC2 instances sending encoded data within DNS queries
- 당연히 CloudWatch Event로 이벤트를 발생시킬 수 있다.
- Can **protect against CryptoCurrency attacks**(has a dedicated(전용) “finding” for it) : 암호화폐 공격 방어에 특화되어 있다.

#134 Amazon Inspector
- Automated Security Assessments for EC2 instances : **오직EC2를 위한** 보안 평가 자동화
- OS위에 Inspector Agent를 설치해서 보안 취약점을 평가받을 수 있고, 분석이 끝나면 report를 받아볼 수 있다.
- Analyze against unintended network accessibility, the running OS against known vulnerabilities : 의도하지 않은 네트워크 액세스, OS보안 취약점 분석
- **For Network assessments(agentless)**
  - Network Reachability : 네트워크 도달가능성
- **For Host assessments(with agent)** 
  - Common Vulnerabilities and Exposures : 보편적인 보안취약점 파악
  - Center for Internet Security (CIS) Benchmarks(성능 측정을 목적으로 표준적인 테스트 실행)

#135 Amazon Macie
- Amazon Macie is a fully managed data security and data privacy service that uses **machine learning and pattern matching** to discover and protect your **sensitive data(PII : Personally identifiable information)** in AWS : s3 bucket을 분석해서 sensitive한 data가 존재한다면 notify가능

#136 AWS Shared Responsibility Model
- AWS responsibility - Security **of** the Cloud
  - **Protecting infrastructure(hardware, software, facilities, and networking)** that runs all the AWS service : aws 서비스의 기반 관리는 당연히 aws의 책임이다. 하드웨어는 물론이고, 소비자가 사용하는 software에 이상이 생기면 안된다. 소프트웨어라 함은 바로 위에서 살펴본 macie를 예로 들면, 머신 러닝을 통해서 sensitive data를 파악하는데 sensitivee하지 않은 data를 sensitive하다고 판단하면 안될 것이므로 질적인 측면에서 aws는 기술의 발전에 힘써야 한다.
- Customer responsibility - Security **in** the Cloud
  - For EC2 instance, customer is responsible for management of the guest OS (including security patches and updates), firewall & networkconfiguration, IAM : EC2의 보안은 사용자가 직접 관리한다.
  - Encrypting application data : 역시 encrypt할 것인지 말 지는 사용자가 관리한다. aws는 단순히 서비스만 제공해준다.
- **Shared controls** : aws와 customer가 공유하는 책임
  - Patch Management(버전 패치), Configuration Management(설정 관리), Awareness & Training
- 645p 도형 참고.

#137 VPC(Virtual Private Cloud)
- **CIDR(Classless Inter-Domain Routing) – IPv4**
  - 사이더라고 부르는 최신의 IP 주소 할당 방법.
  - 기존 IP 주소 할당 방법인 네트워크 클래스를 대체했다.
  - 보안 그룹 룰에서 사용하던 ip가 바로 이 CIDR이다.
  - 기존 A,B,C,D,E클래스를 사용하던 것에 비해 사이더는 유연성을 더해준다.
  - classful addressing vs CIDR(왜 CIDR이 classful에 비해서 좋은가) : https://www.practicalnetworking.net/stand-alone/classful-cidr-flsm-vlsm/
- **Private IP(사설 IP) can only allow certain values** : 사설 IP는 특정 값만 할당하도록 IANA가 정한 규칙이 존재한다.
  - 10.0.0.0 – 10.255.255.255 (10.0.0.0/8) > in big networks
  - 172.16.0.0 – 172.31.255.255 (172.16.0.0/12) > **AWS default VPC** in that range
  - 192.168.0.0 – 192.168.255.255 (192.168.0.0/16) > home networks
  - 서브넷 마스크의 범위는 언제든지 바뀔 수 있다.
  - 위 3가지를 제외한 나머지는 전부 공인 IP(Public IP)
- **Default VPC**
  - All new AWS accounts have a default VPC : 모든 aws 계정은 기본 VPC를 가진다.
  - New EC2 instances are launched into the default VPC if no subnet is specified : 직접 만든 VPC가 없다면 새로운 EC2는 default VPC에 할당된다.
  - Default VPC has Internet connectivity and all EC2 instances inside it have public IPv4 addresses : Default VPC는 기본적으로 인터넷에 연결에 가능하도록 설정되어 있어서, ec2 인스턴스는 인터넷의 모두가 알아볼 수 있는 공인 ip를 가진다. 즉 ec2 인스턴스를 생성했는데 보안그룹 포트를 22번으로 설정해주었을 때 ssh로 ec2에 접속가능했던 것이 인터넷이 연결되어 있기 때문이다. 인터넷이 연결되어 있지 않았다면 ec2인스턴스의 공인 ip를 찾을 수 없었을 것이다.
  - **We also get a public and a private IPv4 DNS names** : 공인 ip, 사설 ip가 모두 할당된다.
- **VPC Overview**
  - You can have multiple VPCs in an AWS region(max 5 per region – soft limit) : 한 리전당 최대 5개(소프트 리밋)의 vpc를 가질 수 있다.
  - Max CIDR per VPC is 5, for each CIDR : 위와 비슷하게 VPC당 CIDR을 최대 5개 할당 가능하다.
    - CIDR의 subnet mask는 16부터 28까지이다.
    - Min size is /28 (16 IP addresses) 
    - Max size is /16 (65536 IP addresses)
  - Because VPC is private, only the Private IPv4 ranges are allowed : **VPC는 말 그대로 private하기 때문에 private ip**만 할당 가능하다.
  - Your VPC CIDR should NOT overlap with your other networks : 당연한 말이지만 CIDR은 다른 네트워크와 구별이 가능해야 하므로 다른 네트워크에 덮어씌우면 안된다.
- **Subnet**
  - AWS reserves 5 IP addresses (first 4 & last 1) in each subnet : aws vpc의 서브넷 내의 5개의 사설 ip는 예약된 ip로, ip를 할당할 때 사용할 수 없다.
  - ex) 10.0.0.0, 10.0.0.1 - vpc router, 10.0.0.2 - mapping for dns, 10.0.0.3 - mapping for future use, 10.0.0.255 
- **Internet Gateway(IGW)**
  - VPC내부의 자원들을 인터넷에 연결해주는 역할
  - It scales horizontally and is highly available and redundant : 고가용성, 용장성(일단 여유분 존재로 이해함)
  - **Must be created separately from a VPC** : VPC와 따로 생성해야 한다!
  - **One VPC can only be attached to one IGW and vice versa** : VPC와 IGW는 무조건 1:1매칭이다.
- **Bastion Hosts**
  - private subnet에 인터넷 연결이 아닌 ssh접속만을 원할 때 사용한다.
  - private subnet에 인터넷을 연결하려면 NAT Instance or NAT Gateway를 사용해야 한다.
  - **The bastion is in the public subnet which is then connected to all other private subnets** : 강의 자료에서 설명한 bastion host는 VPC내부에 public subnet과 private subnet이 존재할 때, public subnet내부에 ec2 instance가 있고, 이 ec2 instance만 private subnet에 접근가능하다.
  - **Bastion Host security group must be tightened** : bastion host에 접근하면 사실상 private subnet에 접근할 가능성이 생기는 것이므로, bastion host에 접근할 수 있는 보안 그룹은 tight하게 설정해야 한다. **즉 22번 포트만 필요한 IP를 대상으로 열어놓는 것이 좋다.(시험)** 

#138 Bastion Host Hands-on
- public subnet의 instance(공인 IP 존재)에서 ssh로 private subnet의 인스턴스에 접속하는 것을 진행했다. private subnet에서 인터넷 사용 불가.

#139 NAT Instance Hands-on
- amazon nat instance ami를 사용해 NAT Instance를 만들고, 보안 그룹에 22 : SSH, 80 : HTTP(VPC CIDR), 443 : HTTPS(VPC CIDR), ICMP(VPC CIDR)를 허용한다.
- Source / destination check disable
- private subnet의 route table에 로컬 네트워킹을 제외한 나머지 destination은 NAT Instance를 향하게 설정
- 이후, private subnet의 인스턴스에서 ping, curl등의 인터넷 사용이 가능함
- 결론적으로 private subnet이 인터넷에 노출되지 않은 상태에서 NAT을 이용해 인터넷을 사용할 수 있음

#140 Public Subnet vs Private Subnet
- **강의 자료에는 없지만 따로 정리하는 것**
- subnet을 만들 때 public으로 할지 private으로 할 지 선택란이 있는 것이 아니다.
- 사용자가 이 서브넷은 public으로 할 것이고, 다른 서브넷은 private으로 할 것으로 직접 정하는 것이다. 그렇기에 네이밍이 중요하다.
- 강의에서는 public subnet에 subnet옵션에서 auto-assign public ip를 활성화해서, 공인 ip를 자동할당했다. (아직 도메인 네임 설정을 하지 않았으므로 도메인 네임 존재 x)
- 따라서 인터넷에서 public subnet에 auto assign된 ip로 이 서브넷을 알아볼 수 있고, public subnet또한 IGW, Route table을 통해서 인터넷에 액세스 할 수 있는 양방향의 통신이 가능해졌다.
- 참고로 route table은 public subnet, private subnet에 한 개씩 따로 만들었다.

#141 NAT Instance
- Deprecated되었고, NAT Gateway의 하위호환이다.
- **Not highly available**
  - highly available하게 만드려면 asg를 multi-az에 만드는 등 추가적인 수고가 들어간다.
- Internet traffic bandwidth depends on **EC2 instance type**
- NAT = Network Address Translation
- Allows EC2 instances in private subnets to connect to the Internet : public subnet이 아닌 private subnet이 인터넷에 접속 가능하도록 해준다.
- **Must be launched in a public subnet** : public subnet의 internet gateway, route table을 사용할 것이기 때문이다.
- **Must disable EC2 setting: Source / destination Check** : NAT이름에서도 볼 수 있듯 요청 간의 src, dest를 바꾸기 때문이다.
- Must have **Elastic IP** attached to it : 고정 ip 주소를 가져야만 한다.
- Route Tables must be configured to route traffic from private subnets to the NAT Instance : route table을 로컬 네트워킹할 경우가 아닌 경우 모든 트래픽을 NAT Instance를 바라보게 해야 한다.

#142 NAT Gateway
- AWS-managed NAT, higher bandwidth, high availability, no administration : 관리가 필요없는 데다가 **보안그룹 설정도 필요없다.**
- **NATGW is created in a specific Availability Zone, uses an Elastic IP**
- Can’t be used by EC2 instance in the same subnet (only from other subnets) : private subnet이 아닌 public subnet에 만들어야 한다.
- **Requires an IGW** (Private Subnet => NATGW => IGW) : NAT 게이트웨이 역시 IGW를 사용한다.
- 5 Gbps of bandwidth with automatic scaling up to 45 Gbps
- **NAT Gateway is resilient within a single Availability Zone** : NAT Gateway는 싱글 AZ에 대해서만 회복 탄력성을 가진다.
  - Must create multiple NAT Gateways in multiple AZs for fault-tolerance : fault-tolerance를 위해서는 multi az에 NAT Gateway를 만들어야 한다.
  - There is no cross-AZ failover needed because if an AZ goes down it doesn't need NAT : 한 AZ가 다운되었다면 NAT가 필요 없다.

#143 DNS Resolution in VPC
- **DNS Support** : 이를 사용하면 Route 53 Resolver를 사용할 수 있기에 Custom DNS server를 따로 만들거나 사용할 필요가 없다.
- **DNS Hostnames** : 이를 사용하면 private dns를 사용할 수 있다. 또한 private이기 때문에 도메인 네임을 구매할 필요가 없다.
  - **VPC DNS Hostname을 활성화하면 public subnet에서 public ipv4을 보유하고 있던 인스턴스는 public ipv4 DNS도 갖게 된다.**
- 위 두가지를 동시에 사용하면, Route 53 Resolver에 private 도메인 네임을 쿼리하면 private ip를 되돌려준다. 
- **Route 53 Hosted Zone을 사용해 private domain name 사용하기**
  - 1. VPC에서 enableDnsSupport & enableDnsHostname 활성화(안 해도 설정창에서 나중에 하게 됨)
  - 2. Route 53 Hosted Zone 접속
  - 3. 트래픽을 aws VPC에서 라우팅하는 Private Hosted Zone선택
  - 4. Specific VPC 선택
  - 5. 위에서 설정한 VPC를 사용하는 바스티온 호스트에 접속
  - 6. Route53에 private domain name(ex. google.demo.internal)에 대해서 CNAME레코드의 값으로 www.google.com을 입력하면 www.google.com에 접속할 수 있다.(인터넷 게이트웨이 & Route table & NAT Gateway 활성화 상태)
  - 7. 결론적으로 Route 53 Private Hosted Zone을 사용한다면 VPC안에서 private domain name을 사용할 수 있다. 인터넷 게이트웨이 & Route table & NAT Gateway 활성화 상태라면 private domain name과 public domain name을 동시에 사용할 수 있어서 굉장히 유용하다. (위에서 본 google.demo.internal(private domain name)과 www.google.com(public domain name)을 동시에 사용한 것이 그 예이다.)

#144 Security Groups & NACLs
- NACL : 요청 및 응답에 대해 Stateless하다. Stateless하다는 것은 상태가 없기 때문에 요청 및 응답은 항상 평가된다.
- Security Group : 요청 및 응답에 대해 Stateful하다. Stateful하다는 것은 상태가 있기 때문에 요청 및 응답이 상태에 따라 평가된다. 따라서 Inbound가 허용되었다면 그에 따른 Outbound도 허용되고 vice versa이다.

#145 Network Access Control List (NACL)
- NACL are like a firewall which control traffic from and to subnets : 서브넷 방화벽과 같음
- **One NACL per subnet, new subnets are assigned the Default NACL** : 서브넷과 NACL은 1:1 매핑이다.
  - default NACL은 모든 요청과 응답을 허용한다.
- **Newly created NACLs will deny everything**
  - default가 아닌 커스텀 생성된 NACL은 기본 룰로 모든 요청과 응답을 deny한다.
- NACL are a great way of blocking a specific IP address at the subnet level : 보안 그룹은 ec2 instance level

#146 Ephemeral Ports
- 응답이 요청된 ip의 포트에 전달되기 위해서 임시 포트를 사용하는데, 이것을 ephemeral port라고 부른다.
- 677p 다이어그램 참고 : 임시 포트가 될 수 있는 모든 포트를 inbound, outbound rule에서 허용해줘야 한다.

#147 Security Group vs NACLs
- 679p 표 필히 참고

#148 More On VPC
- 679p 이후는 다이어그램, 예시를 보는 것이 편하다. VPC Section은 자료를 보자.

#149 ClassicLink
- ec2 classic은 현재 만들 수 없음 그냥 배경지식 classic link또한 과거의 ec2 classic 때문에 존재하는 것

#150 VPC Endpoints & Private Link
- VPC Endpoint와 VPC PrivateLink는 동일한 VPC Endpoint를 사용하는데, 차이점은 VPC PrivateLink는 네트워크 로드밸런서나 게이트웨이 로드밸런서를 사용한 커스텀 Endpoint Service를 만든 후 CreateVPCEndpoint에서 find service by name 해야하지만, VPC Endpoint는 커스텀 Endpoint Service를 만들 필요 없이
CreateVPCEndpoint에서 AWS Service를 선택하고 원하는 Interface, Gateway를 고르면 된다.당연히 이를 사용하는 이유는 동일하게 전체 vpc를 노출하지 않고 원하는 서비스에 대해서만 노출하고 private network(aws network)를 사용해 public network를 사용하고 싶지 않을 때 사용한다.

#151 Direct Connect
- **Direct Connect – Connection Types : Lead times are often longer than 1 month to establish a new connection** : 새로운 커넥션을 만드는데 1달 이상이 걸리므로 설계 이전에 인지하고 있어야 한다.

#152 ECMP(Equal-cost multi-path routing) 
- VPN to private virtual gateway : two tunnel은 하나의 forward, 하나의 backward로 동시에 이뤄질 수 없음
- VPN to transit gateway : two tunnel은 두 개의 tunnel을 동시에 사용하므로 ecmp를 구현하고, increase bandwidth이며, 게이트웨이를 통해 전송된 gb만큼 pay함

#153 Route Table
- VPC Peering : route table로 어떤 private ip일 경우 어떻게 라우팅 될 것인지를 각 VPC의 라우팅 테이블마다 정해야 함. VPC Peering hands on 참고
- VPC Endpoint Gateway : 또 하나 VPC Endpoint interface유형과 달리 VPC Endpoint Gateway는 라우트 테이블에 적용되기 때문에 어떤 subnet route table에 적용할 건지 선택해야 한다. ex. private subnet route table에 적용하면 route table에 자동으로 private link가 적용됨(s3, dynamoDB only)
- VPC Endpoint interface유형은 특정 aws서비스를 선택한 후 VPC와 Subnet, security group을 설정해야 함

#154 RPO and RTO
- RPO(Recovery Point Objective) : 쉽게 말하면 세이브 시점이다. 세이브 시점 이후에는 백업이 되지 않았으므로 세이브 시점부터 disaster발생 시점까지는 데이터 손실이 생긴다.
- RTO(Recovery Time Objective) : Disaster발생 시점부터 RTO까지 걸린 시간이 **downtime**이다.

#155 DMS – Database Migration Service
- DMS는 Schema Conversion Tool(SCT)를 사용해서 스키마가 다른 db간의 마이그레이션도 지원한다.
- 또한 Continuous Replication이 가능해서 실행중인 서비스의 db마이그레이션도 점진적으로 가능하다.

#156 AWS DataSync
- 많은 양의 데이터를 on-premise서버에서 aws로 옮기는 서비스
- Amazon S3 (any storage classes – including Glacier), Amazon EFS, Amazon FSx for Windows 
- Move data from your NAS or file system via NFS or SMB : NAS(Network Attached Service)로부터 NFS(Network File System) or SMB(Server Message Protocol)프로토콜을 사용해서 데이터를 전달한다.
- Replication tasks can be scheduled hourly, daily, weekly : 실시간이 아닌 스케쥴링 서비스이다.
- Leverage the DataSync agent to connect to your systems : on-premise 서버에 datasync agent가 설치되어 있어야 한다.

#157 High Performance Computing(HPC)
- **EC2 Enhanced Networking(SR-IOV)** : ENA를 사용함으로써 100 Gbps까지 네트워킹 속도를 올릴 수 있다.
  - Elastic Network Adapter(ENA) up to 100 Gbps
- **Elastic Fabric Adapter(EFA)** : ENA의 강화 버전인 EFA도 존재한다.
  - Improved ENA for HPC, only works for Linux
  - Great for inter-node communications, tightly coupled workloads

#158 CloudFormation
- Terraform같은 Infrastructure as code
- **Resources**: your AWS resources declared in the template(MANDATORY) : Cloudformation으로 코드를 작성할 때 resource는 필수적으로 적어야 한다는 의미이다.
- **CloudFormation StackSets**
  - 한 코드로 여러 계정, 여러 리전의 infrastruture를 제어할 수 있다.

#159 AWS Step Functions
- Build serverless visual workflow to orchestrate your Lambda functions : 다이어그램을 통해 workflow를 그리면, lambda function의 sequence를 만들 수 있게 된다. 한 마디로 람다 function의 순서 및 구성도를 보면서, 실제 진행상황을 관찰하며 실행을 할 수 있는 서비스이다.
- AWS SWF(Simple Workflow Service)
  - **AWS Step Functions이 권장되고 SWF는 deprecated버전이다.**
  - 아래 두 경우일 경우에만 제외하고 모두 Step Function을 사용하면 된다.
  - If you need external signals to intervene in the processes
  - If you need child processes that return values to parent processes

#160 Amazon EMR
- EMR helps creating **Hadoop clusters(Big Data)** to analyze and process vast amount of data
- The clusters can be made of hundreds of EC2 instances, Auto-scaling and integrated with Spot instances

#161 AWS Opsworks
- **Chef & Puppet** help you perform server configuration automatically, or repetitive actions : 서버 config 설정 자동화
- Chef / Puppet have similarities with SSM / Beanstalk / CloudFormation but they’re **open-source tools that work cross-cloud** : Chef, Puppet은 오픈소스이기 때문에 특정 클라우드 업체에 국한되지 않고 사용 가능

#162 AWS WorkSpaces
- Managed, Secure Cloud Desktop : 관리형이자 보안강화형의 클라우드 데스크탑 서비스
- Great to eliminate management of on-premise VDI(Virtual Desktop Infrastructure) : on-premise의 자원을 클라우드에서 사용하기 때문에 on-premise의 VDI를 제거할 수 있다고 표현한 것
- On Demand, pay per by usage : 사용한 만큼 돈 지불
- Microsoft Active Directory와 통합 가능

#163 Cost Explorer 
– **Savings Plan**존재

#164 Well Architected Framework 5 Pillars
- 1.**Operational Excellence** : Includes the ability to run and monitor systems to deliver business value and to continually improve supporting processes and procedures
  - 동작 탁월성 : 비즈니스 가치를 실행하고 모니터링하며 이상이 없는지 관찰할 수 있어야 하며, 지속적으로 프로세스와 절차를 개선시켜나가야한다.
- 2.**Security** : Includes the ability to protect information, systems, and assets while delivering business value through risk assessments and mitigation strategies
  - 보안 : 비즈니스 가치를 전달함과 동시에 위험에 대한 평가를 실행하며, 위험 완화 전략을 세우고 정보, 시스템, 자원들을 보호할 수 있어야 한다.
- 3.**Reliability** : Ability of a system to recover from infrastructure or service disruptions, dynamically acquire computing resources to meet demand, and mitigate disruptions such as misconfigurations or transient network issues
  - 신뢰성 : 인프라나 서비스의 다운이 발생해도, 동적으로 컴퓨팅 리소스들을 수요에 맞게 공급해야 하며 설정 오류, 네트워크 지연과 같은 이슈들을 완화할 수 있어야 한다.
- 4.**Performance Efficiency** : Includes the ability to use computing resources efficiently to meet system requirements, and to maintain that efficiency as demand changes and technologies evolve
  - 퍼포먼스 효율성 : 시스템의 요구에 맞는 컴퓨팅 자원을 사용할 줄 알아야 하며, 그 자원들을 기술 진화에 따라 효율적으로 유지보수할 수 있어야 한다.
- 5.**Cost Optimization** : Includes the ability to run systems to deliver business value at the lowest price point
  - 가장 낮은 가격으로 비즈니스 가치를 전달할 줄 알아야 한다.
- AWS Well-Architected Tool를 사용해서 위 질문에 대해 보완 가능(**AWS Well-Architected Tool** : 위 5가지 핵심 코어에 대한 질문들로 구성)
- 응답 결과에 대한 리포트 제공, best practice 제공

#165 Trusted Advisor
- **High level AWS account assessment** : 얕은 레벨의 AWS 계정 평가, 분석 서비스
- **5 Standards**
  - Cost Optimization
  - Performance
  - Security
  - Fault Tolerance
  - Service Limits
- **Core Checks and recommendations** – all customers : 모든 사용자가 코어 체크를 하거나 추천받을 수 있다.
- Full Trusted Advisor – Available for Business & Enterprise support plans : Business & Enterprise의 유료 플랜을 사용하면 아래 기능을 사용할 수 있다.
  - Ability to set CloudWatch alarms when reaching limits : limit도달 시 알람 제공
  - Programmatic Access using AWS Support API : Trusted Advisor로의 Programmatic Access를 위한 api제공

#166 More Architecture Link
- https://aws.amazon.com/architecture/
- https://aws.amazon.com/solutions/

#177 DNS
- DNS : Domain Name System which translates the human friendly hostnames into the machine IP addresses
- NAME SERVER : resolves DNS queries
- Domain Registrar : 도메인 네임 등록 대행자 > route53, 가비아 등등
- SAA 문제 정리 > #69참고할 것

#178 Route53
- **Except for Alias records, TTL is mandatory for each DNS record**