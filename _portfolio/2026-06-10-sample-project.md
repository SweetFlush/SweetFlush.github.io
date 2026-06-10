---
title: "유니티 프로젝트 포트폴리오 템플릿 (예시)"
excerpt: "프로젝트에 대한 짧은 한 줄 요약을 적어주세요. (예: ECS 기반 대규모 군집 시뮬레이션 프로젝트)"
header:
  teaser: "/assets/images/teaser-placeholder.png" # 포트폴리오 목록에 보일 대표 이미지 경로
  overlay_image: "/assets/images/header-placeholder.png" # 포스트 상단 헤더 배경 이미지 경로
  overlay_filter: 0.5 # 배경 어둡기 조절 (0.1 ~ 1.0)
sidebar:
  - title: "역할"
    image: "/assets/images/gunseaGIF.gif"
    image_alt: "logo"
    text: "Unity Client Developer (1인 개발)"
  - title: "대표 링크"
    text: "<a href='https://github.com/SweetFlush' class='btn btn--info' target='_blank'><i class='fab fa-fw fa-github'></i> GitHub</a><br><a href='#' class='btn btn--success'><i class='fas fa-fw fa-download'></i> Play Demo</a>"
classes: wide
---

이 문서는 새로운 유니티 프로젝트 포트폴리오를 작성할 때 복사하여 사용할 수 있는 **작성 템플릿 및 가이드**입니다. 각 항목을 본인의 실제 프로젝트 정보로 변경하여 포트폴리오를 완성해 보세요.

---

## 📌 1. 프로젝트 개요 (Overview)

| 항목 | 내용 |
| :--- | :--- |
| **개발 기간** | 2026.01 - 2026.03 (약 2개월) |
| **개발 인원** | 1명 (개인 개발) |
| **엔진 & 플랫폼**| Unity 2022.3 LTS / PC (Windows), Android |
| **사용 기술** | C#, Unity UGUI, Addressables, ScriptableObjects |
| **장르 / 한 줄 요약**| 장르를 입력하세요 (예: 2D Grid-based Tycoon Game) |

> **핵심 목표:**  
> 이 프로젝트를 통해 구현하고자 했던 주요 목표를 작성하세요.  
> (예: 대용량 데이터의 안정적인 직렬화 및 로드 시스템 구축, 격자 무늬 그리드 기반의 최적화된 길찾기 시스템 개발 등)

---

## 🛠️ 2. 사용 기술 및 라이브러리 (Tech Stack)

사용했던 주요 기술 스택과 선정 이유를 간단히 정리합니다.

*   **Unity UI (UGUI)**: MVC/MVP 패턴 구조를 적용하여 UI 비즈니스 로직과 뷰의 의존성을 분리해 유지보수성을 극대화했습니다.
*   **Addressable Asset System**: 프로젝트의 리소스 에셋들을 메모리 릭 없이 동적으로 로드 및 해제하도록 관리했습니다.
*   **ScriptableObjects**: 게임 내 다양한 데이터 군(아이템 스키마, 캐릭터 기본 능력치 등)을 데이터베이스화하여 유연하게 기획 데이터를 제어했습니다.
*   **Jekyll & Minimal Mistakes**: 포트폴리오 페이지 구축 및 반응형 웹 인터페이스 적용.

---

## 🚀 3. 핵심 구현 내용 (Key Implementations)

본인이 프로젝트에서 가장 중점적으로 직접 설계하고 개발한 시스템을 기재합니다.

### A. 커스텀 UI 프레임워크 (Custom UI Framework)
*   **의도**: 다중 팝업 및 화면 전환 시 UI 스택을 안정적으로 관리하고 계층 간 데이터 전달을 정형화하기 위해 개발했습니다.
*   **구현 방법**: 
    *   UI Base 클래스를 정의하여 공통 라이프사이클(`OnOpen`, `OnClose`, `OnBack`) 관리.
    *   UI Manager를 통해 스택 형태로 팝업 관리 및 에셋 번들 동적 로드 연동.

### B. 최적화된 데이터 관리 시스템 (Data Serialization)
*   **의도**: 대규모 기획 데이터를 바이너리나 JSON 포맷으로 압축하고 암호화하여 로컬에 안전하게 저장하기 위해 설계했습니다.
*   **구현 방법**:
    *   `Newtonsoft.Json` 라이브러리를 래핑한 직렬화 유틸리티 구축.
    *   비동기 I/O를 활용해 저장 중 프리징 현상 방지.

---

## 🔍 4. 문제 해결 사례 (Troubleshooting)

개발 과정 중 겪었던 어려운 이슈와 이를 논리적으로 해결해 나간 과정을 기록합니다. 가장 큰 기술적 역량을 보여줄 수 있는 부분입니다.

### 🔴 이슈: 에셋 동적 로드 시 프레임 드랍 및 메모리 누수 발생
*   **상황**: 스테이지를 전환하거나 새로운 캐릭터 리소스를 로드할 때 화면이 약 0.5초 동안 멈추는 프레임 드랍 현상이 관찰되었습니다. Profiler를 통해 확인한 결과, 가비지 컬렉터(GC) 부하 및 메모리 파편화가 원인이었습니다.
*   **분역 및 원인**: 
    1.  기존 `Resources.Load` 방식을 사용하여 한 번에 무거운 텍스처와 에셋을 동기적으로 불러와 메인 스레드를 점유함.
    2.  에셋을 파괴(`Destroy`)한 이후에도 언로드(`Resources.UnloadUnusedAssets`)를 즉각 호출하지 않아 참조가 풀리지 않은 메모리가 잔존함.
*   **해결 방법**:
    *   에셋 로드 방식을 **Unity Addressables (비동기)** 구조로 전면 교체하여 로딩 연산을 백그라운드 프레임으로 분산시켰습니다.
    *   에셋의 생성 주기를 참조 카운트 기반으로 관리하는 래퍼 클래스를 만들어, 화면에서 사라진 오브젝트의 메모리가 확실히 해제(`Addressables.Release`)되도록 구조화했습니다.
*   **결과**: 
    *   스테이지 전환 시 메모리 누수가 100% 해소되었습니다.
    *   로딩 중 발생하던 스파이크 프레임 드랍이 사라지고 평균 프레임이 **60 FPS**로 안정화되었습니다.

---

## 📺 5. 플레이 데모 및 스크린샷 (Demo)

여기에 프로젝트를 직관적으로 보여줄 수 있는 사진이나 영상을 첨부합니다.

![플레이 화면 예시](https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80)
*설명: 격자 그리드 기반으로 캐릭터들이 동작하고 있는 인게임 화면 데모 이미지입니다.*

---

## 🔗 6. 관련 링크 (Links)
*   💻 **GitHub Repository**: [GitHub 링크 바로가기](https://github.com/SweetFlush)
*   💾 **다운로드 빌드 파일**: [Itch.io 또는 구글 드라이브 링크]
