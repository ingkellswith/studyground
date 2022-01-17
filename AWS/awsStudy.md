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