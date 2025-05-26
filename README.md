## 프론트엔드 배포 파이프 라인

> Chapter 4-1. **인프라 관점의 성능 최적화**

### 개요
![Frontend deploy pipeline](https://github.com/user-attachments/assets/ae115427-e6d4-40e2-97d1-4525da36f392)

1. 프로젝트의 코드가 수정되어 main 브랜치에 병합될 경우 git actions가 동작합니다.

   - .yml에서 `*.md` 파일은 예외처리하여, 단순 README.md 수정은 git actions 가 트리거 되지 않도록 예외처리했습니다.

2. Git actions으로 가상 머신에서 빌드된 정적 파일들을 S3 Sync로 배포가 됩니다.
3. S3 배포 이후, CloudFront 엣지 로케이션에 캐싱된 이전 파일들을 무효화합니다.
4. 사용자는 S3 혹은 CloudFront 도메인으로 접근이 가능합니다.

   -  s3-website 로 접근 시는 원본에 바로 접근하며, http 만 접근이 가능합니다.
   -  CloudFront 접근 시에는 엣지 로케이션에 저장된 캐싱 정보가 있다면 캐싱을 먼저 제공하고, 없는 경우에 원본을 제공합니다.


### 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://anveloper.test.s3-website.ap-northeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d389dbmlocu44c.cloudfront.net

### 주요 개념

- GitHub Actions과 CI/CD 도구:
- S3와 스토리지:
- CloudFront와 CDN:
- 캐시 무효화(Cache Invalidation):
- Repository secret과 환경변수:

---

### CDN과 성능최적화

-
