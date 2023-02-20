# cicd-practice

Hello, jt-oh!

Does Continuous Deployment Works?

## Introduction

Github Action 과 AWS CodeDeploy를 이용한 자동배포 파이프라인 구성 실습입니다.
Main 브렌치에 Push 이벤트가 발생할 때마다 배포용 패키지가 AWS S3 에 업로드되고, 업로드된 패키지는 배포환경에 자동 배포됩니다.
배포용 패키지는 .zip 파일이며, 배포환경은 AWS EC2 Instance 로 구성되어 있습니다.

## Concepts

### Github Action

Github Action 은 Github 에서 제공하는 CI/CD 플랫폼입니다.
CI/CD 파이프라인의 동작을 정의하는 파일들은 Git Root Direcotry 하위 .github/workflow 디렉토리에 yml 파일 형식으로 존재합니다.

Github Action 을 구성하는 요소는 다음과 같습니다.

<https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions>

#### Workflow

CI/CD 파이프라인에 대응되는 개념이며, Git Root Direcotry 하위 .github/workflow 디렉토리에 yml 파일에 정의됩니다.

Workflow 를 구성하는 개념과 구성 방법은 다음과 같습니다.

<https://docs.github.com/en/actions/using-workflows/about-workflows#workflow-basics>
<https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#about-yaml-syntax-for-workflows>

#### Variables

Workflow 파일에는 변수 개념이 존재합니다.
사용자는 Workflow 에 Predefined Variable, User-defined Variable, User-defined Secrets 등의 변수를 사용할 수 있습니다.

<https://docs.github.com/en/actions/learn-github-actions/variables>

### AWS S3

AWS S3 는 AWS 에서 제공하는 Cloud Object Storage 입니다.

<https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html>

본 실습에서는 배포용 패키지가 운영환경에 배포될 때 사용할 Application Revision 보관용으로 사용됩니다.

### AWS CodeDeploy

AWS CodeDeploy 는 AWS EC2 Instances, Lambda Functions, ECS 에 Application 자동 배포 기능을 제공하는 배포 서비스입니다.

<https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html>

기본 개념은 다음과 같습니다.

#### CodeDeploy Agent

배포환경이 EC2 Instance 이거나 On-premise 일 경우, 배포환경에 설치되어 배포용 패키지를 배포하는 프로그램입니다.
배포 시 배포스크립트 (appspec.yml) 에 정의된 동작을 실행하여 배포용 패키지를 배포합니다.

배포환경에 CodeDeploy Agent 를 설치하는 방법은 아래와 같습니다.

<https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install.html>

CodeDeploy Agent 의 로그 파일 위치는 /var/log/aws/codedeploy-agent/codedeploy-agent.log 입니다.

#### Application

배포용 서비스에 대응되는 개념입니다.
DeploymentGroup, Revision, Deployment 는 Application 단위로 정의됩니다.

#### Deployment Group

배포환경 그룹에 대응되는 개념입니다.
Instance 의 태그를 이용해 배포환경 그룹을 지정합니다.
배포방식을 지정합니다.

#### Revision

Deployment 마다 사용되는 배포용 패키지에 대응되는 개념입니다.
모든 Revision 은 배포 동작을 정의하는 appspec.yml 이 root directory 에 정의되어 있어야 합니다.
Revision 은 Github Repository 혹은 AWS S3 에 위치해 있어야 합니다.

#### Deployment

각 배포에 대응되는 개념입니다.
Deployment 생성 시 Application, Deployment Group, Revision 을 지정해주어야 합니다.
생성된 Deployment 는 CodeDeploy Agent 에 의해 운영환경에서 배포스크립트가 수행됨에 따라 상태가 변경됩니다.

### Implementation

#### Prerequisite

- Revision 저장용 AWS S3 Bucket (CodeDeploy Application 을 생성할 Region 과 동일한 Region)
- AWSCodeDeployFullAccess 와 AMAZONS3FullAccess 권한이 있는 IAM User
- 배포환경용 EC2 Instance

#### 배포환경 EC2 Instance 용 IAM Role 생성

AWSCodeDeoployFullAccess 와 AMAZONS3FullAccess Policy 를 부여한 EC2 Instance 용 IAM Role 을 생성합니다.

#### 배포환경 EC2 Instnace 에 배포환경 EC2 Instance 용 IAM Role 부여

위에서 생성한 배포환경 EC2 IAM Role 을 배포환경 EC2 Instance 에 부여합니다.

#### CodeDeploy 용 IAM Role 생성

AWSCodeDeployFullAccess 권한을 부여한 CodeDeploy DeploymentGroup 용 IAM Role 을 생성합니다.

#### CodeDeploy Application 생성

CodeDeploy Application 을 생성합니다.

#### CodeDeploy DeploymentGroup 생성

위에서 생성한 CodeDeploy 용 AWSCodeDeployFullAccess 권한이 있는 IAM Role 역할을 가지며, 배포환경 EC2 Instance 를 연결한 Deployment Group 을 생성합니다.
배포 방법을 설정할 수도 있습니다.

#### Github Workflow 정의

Github Checkout Action, AWS Crendential Configuration Action, AWS CLI Command 를 이용해 workflow 를 구성합니다.

<https://github.com/jt-oh/cicd-practice/blob/main/.github/workflows/CDWithCodeDeploy.yml>

AWS Credential 과 같이 외부에 공개되면 안되는 값은 Github Repository Secret 으로 등록하여 사용합니다.

#### 서비스 Application 에 appspec.yml 정의

배포 대상 Root Directory 에 운영환경에 배포 시의 배포 동작을 정의한 appspec.yml 파일을 작성합니다.
설치 경로, 설치 파일 소유권 설정, 설치 전/후 동작 등을 정의할 수 있습니다.

### Conclusion

Github Action 과 AWS CodeDeploy 를 이용해 AWS EC2 Intsance 배포환경에 배포 페키지를 자동 배포하는 CD 파이프라인을 구성하였습니다.
CD 파이프라인은 반복적인 배포 과정을 자동화하여 개발자와 인프라 관리자가 그들의 자원을 더 중요한 곳에 투자할 수 있게 해주기 때문에 중요하다고 생각합니다.
다양한 실습 프로젝트를 시작하기 앞서 효율적으로 프로젝트를 진행할 수 있게 프로젝트를 구성하는 방법을 익힐 수 있었습니다.

추후에는 패키지 자동 배포를 넘어 Container 자동 배포, Infrastructure As Code Tool 을 이용해 AWS Service 간 구성 및 설정에 소요되는 시간을 줄여볼 예정입니다.

### References

- <https://blog.bespinglobal.com/post/github-action-%EC%9C%BC%EB%A1%9C-ec2-%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/>
- <https://docs.github.com/en/actions/learn-github-actions>
- <https://awscli.amazonaws.com/v2/documentation/api/latest/reference/deploy/index.html#cli-aws-deploy>
- <https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html>
- <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html>
- <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html>