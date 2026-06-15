# OpenShift Migration Advisor 문서 한국어 번역

이 저장소는 [OpenShift Migration Advisor Docs](https://kubev2v.github.io/openshift-migration-advisor-docs/docs/)의 한국어 번역본입니다.

- 원문 저장소: `kubev2v/openshift-migration-advisor-docs`
- 원문 라이선스: CC BY 4.0
- 번역 기준일: 2026-06-16

## 구성 방식

원문과 동일하게 Hugo + Docsy 테마 구조를 사용합니다. 표, 링크, iframe, 이미지 경로, 코드블록, front matter를 유지하고 본문 텍스트를 한국어로 번역했습니다.

## GitHub Pages 설정

이 저장소는 Hugo 빌드를 위해 GitHub Actions 방식으로 배포해야 합니다.

1. Repository → Settings → Pages
2. Source: **GitHub Actions** 선택
3. 저장 후 `main` 브랜치에 push하면 `.github/workflows/hugo.yml`이 Pages를 배포합니다.

웹 주소:

```text
https://yesol1108.github.io/openshift-migration-advisor-docs-ko/
```
