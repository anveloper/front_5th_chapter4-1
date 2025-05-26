## 프론트엔드 배포 파이프 라인

> Chapter 4-1. **인프라 관점의 성능 최적화**

### 개요
![Frontend deploy pipeline](https://github.com/user-attachments/assets/ae115427-e6d4-40e2-97d1-4525da36f392)

- 프로젝트의 코드가 수정되어 main 브랜치에 병합될 경우 git actions가 동작합니다.
  - .yml에서 `*.md` 파일은 예외처리하여, 단순 README.md 수정은 git actions 가 트리거 되지 않도록 예외처리했습니다.
- Git actions으로 가상 머신에서 빌드된 정적 파일들을 S3 Sync로 배포가 됩니다.
- S3 배포 이후, CloudFront 엣지 로케이션에 캐싱된 이전 파일들을 무효화합니다.
- 사용자는 S3 혹은 CloudFront 도메인으로 접근이 가능합니다.
  - s3-website 로 접근 시는 원본에 바로 접근하며, http 만 접근이 가능합니다.
  - CloudFront 접근 시에는 엣지 로케이션에 저장된 캐싱 정보가 있다면 캐싱을 먼저 제공하고, 없는 경우에 원본을 제공합니다.


### 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://anveloper.test.s3-website.ap-northeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d389dbmlocu44c.cloudfront.net

### 주요 개념

- Repository secret과 환경변수
  - GitHub에서 제공하는 보안 기능으로, 액세스 키, 비밀번호, 토큰 등의 민감한 정보를 **워크플로우 파일 내에 직접 작성하지 않고 안전하게 참조**할 수 있도록 도와줍니다.
  - 이 값은 GitHub Actions의 워크플로우 실행 시 `${{ secrets.SECRET_NAME }}` 형식으로 사용할 수 있습니다.
  - 환경변수는 특정 job이나 step의 실행 환경에 영향을 미치는 변수로, env: 키를 통해 선언할 수 있습니다.
  - secrets와 환경변수를 적절히 활용하면 공개 Repository에서도 CI/CD 파이프라인의 보안성과 유연성을 높일 수 있습니다. 

- GitHub Actions과 CI/CD 도구
  - `GitHub Actions`은 GitHub에서 제공하는 자동화 도구로, 코드 변경이 감지되었을 때 테스트, 빌드, 배포 등의 작업을 자동으로 실행할 수 있도록 워크플로우를 구성할 수 있습니다.
  - 이는 CI(Continuous Integration)와 CD(Continuous Deployment 또는 Delivery)를 위한 대표적인 도구 중 하나입니다.
  - `CI/CD`는 개발자의 코드 변경 사항을 빠르고 안정적으로 배포하기 위한 일련의 자동화된 프로세스를 의미합니다.
  - GitHub Actions는 `.github/workflows/` 디렉토리에 정의된 YAML 파일을 기반으로 작동하며, 다양한 운영체제와 런타임 환경을 선택할 수 있습니다.
  - 러너를 통해 가상머신에서 빌드된 파일을 연동되는 다른 서비스들에 배포하거나 lint, e2e 테스트 환경으로도 사용이 가능합니다.
  - 과제에선 S3와 CloudFront로만 접근하였지만, AWS의 EKS에 docker 이미지를 배포한다거나, EC2에 직접 구축한 환경으로도 배포를 자동화할 수 있습니다.

- S3와 스토리지
  - S3(Simple Storage Service)는 AWS에서 제공하는 **객체 스토리지 서비스**로, 정적 파일, 이미지, 동영상, 백업 데이터 등을 저장하는 데 사용됩니다.
  - 특히 정적 웹사이트를 호스팅할 때 S3를 이용하면 HTML, CSS, JavaScript 등의 정적 파일을 손쉽게 업로드하고 공개할 수 있습니다.
  - 버킷이라는 단위로 파일을 분류하며, 각 객체는 고유한 키를 가집니다.
  - S3는 고가용성, 확장성, 내구성을 보장하며, 퍼블릭 접근 여부를 설정할 수 있고 IAM 정책이나 버킷 정책을 통해 권한을 세밀하게 제어할 수 있습니다.
  - 과제에서는 정적 파일의 저장소로만 사용되었지만, 일반 스토리지로써도 파일별로 고유한 키를 통해 데이터에 접근이 가능하며, 보안 및 권한 설정 등을 통해 접근을 제어하기 용이합니다.
  - AWS의 다양한 다른 서비스 들과 연동이 용이하며, CloudFront와 같은 지역에 사용시 별도의 데이터 소모없이 사용이 가능합니다.

- CloudFront와 CDN
  - CloudFront는 AWS에서 제공하는 **글로벌 콘텐츠 전송 네트워크(Content Delivery Network)** 서비스입니다.
  - 전 세계에 위치한 **엣지 로케이션(Edge Location)** 을 통해 사용자에게 더 빠르게 콘텐츠를 전송할 수 있도록 도와줍니다.
  - CloudFront는 S3, EC2, ALB 등 다양한 오리진을 지원하며, 정적 파일뿐 아니라 동적 요청에도 사용할 수 있습니다.
  - 이를 통해 사이트의 응답 속도를 높이고, 지연 시간을 줄이며, 전송 효율을 높일 수 있습니다.
  - 또한 HTTPS와 보안 설정, 캐시 제어, 사용자별 콘텐츠 제한 등의 고급 기능도 제공됩니다.

- 캐시 무효화(Cache Invalidation)
  - S3에 새로운 파일을 업로드하거나 기존 파일을 수정해도 CloudFront 캐시에 이전 버전이 남아 있을 수 있습니다.
  - 이를 해결하기 위해 캐시 무효화(Cache Invalidation)를 수행합니다.
  - 캐시 무효화는 CloudFront에 특정 경로나 파일에 대해 새로운 버전을 요청하게 만들며, `aws cloudfront create-invalidation` 명령어 또는 API를 통해 실행할 수 있습니다.
  - 무효화는 전체 경로`("/*")`나 선택적 경로`(/index.html, /css/*)`로 지정할 수 있습니다.

### CDN과 성능최적화

-
