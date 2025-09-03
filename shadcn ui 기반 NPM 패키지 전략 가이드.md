## 전략 개요

### 목표

여러 Next.js 서비스에서 재사용 가능한 관리자 페이지 컴포넌트를 NPM 패키지로 중앙 관리하여 개발 생산성을 높이고 일관성을 유지한다.

### 핵심 원칙

- **독립성**: 각 서비스는 독립적으로 패키지 버전 관리
- **확장성**: 공통 컴포넌트를 상속받아 서비스별 커스터마이징 가능
- **유지보수성**: 중앙에서 업데이트하면 모든 서비스에 반영 가능

### 기술 스택 선택 이유

- **shadcn/ui**: 소스코드 복사 방식으로 의존성 충돌 없음, 패키지 버전 업데이트와 무관하게 안정적
- **tsup**: 가장 간단한 번들링 도구로 ESM/CJS 동시 지원, 설정 최소화
- **changesets**: 체계적인 버전 관리와 변경 이력 추적, 자동 CHANGELOG 생성
- **GitHub Packages**: Private 패키지 무료 호스팅, 기존 GitHub 권한 체계 활용

## 패키지 구조 설계

### 컴포넌트 계층 구조

```
components/
├── ui/          # shadcn/ui 원본 (수정 최소화)
├── admin/       # 비즈니스 로직이 포함된 관리자 컴포넌트
└── composite/   # ui와 admin을 조합한 고수준 컴포넌트
```

각 계층의 역할과 책임:

**ui 레이어 (기초 컴포넌트)**

- 순수 UI 컴포넌트로 비즈니스 로직 없음
- shadcn/ui에서 복사한 Button, Table, Dialog 등
- 스타일링과 기본 인터랙션만 담당
- 모든 프로젝트에서 범용적으로 사용 가능

**admin 레이어 (비즈니스 컴포넌트)**

- 관리자 페이지 특화 비즈니스 로직 포함
- CRUD 작업, 데이터 검색/정렬/필터링 로직 내장
- API 통신 패턴과 에러 처리 표준화
- 권한 체크, 유효성 검증 등 공통 로직 구현
- 예: DataTable (검색/페이지네이션 포함), AdminForm (검증 로직 포함)

**composite 레이어 (완성형 컴포넌트)**

- ui와 admin 컴포넌트를 조합한 페이지 단위 컴포넌트
- 완전한 워크플로우를 구현한 즉시 사용 가능한 모듈
- 예: UserManagementPage (테이블+폼+다이얼로그 조합)

계층 간 의존성: composite → admin → ui (단방향)

### 모듈 분류 전략

- **Core**: 모든 서비스가 사용하는 필수 컴포넌트 (DataTable, Form, Layout)
- **Extensions**: 선택적 기능 모듈 (Charts, Calendar, FileUpload)
- **Themes**: 서비스별 테마 커스터마이징 지원

### Export 전략

```typescript
// 개별 export로 트리셰이킹 최적화
export { DataTable } from './components/admin/DataTable';
export type { DataTableProps } from './components/admin/DataTable';

// shadcn/ui는 필요한 것만 재수출
export { Button, buttonVariants } from './components/ui/button';
```

## 개발 워크플로우

### 로컬 개발 환경

```bash
# 패키지 프로젝트에서
npm link

# 사용 프로젝트에서
npm link @your-company/admin-kit
```

### 테스트 전략

- **Storybook**: 컴포넌트 독립 테스트 및 문서화, 웹에서 인터랙티브 데모 제공
- **예제 프로젝트**: 실제 사용 환경 시뮬레이션
- **CI/CD**: PR시 자동 빌드 및 타입 체크

### 웹 데모 사이트 구축

**Storybook 배포 (추천)**

- 모든 컴포넌트를 웹에서 직접 테스트 가능
- Vercel/Netlify에 자동 배포 설정
- 각 컴포넌트의 props를 실시간으로 변경하며 테스트
- 접근 URL: `https://your-company-admin-kit.vercel.app`

**Next.js 데모 앱**

- 실제 관리자 페이지 전체 플로우 체험
- 샘플 데이터로 CRUD 작업 시뮬레이션
- 다크모드, 반응형 등 실제 환경 테스트
- 소스코드 함께 제공으로 구현 예시 역할

**Sandbox 환경**

- CodeSandbox/StackBlitz 템플릿 제공
- 브라우저에서 바로 코드 수정 및 테스트
- 설치 없이 즉시 체험 가능
- 팀원/고객에게 공유 용이

### 문서화 방식

- 각 컴포넌트에 JSDoc 주석 필수
- Storybook으로 시각적 문서 자동 생성
- README에 마이그레이션 가이드 포함

## 사용 프로젝트 통합

### Tailwind 설정 (필수)

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{ts,tsx}',
    './node_modules/@your-company/admin-kit/**/*.js',
  ]
}
```

### 프로젝트별 커스터마이징

- **CSS 변수로 테마 오버라이드**: 색상, 간격, 폰트 등을 CSS 변수로 조정
- **컴포넌트 확장으로 기능 추가**: 상속이 아닌 Wrapper 컴포넌트로 확장
- **Composition 패턴으로 조합**: 여러 컴포넌트를 조합해 새로운 컴포넌트 생성

커스터마이징 예시:

```typescript
// 기본 컴포넌트 확장
function CustomDataTable(props) {
  return (
    <DataTable 
      {...props}
      className="custom-table"
      onEdit={(item) => {
        // 서비스별 로직 추가
        logEvent('edit', item);
        props.onEdit?.(item);
      }}
    />
  );
}
```

## 웹 테스트 환경 구축

### Storybook 설정

```bash
# 패키지 프로젝트에서
npx storybook@latest init
npm run storybook # 로컬 테스트

# stories 작성
# src/components/admin/DataTable.stories.tsx
```

Storybook 배포:

1. GitHub Pages: 무료, GitHub Actions 자동 배포
2. Vercel: PR별 프리뷰 URL 자동 생성
3. Chromatic: Storybook 전용 호스팅, 시각적 회귀 테스트 포함

### 데모 사이트 구성

```
demo-site/
├── app/
│   ├── page.tsx          # 컴포넌트 목록
│   ├── examples/
│   │   ├── data-table/   # DataTable 예제
│   │   ├── forms/        # Form 예제
│   │   └── dashboard/    # 전체 대시보드 예제
│   └── playground/       # 실시간 props 조작 가능
```

### 접근성 제공

- **공개 URL**: 팀원 누구나 접근 가능
- **인증 옵션**: 민감한 경우 Basic Auth 설정
- **버전별 URL**: `/v1`, `/v2` 등으로 버전별 데모 유지
- **모바일 지원**: QR 코드로 모바일 테스트 용이

## 초기 구축 체크리스트

### 필수 준비사항

- [ ] Node.js 18+ 설치 확인
- [ ] GitHub 계정 및 조직 생성
- [ ] GitHub Personal Access Token 발급 (packages 권한 포함)
- [ ] 환경변수 설정 (`NPM_TOKEN`)

### 패키지 개발 단계

- [ ] 프로젝트 초기화 및 의존성 설치
- [ ] shadcn/ui 컴포넌트 설치 (button, table, form, dialog)
- [ ] 첫 번째 admin 컴포넌트 작성 (DataTable 권장)
- [ ] Storybook 설정 및 stories 작성
- [ ] 빌드 테스트 (`npm run build`)
- [ ] 로컬 링크 테스트 (`npm link`)

### 배포 준비

- [ ] changeset 초기화
- [ ] GitHub Packages 설정 (.npmrc)
- [ ] 첫 버전 배포 (0.0.1)
- [ ] Storybook 웹 배포 (Vercel/Netlify)
- [ ] 데모 사이트 구축 및 배포
- [ ] 테스트 프로젝트에서 설치 확인

### 운영 준비

- [ ] README 작성 (설치 방법, 사용 예시)
- [ ] GitHub Actions 자동 배포 설정
- [ ] 팀 내 사용 가이드 공유

### 단계별 구현

1. **Phase 1**: 핵심 CRUD 컴포넌트 (DataTable, Form, Modal)
2. **Phase 2**: 레이아웃 시스템 (Sidebar, Header, Navigation)
3. **Phase 3**: 고급 기능 (Dashboard, Analytics, Reports)

### 성능 최적화

- 코드 스플리팅을 위한 dynamic import 지원
- 큰 컴포넌트는 별도 엔트리 포인트 제공
- 번들 사이즈 모니터링 자동화

### 마이그레이션 전략

- 기존 프로젝트는 점진적 교체
- 신규 프로젝트는 처음부터 패키지 사용
- Breaking Changes시 codemods 제공 고려