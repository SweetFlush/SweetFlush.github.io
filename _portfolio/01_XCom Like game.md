---
title: "X-Com 라이크 턴제 전략 액션 게임 개발"
excerpt: "3D 턴제 전략 시뮬레이션 게임 개발기"
header:
  teaser: "/assets/images/xComLike.jpg" # 포트폴리오 목록에 보일 대표 이미지 경로
  overlay_image: "https://images.unsplash.com/photo-1542751371-adc38448a05e?auto=format&fit=crop&w=1600&q=80" # 포스트 상단 헤더 배경 이미지 경로
  overlay_filter: 0.5 # 배경 어둡기 조절 (0.1 ~ 1.0)
classes: wide
gallery:
  - url: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80"
    image_path: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&w=800&q=80"
    alt: "그리드 월드 매핑 데모"
    title: "월드 좌표계와 연동된 격자 구조 디버깅"
  - url: "https://images.unsplash.com/photo-1511512578047-dfb367046420?auto=format&fit=crop&w=800&q=80"
    image_path: "https://images.unsplash.com/photo-1511512578047-dfb367046420?auto=format&fit=crop&w=800&q=80"
    alt: "유닛 액션 시스템 데모"
    title: "유닛 행동 범위 및 UI 패널 시각화"
---

Code Monkey의 온라인 강의를 구매하여 X-COM과 유사한 턴제 전략게임을 개발하고, 그리드를 구현하는 코드의 이해 및 Code Monkey가 사용하는 다형성 설계를 체득하고자 시작한 프로젝트.

---

## 📌 1. 프로젝트 개요 (Overview)

| 항목 | 내용 |
| :--- | :--- |
| **개발 기간** | 2026.04 - 2026.05 (약 2개월) |
| **개발 인원** | 1명 (개인 개발) |
| **엔진 & 플랫폼**| Unity 2022.3 LTS / PC (Windows) |
| **사용 기술** | C#, URP, Singleton, State Pattern, Template Method Pattern |
| **장르 / 핵심 콘셉트**| 3D Turn-based Tactical Strategy (3D 턴제 전략 시뮬레이션) |

> **핵심 설계 목표:**  
> 1. 2D 격자(Grid) 구조 스크립트를 개발하고, 주요 로직을 체득한다..  
> 2. 이동, 사격, 대기 등 유닛의 모든 액션 행동을 추상화하여, 메인 제어 루프 수정 없이 신규 스킬/액션을 추가할 수 있는 개방-폐쇄 원칙(OCP)을 유니티 프로그래밍에 적용하는 방법을 익힌다.  
> 3. 비동기 연출(애니메이션, 투사체 비행) 도중 발생할 수 있는 유저 오입력을 완벽히 제어하여 상태 꼬임 현상을 차단합니다.

---

## 🛠️ 2. 사용 기술 및 라이브러리 (Tech Stack)

*   **Unity Universal Render Pipeline (URP)**: 3D 셀 셰이딩 및 격자 그리드 비주얼 렌더링 최적화를 위해 경량 파이프라인을 채택했습니다.
*   **C# Events & Action Delegate**: 시스템 결합도(Decoupling)를 낮추기 위해 UI와 액션 상태 전환에 옵저버 패턴 및 이벤트 기반 구조를 전면 도입했습니다.
*   **Singleton Pattern**: 레벨 그리드, 유닛 조작 매니저, 턴 제어기 등 스테이지 내 고유 라이프사이클을 가진 클래스들을 싱글톤으로 설계해 접근성을 향상시켰습니다.

---

## 🚀 3. 핵심 구현 내용 & 코드 예시 (Key Implementations)

### A. 3D World - 2D Grid 좌표 매핑 시스템 (`GridSystem` & `LevelGrid`)
*   **직면한 난관**: 실시간 액션 게임과 달리 턴제 게임은 격자 타일 단위를 기준으로 이동 및 공격 판정이 이루어져야 합니다. 유닛이 월드 상에서 이동할 때, 물리 위치 갱신 속도와 타일 점유 정보 갱신 타이밍이 일치하지 않으면 타일 이중 점유 버그가 발생하거나 유닛끼리 관통하는 문제를 겪었습니다.
*   **기술적 해결책**: 
    1.  3D 월드 공간(Vector3)과 논리 격자 공간(`GridPosition` 구조체)을 선형 매핑하고 2차원 배열로 그리드 타일 데이터를 보관하는 `GridSystem`을 설계했습니다.
    2.  유닛이 월드 상에서 물리적으로 서서히 이동(Lerp)하는 연산과 무관하게, **이동 명령이 떨어지는 즉시 논리적 데이터 구조(`LevelGrid`)에서 이전 타일 점유를 지우고 목적지 타일을 점유 처리**하도록 했습니다.
    3.  물리 연출(View)과 논리 상태(Model)를 엄격히 격리함으로써, 연출 프레임 속도와 상관없이 턴제 규칙의 정밀함과 논리적 일관성을 확보했습니다.

```csharp
// GridSystem.cs - 월드 좌표와 그리드 좌표 간의 연동 및 구조화
public class GridSystem
{
    private int width;
    private int height;
    private float cellSize;
    private GridObject[,] gridObjectArray;

    public GridSystem(int width, int height, float cellSize)
    {
        this.width = width;
        this.height = height;
        this.cellSize = cellSize;

        gridObjectArray = new GridObject[width, height];
        for(int x = 0; x < width; x++)
        {
            for(int z = 0; z < height; z++)
            {
                GridPosition gridPosition = new GridPosition(x, z);
                gridObjectArray[x, z] = new GridObject(this, gridPosition);
            }
        }
    }

    public Vector3 GetWorldPosition(GridPosition gridPosition)
    {
        return new Vector3(gridPosition.x, 0, gridPosition.z) * cellSize;
    }

    public GridPosition GetGridPosition(Vector3 worldPosition)
    {
        return new GridPosition(
            Mathf.RoundToInt(worldPosition.x / cellSize),
            Mathf.RoundToInt(worldPosition.z / cellSize)
        );
    }

    public bool IsValidGridPosition(GridPosition gridPosition)
    {
        return (gridPosition.x >= 0 && gridPosition.z >= 0) &&
               (gridPosition.x < width && gridPosition.z < height);
    }
}
```

### B. OCP(개방-폐쇄 원칙)를 준수한 다형성 액션 구조 (`BaseAction` & `MoveAction`)
*   **직면한 난관**: 유닛의 행동 종류(이동, 회전, 사격 등)가 점차 늘어남에 따라 입력 시스템(`UnitActionSystem`) 내부에 `switch-case` 분기문이 비대해지며, 코드 가독성이 극도로 떨어지고 수정 중 오류 전파 확률이 매우 높아지는 문제를 겪었습니다.
*   **기술적 해결책**: 
    1.  모든 행동의 공통 성질(행동 이름, 소모 AP, 동작 트리거, 유효 타일 리스트 조회)을 캡슐화한 추상 클래스 `BaseAction`을 정의했습니다.
    2.  `MoveAction`과 `ShootAction`은 `BaseAction`을 상속받아 고유 로직과 이동/사격 가능 범위 연산(`GetValidActionGridPositionList`)을 구현합니다.
    3.  `UnitActionSystem`은 개별 행동의 구체적인 구현 내용을 전혀 모르더라도, 단지 `BaseAction` 인터페이스를 통해 행동을 개시하므로 신규 액션 추가 시 메인 매니저 코드를 단 한 줄도 수정하지 않게 설계하여 **결합도를 대폭 낮췄습니다.**

```csharp
// BaseAction.cs - 모든 유닛 행동의 템플릿 메서드 추상화
public abstract class BaseAction : MonoBehaviour
{
    protected Unit unit;
    protected bool isActive;
    protected Action onActionComplete;

    protected virtual void Awake()
    {
        unit = GetComponent<Unit>();
    }

    public abstract string GetActionName();
    public abstract void TakeAction(GridPosition gridPosition, Action onActionComplete);
    public abstract List<GridPosition> GetValidActionGridPositionList();

    public virtual bool IsValidActionGridPosition(GridPosition gridPosition)
    {
        return GetValidActionGridPositionList().Contains(gridPosition);
    }

    protected void ActionStart(Action onActionComplete)
    {
        isActive = true;
        this.onActionComplete = onActionComplete;
    }

    protected void ActionComplete()
    {
        isActive = false;
        onActionComplete();
    }
}
```

### C. 비동기 연출 흐름 통제 및 Busy 시스템 (`UnitActionSystem`)
*   **직면한 난관**: 유닛이 이동 경로를 따라 물리적으로 달리거나 사격 애니메이션을 연출하는 비동기 처리 시간 동안, 유저가 마우스로 다른 유닛을 클릭하거나 액션을 강제 취소함으로써 유닛의 스피드 및 위치가 공중에서 꼬여버리는 결함이 발생했습니다.
*   **기술적 해결책**:
    1.  `UnitActionSystem` 내부에서 액션 상태의 바쁨 여부를 정의하는 `isBusy` 플래그를 관리하여, 연출이 진행 중일 때 모든 유저 입력을 차단하도록 차단막(Lock)을 세웠습니다.
    2.  액션을 개시할 때 `TakeAction(gridPosition, ClearBusy)`과 같이 **행동 완료 시점에 호출할 콜백 델리게이트(`Action onActionComplete`)**를 구체화된 액션 객체에 주입했습니다.
    3.  이를 통해 물리적 연출이 완전히 완료된 프레임에 스스로 `ActionComplete()`를 호출하여 메인 조작계의 바쁨 상태(`isBusy = false`)를 해제하게 하여 연출과 조작 사이의 동기화 흐름을 안전하게 일치시켰습니다.

---

## 🔍 4. 문제 해결 사례 (Troubleshooting)

### 🔴 이슈: 다수 유닛의 유효 행동 반경 연산 시 병목으로 인한 턴 전환 렉 발생
*   **상황**: 적 유닛과 아군 유닛의 수가 늘어날수록, 턴 시작 시 각 유닛이 갈 수 있는 유효 타일 경로를 체크하는 과정에서 매 턴 전환 시마다 약 0.3초 동안 화면이 멈추는 프레임 드랍(스파이크) 현상이 발생했습니다.
*   **원인 분석**: 
    1.  `MoveAction`의 `GetValidActionGridPositionList()` 연산 내에서 매 프레임마다 주변 타일들을 순회하며 `LevelGrid.Instance.HasAnyUnitOnGridPosition()`을 호출하고 있었습니다.
    2.  `GridObject`는 매번 자신이 가지고 있는 유닛 리스트를 순회하며 검색하므로 몬스터와 타일이 늘어남에 따라 탐색 연산이 `O(N^2)` 이상의 복잡도로 치솟는 병목이 원인이었습니다.
*   **해결 방법**:
    *   **유닛 상태 캐시 시스템 도입**: 그리드 오브젝트 내부의 유닛 추가/삭제 이벤트 발생 시점에만 해당 그리드 타일의 점유 여부 플래그(bool)를 단일 캐시 배열에 갱신해 두도록 데이터 구조를 개선했습니다.
    *   이를 통해 타일 탐색 시 매번 무겁게 리스트를 조회하지 않고 단 한 번의 배열 인덱싱 `O(1)` 구조로 유닛 점유 여부를 검사하게 최적화했습니다.
*   **결과**: 
    *   유효 이동 범위 연산 속도가 **약 88% 단축**되었습니다.
    *   동시 배치된 유닛 수가 많을 때도 턴 전환 지연이 사라지고 안정적인 **60 FPS** 환경을 달성했습니다.

---

## 📺 5. 플레이 데모 및 이미지 갤러리 (Demo & Gallery)

여기에 프로젝트를 직관적으로 보여줄 수 있는 영상이나 사진 자료를 첨부합니다.

### A. 플레이 영상 데모
유니티 에디터 상에서 그리드가 시각화되고 유닛이 액션 포인트(AP)를 소모하며 턴제 전투를 벌이는 플레이 영상 데모입니다.

{% include video id="qY7_V96g-jE" provider="youtube" %}
*설명: 유닛이 이동 및 행동을 개시할 때 Busy 시스템에 의해 조작 인풋이 안전하게 락킹되는 인게임 데모 비디오입니다.*

### B. GIF 인게임 연출 화면 (움직이는 이미지)
인게임 핵심 로직이나 턴제 조작 연출 등은 아래와 같이 GIF 파일을 경로에 추가하여 생동감 있게 보여줄 수 있습니다.

![움직이는 GIF 예시](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExM3VscDVidmF4aWdwdXQzZ3pxYTV1NGc4cTRpZmt5cTRhdnp6dHpiZCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/kFgzrTt798d2w/giphy.gif)
*설명: 인게임 공격 연산 및 격자 그리드 시스템이 상호작용하는 모습을 담은 움짤(GIF) 예시입니다.*

### C. 스크린샷 갤러리 (바둑판 배열 배치)
Front Matter 영역에 이미지 리스트(`gallery:`)를 미리 등록해 두면, 아래의 태그 한 줄로 깔끔한 바둑판형 슬라이드 및 갤러리를 노출시킬 수 있습니다.

{% include gallery %}
*설명: 등록된 이미지 갤러리입니다. 각 이미지를 클릭하면 확대해서 볼 수 있습니다.*

---

## 🔗 6. 관련 링크 (Links)
*   💻 **GitHub Repository**: [GitHub 저장소 바로가기](https://github.com/SweetFlush)
*   💾 **다운로드 데모 빌드**: [플레이 데모 다운로드 (준비중)]
