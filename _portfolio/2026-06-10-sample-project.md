---
title: "유니티 프로젝트 포트폴리오 템플릿 (예시)"
excerpt: "프로젝트에 대한 짧은 한 줄 요약을 적어주세요. (예: ECS 기반 대규모 군집 시뮬레이션 프로젝트)"
header:
  teaser: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80" # 포트폴리오 목록에 보일 대표 이미지 경로
  overlay_image: "https://images.unsplash.com/photo-1542751371-adc38448a05e?auto=format&fit=crop&w=1600&q=80" # 포스트 상단 헤더 배경 이미지 경로
  overlay_filter: 0.5 # 배경 어둡기 조절 (0.1 ~ 1.0)
sidebar:
  - title: "역할"
    image: "/assets/images/gunseaGIF.gif"
    image_alt: "logo"
    text: "Unity Client Developer (1인 개발)"
  - title: "대표 링크"
    text: "<a href='https://github.com/SweetFlush' class='btn btn--info' target='_blank'><i class='fab fa-fw fa-github'></i> GitHub</a><br><a href='#' class='btn btn--success'><i class='fas fa-fw fa-download'></i> Play Demo</a>"
classes: wide
gallery:
  - url: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80"
    image_path: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80"
    alt: "인게임 스크린샷 1"
    title: "그리드 배치 시스템 화면"
  - url: "https://images.unsplash.com/photo-1511512578047-dfb367046420?auto=format&fit=crop&w=800&q=80"
    image_path: "https://images.unsplash.com/photo-1511512578047-dfb367046420?auto=format&fit=crop&w=800&q=80"
    alt: "인게임 스크린샷 2"
    title: "유저 인터페이스 및 통계 패널"
  - url: "https://images.unsplash.com/photo-1542751371-adc38448a05e?auto=format&fit=crop&w=800&q=80"
    image_path: "https://images.unsplash.com/photo-1542751371-adc38448a05e?auto=format&fit=crop&w=800&q=80"
    alt: "인게임 스크린샷 3"
    title: "플레이어 데이터 로드 확인"
---

이 문서는 새로운 유니티 프로젝트 포트폴리오를 작성할 때 복사하여 사용할 수 있는 **작성 템플릿 및 미디어 가이드**입니다. 각 항목을 본인의 실제 프로젝트 정보로 변경하여 포트폴리오를 완성해 보세요.

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

---

## 🚀 3. 핵심 구현 내용 & 코드 예시 (Key Implementations)

본인이 프로젝트에서 가장 중점적으로 직접 설계하고 개발한 시스템을 기재하며, 관련 핵심 C# 소스 코드를 추가하여 전문성을 돋보이게 합니다.

### A. 비동기 에셋 로딩 매니저 (Addressable Asset Manager)
*   **의도**: 동적 에셋 로딩 과정에서 발생하는 프레임 드랍을 방지하고 메모리 관리를 일관성 있게 자동화하기 위해 개발했습니다.
*   **구현 방법**: UniTask를 사용해 비동기 처리를 직관화하고, 딕셔너리로 에셋 핸들을 참조 관리하여 중복 로딩을 방지했습니다.

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;
using Cysharp.Threading.Tasks;

public class AssetLoadManager : MonoBehaviour
{
    private Dictionary<string, AsyncOperationHandle> m_LoadedHandles = new();

    /// <summary>
    /// 지정된 어드레스의 에셋을 비동기로 로드하고 참조를 캐싱합니다.
    /// </summary>
    public async UniTask<T> LoadAssetAsync<T>(string address) where T : Object
    {
        if (m_LoadedHandles.TryGetValue(address, out var cachedHandle))
        {
            return cachedHandle.Result as T;
        }

        AsyncOperationHandle<T> handle = Addressables.LoadAssetAsync<T>(address);
        m_LoadedHandles.Add(address, handle);

        await handle.ToUniTask();

        if (handle.Status == AsyncOperationStatus.Succeeded)
        {
            return handle.Result;
        }

        Debug.LogError($"[AssetLoadManager] Failed to load asset at: {address}");
        m_LoadedHandles.Remove(address);
        return null;
    }

    /// <summary>
    /// 캐싱된 에셋의 메모리 할당을 해제합니다.
    /// </summary>
    public void UnloadAsset(string address)
    {
        if (m_LoadedHandles.TryGetValue(address, out var handle))
        {
            Addressables.Release(handle);
            m_LoadedHandles.Remove(address);
            Debug.Log($"[AssetLoadManager] Asset successfully released: {address}");
        }
    }
}
```

---

## 🔍 4. 문제 해결 사례 (Troubleshooting)

개발 과정 중 겪었던 어려운 이슈와 이를 논리적으로 해결해 나간 과정을 기록합니다. 가장 큰 기술적 역량을 보여줄 수 있는 부분입니다.

### 🔴 이슈: 대규모 몬스터 AI 업데이트 시 심각한 프레임 스파이크 발생
*   **상황**: 인게임 상에서 다수의 AI 몬스터가 길찾기를 통해 플레이어를 추적할 때 프레임이 순간적으로 **15 FPS**까지 떨어지는 현상이 관찰되었습니다.
*   **분석 및 원인**: 
    1.  유니티 기본 `NavMeshAgent`가 매 프레임 목적지 갱신(`SetDestination`)을 연산하여 메인 스레드 점유율이 85% 이상 상승함.
    2.  모든 몬스터가 동시 프레임에 길찾기 경로를 계산(Pathfinding Calculation)하는 스파이크 병목 현상 확인.
*   **해결 방법**:
    *   **코루틴 프레임 분산 연산 (Coroutine Throttling)**: 모든 AI가 동일한 프레임에 갱신되지 않도록 갱신 주기를 난수(`Random.Range(0.2s, 0.4s)`)로 분산 제어했습니다.
    *   **경로 계산 최소화**: 플레이어와의 거리가 가깝지 않은 경우 갱신 주기를 1초 단위로 대폭 늘리는 거리 기반 휴리스틱 탐색 방식을 도입했습니다.
*   **결과**: 
    *   메인 스레드의 CPU 점유율이 약 **65% 감소**했습니다.
    *   다수 몬스터 동시 스폰 시에도 프레임 드랍 없이 안정적인 **60 FPS**를 유지했습니다.

---

## 📺 5. 플레이 데모 및 이미지 갤러리 (Demo & Gallery)

여기에 프로젝트를 직관적으로 보여줄 수 있는 영상이나 사진 자료를 첨부합니다.

### A. 플레이 영상 데모 (유튜브 임베딩 예시)
Minimal Mistakes 테마에서는 아래와 같은 Liquid 태그를 통해 유튜브 영상을 반응형으로 간단히 삽입할 수 있습니다. 
`(id 속성에 유튜브 비디오의 코드 주소를 입력하세요)`

{% include video id="qY7_V96g-jE" provider="youtube" %}
*설명: 위 비디오 컴포넌트는 모든 디스플레이 크기에 맞추어 반응형으로 자동 리사이즈됩니다.*

### B. GIF 인게임 연출 화면 (움직이는 이미지 예시)
인게임 핵심 로직이나 역동적인 씬(Scene)은 아래와 같이 GIF 파일을 경로에 추가하여 생동감 있게 보여줄 수 있습니다.

![움직이는 GIF 예시](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExM3VscDVidmF4aWdwdXQzZ3pxYTV1NGc4cTRpZmt5cTRhdnp6dHpiZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/kFgzrTt798d2w/giphy.gif)
*설명: 인게임 물리 연산 및 격자 그리드 시스템이 상호작용하는 모습을 담은 움짤(GIF) 예시입니다.*

### C. 스크린샷 갤러리 (바둑판 배열 배치)
Front Matter 영역에 이미지 리스트(`gallery:`)를 미리 등록해 두면, 아래의 태그 한 줄로 깔끔한 바둑판형 슬라이드 및 갤러리를 노출시킬 수 있습니다.

{% include gallery %}
*설명: 등록된 이미지 갤러리입니다. 각 이미지를 클릭하면 확대해서 볼 수 있습니다.*

---

## 🔗 6. 관련 링크 (Links)
*   💻 **GitHub Repository**: [GitHub 링크 바로가기](https://github.com/SweetFlush)
*   💾 **다운로드 빌드 파일**: [Itch.io 또는 구글 드라이브 링크]
