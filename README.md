# ORBIT — Interactive Solar System

ORBIT은 태양계를 주제로 제작한 스크롤 기반 3D 인터랙티브 웹사이트입니다.

사용자가 스크롤하면 카메라가 태양부터 해왕성까지 이동하며, 각 행성의 특징과 정보를 몰입감 있게 탐험할 수 있도록 구현했습니다. 별도 프레임워크나 빌드 도구 없이 HTML, CSS, Vanilla JavaScript와 Three.js로 구성했습니다.

## 기술 스택

| 기술 | 활용 |
| --- | --- |
| HTML5 | 시맨틱 마크업 및 콘텐츠 구조 |
| CSS3 | 반응형 레이아웃, 타이포그래피, UI 애니메이션 |
| JavaScript (ES6+) | 스크롤 인터랙션 및 애니메이션 제어 |
| Three.js (v0.161.0) | 3D 행성, 카메라, 조명, 텍스처 렌더링 |
| WebGL | 실시간 3D 그래픽 렌더링 |
| GLSL Shader | 행성 대기광 및 지구 야간 조명 구현 |
| Canvas API | 태양 발광 텍스처 및 토성 고리의 초기 텍스처 생성 |

## 주요 구현 기능

1. **스크롤 기반 3D 카메라 이동**
   - 스크롤 위치에 따른 행성 간 전환과 곡선형 비행 경로
   - 카메라 회전, 비행 중 기울기 및 시야각(FOV) 조절
   - 시간 기반 감쇠와 보간을 이용한 부드러운 이동
2. **관측 자료 기반 3D 행성**
   - Solar System Scope의 천체 지도 텍스처 적용
   - 행성별 자전 애니메이션과 태양 방향 조명에 따른 명암 표현
   - 지구 주변의 달 표현
3. **행성별 시각 효과**
   - 지구의 독립적인 구름층과 밤 영역에 표시되는 도시 조명
   - 지구·금성·천왕성·해왕성과 태양의 대기광 효과
   - 토성 고리, 태양 발광 효과 및 별 배경
4. **인터랙티브 UI**
   - 행성 내비게이션과 스크롤 진행률 표시
   - 현재 섹션에 따른 콘텐츠 및 내비게이션 활성 상태 전환
   - 로컬 텍스처 로딩 상태 표시
5. **반응형 웹**
   - 모바일·데스크톱 레이아웃
   - 화면 너비에 따른 카메라 FOV 및 비행 경로 진폭 조정
   - `prefers-reduced-motion`에 따른 CSS 전환·로딩 애니메이션 및 카메라 보간·기울기·FOV 변화 조정
   - 동작 줄이기 설정에서도 행성 자전과 렌더링은 계속 실행

## 프로젝트 구조

GitHub에 게시하는 파일은 다음과 같습니다. 기존 로컬 안내문과 다운로드 보조 스크립트는 `.gitignore`로 제외합니다.

```text
orbit-solar-system/
├── index.html
├── textures/
│   ├── 2k_sun.jpg
│   ├── 2k_mercury.jpg
│   ├── 2k_venus_atmosphere.jpg
│   ├── 2k_earth_daymap.jpg
│   ├── 2k_earth_clouds.jpg
│   ├── 2k_earth_nightmap.jpg
│   ├── 2k_moon.jpg
│   ├── 2k_mars.jpg
│   ├── 2k_jupiter.jpg
│   ├── 2k_saturn.jpg
│   ├── 2k_saturn_ring_alpha.png
│   ├── 2k_uranus.jpg
│   └── 2k_neptune.jpg
├── README.md
└── .gitignore
```

스타일과 JavaScript 모듈은 `index.html`에 포함되어 있습니다. 텍스처 13개는 저장소에 함께 포함하며, 별도 다운로드나 패키지 설치·빌드가 필요하지 않습니다.

## 실행 방법

### 로컬 실행

Python 3가 설치된 환경에서 프로젝트 폴더로 이동한 뒤 실행합니다.

```sh
python3 -m http.server 8000
```

브라우저에서 <http://localhost:8000>에 접속합니다. 종료하려면 터미널에서 `Ctrl+C`를 누릅니다. ES 모듈과 텍스처 로딩을 위해 파일을 직접 여는 대신 HTTP 서버 사용을 권장합니다.

Three.js는 [jsDelivr CDN](https://cdn.jsdelivr.net/npm/three@0.161.0/build/three.module.js)에서 불러오므로 인터넷 연결이 필요합니다. Google Fonts도 외부에서 로드합니다. 3D 화면은 WebGL을 지원하는 브라우저에서 실행해야 합니다.

### GitHub Pages

- **웹사이트:** <https://seulbichoe.github.io/orbit-solar-system/>
- **GitHub 저장소:** <https://github.com/SeulbiChoe/orbit-solar-system>

저장소의 **Settings → Pages → Build and deployment**에서 **Deploy from a branch**, `main` 브랜치와 `/(root)`를 사용합니다. 배포가 완료되면 위 웹사이트 주소 또는 해당 설정 화면의 **Visit site** 링크로 접속할 수 있습니다.

텍스처는 `./textures/파일명`의 상대 경로를 사용하므로 프로젝트 저장소 이름이 포함된 Pages 하위 경로에서도 로드됩니다.

## 기술적 구현 포인트

### Scene / Camera / Renderer

`THREE.Scene`에 구체 행성, 달, 별 배경을 배치하고 `PerspectiveCamera`로 렌더링합니다. `WebGLRenderer`는 고정 배경 캔버스를 사용하며, sRGB 출력과 ACES Filmic 톤 매핑을 적용합니다. 주변광과 두 개의 방향광으로 기본 명암과 보조광을 구성합니다.

### requestAnimationFrame 렌더링 루프

`render()`가 `requestAnimationFrame(render)`로 다음 프레임을 예약합니다. `THREE.Clock`의 프레임 간 시간으로 카메라 이동과 자전을 계산하고, 마지막에 `renderer.render(scene, camera)`를 호출합니다. 시간 간격은 최대 0.05초로 제한합니다.

### 스크롤과 카메라 이동

`scrollUpdate()`는 전체 문서에서의 스크롤 비율로 상단 진행률을 표시하고 현재 섹션을 활성화합니다. 카메라용 `getTravel()`은 인접한 섹션의 `offsetTop` 사이에서 현재 스크롤 위치를 정규화해 **섹션 인덱스 + 구간 진행률**을 반환합니다.

진행값을 시간 기반 감쇠로 부드럽게 만든 뒤 `phase * phase * (3 - 2 * phase)`로 구간 이동을 완화합니다. 행성별 카메라 기준점 사이를 보간하고 사인 함수로 위치·주시점에 곡선형 오프셋을 더합니다. 비행 구간에 따라 카메라 기울기와 FOV도 함께 변합니다.

### Vector3와 lerp 보간

`cameraGoal`, `focusGoal`, `focusCurrent`를 `THREE.Vector3`로 관리합니다. `THREE.MathUtils.lerp`로 진행값과 좌표·회전·FOV를 보간하고, `camera.position.lerp()`와 `focusCurrent.lerp()`로 카메라 위치와 주시점을 목표에 접근시킵니다. 이후 `camera.lookAt()`과 `rotateZ()`로 시선과 기울기를 적용합니다.

### ShaderMaterial 시각 효과

대기광은 표면 법선과 시선 벡터의 내적을 이용해 가장자리 빛을 계산합니다. 지구 야간 조명은 별도 구체의 `ShaderMaterial`에서 법선과 광원 방향으로 밤 영역을 판별하고, 야간 지도에서 밝은 도시 영역을 추출해 가산 합성합니다.

### TextureLoader와 CanvasTexture

`THREE.TextureLoader`가 `./textures/`의 로컬 이미지 13개를 비동기로 로드합니다. 텍스처에 sRGB 색 공간과 지원 범위 내 이방성 필터링을 설정하고 로딩 성공·실패를 UI에 반영합니다.

태양 발광은 Canvas의 방사형 그라디언트를 `CanvasTexture`로 만들어 스프라이트에 적용합니다. 토성 고리는 `RingGeometry`의 UV를 반지름 방향으로 조정하고, Canvas로 초기 띠 텍스처를 생성한 뒤 로컬 고리 이미지가 로드되면 해당 이미지로 교체합니다.

## 이미지 출처 및 라이선스

**텍스처 제작·제공: Solar System Scope (INOVE)**

원본 모음: <https://www.solarsystemscope.com/textures/>

라이선스: [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)

| 사용 이미지 | 파일 |
| --- | --- |
| 태양 | `2k_sun.jpg` |
| 수성 | `2k_mercury.jpg` |
| 금성 대기 | `2k_venus_atmosphere.jpg` |
| 지구 주간·야간·구름 | `2k_earth_daymap.jpg`, `2k_earth_nightmap.jpg`, `2k_earth_clouds.jpg` |
| 달 | `2k_moon.jpg` |
| 화성 | `2k_mars.jpg` |
| 목성 | `2k_jupiter.jpg` |
| 토성·고리 | `2k_saturn.jpg`, `2k_saturn_ring_alpha.png` |
| 천왕성 | `2k_uranus.jpg` |
| 해왕성 | `2k_neptune.jpg` |

제공자 설명에 따르면 이 지도들은 NASA의 고도·이미지 자료와 탐사선 및 허블 우주망원경의 관측 사진을 바탕으로 제작되었습니다. 일부 미관측 영역에는 가상 지형이 보완되어 있으며 색상도 조정되어 있어, 모든 부분이 가공되지 않은 실사 사진인 것은 아닙니다.

이번 배포 준비 과정에서 기존 이미지 파일은 수정하지 않았습니다. 화면에서는 3D 구체·고리 매핑, 조명, 투명도 및 셰이더 효과를 적용합니다. 기존 로컬 파일이 제공자의 원본과 바이트 단위로 동일한지는 별도로 검증하지 않았습니다.

CC BY 4.0은 출처와 라이선스 링크를 표시하고 변경 사항을 알리는 조건으로 복제·공유·수정 및 상업적 이용을 허용합니다. 재배포 시 위 제작자·출처·라이선스 표기를 유지하고, 추가 수정이 있다면 밝혀야 합니다. 제공자가 이 프로젝트를 보증하거나 후원한다는 의미는 아닙니다. 이 텍스처 라이선스는 프로젝트 코드 전체의 라이선스를 뜻하지 않습니다.

## 프로젝트 안내

이 프로젝트는 태양계의 실제 거리와 크기를 정확히 재현하는 천문 시뮬레이터가 아닙니다. 행성 크기·배치·카메라 경로는 시각적 몰입감과 인터랙션 경험을 목적으로 구성했습니다.
