# 🎯 Conditional & Snap Attribute System for Unity

- [🎯 Conditional \& Snap Attribute System for Unity](#-conditional--snap-attribute-system-for-unity)
  - [📘 개요](#-개요)
  - [🗂️ 폴더 구조](#️-폴더-구조)
  - [⚙️ 기능 요약](#️-기능-요약)
  - [🧩 Conditional Inspector System](#-conditional-inspector-system)
    - [✅ 주요 기능](#-주요-기능)
    - [💡 사용 예시](#-사용-예시)
    - [✅ 부모 상대 경로](#-부모-상대-경로)
    - [✅ 점 경로](#-점-경로)
    - [✅ Enum Flags](#-enum-flags)
  - [🧩 SnapTo Attribute System](#-snapto-attribute-system)
    - [✅ 주요 기능](#-주요-기능-1)
    - [💡 사용 예시](#-사용-예시-1)
    - [입력 결과 예시](#입력-결과-예시)
  - [✅ 지원 정보](#-지원-정보)


## 📘 개요
이 리포지토리는 Unity Inspector를 확장하기 위한 **2가지 커스텀 Attribute 시스템**을 포함한다.

1. **Conditional Inspector System** — 필드를 조건에 따라 표시하거나 숨길 수 있는 시스템  
2. **SnapTo System** — 입력값을 지정된 간격(예: 90 단위)으로 자동 스냅(snap)시키는 시스템  

두 기능 모두 **프로젝트 간 독립적으로 사용 가능**하며, Editor/Runtime 구조만 지키면 바로 동작한다.


---
## 🗂️ 폴더 구조
```plaintext
Assets/
 ├─ Scripts/
 │   ├─ ConditionalAttribute.cs  ← 조건부 표시용 Attribute 정의
 │   └─ SnapToAttribute.cs       ← 값 스냅용 Attribute 정의
 └─ Editor/
     ├─ ConditionalDrawer.cs     ← 조건부 표시 Drawer
     ├─ CondPathUtil.cs          ← 조건부 표시용 경로 유틸리티
     └─ SnapToDrawer.cs          ← 값 스냅 Drawer
```


---
## ⚙️ 기능 요약
| 기능                       | 설명                                                        | 적용 대상              | 주요 Attribute                                     |
| ------------------------ | --------------------------------------------------------- | ------------------ | ------------------------------------------------ |
| **조건부 표시 (Conditional)** | bool, int, float, string, enum 값에 따라 Inspector에서 필드 표시/숨김 | SerializedField 전반 | `[ShowIfAny]`, `[HideIfAll]`, `[ShowIfFlagsAny]` |
| **값 스냅 (SnapTo)**        | Inspector 값 입력 시 지정 간격(예: 5, 90 등)에 맞게 자동 보정              | float, int 필드      | `[SnapTo(90f)]`                                  |


---
## 🧩 Conditional Inspector System

### ✅ 주요 기능
- Enum / Bool / Int / Float / String / Enum Flags 비교 지원
- 점(.), 부모 상대(../), 절대($) 경로 지원
- 다중 조건 AND / OR 결합 가능
- ShowIf, HideIf, ShowIfFlags, HideIfFlags 시리즈 제공

### 💡 사용 예시
```csharp
public class Weapon : MonoBehaviour
{
    public enum Type { Melee, Range }

    [SerializeField] private Type _type;

    [ShowIfAny(nameof(_type), Type.Melee)]
    [SerializeField] private int _meleeVar;

    [ShowIfAny(nameof(_type), Type.Range)]
    [SerializeField] private int _rangeVar;
}
```
- _type == Melee → _meleeVar만 표시
- _type == Range → _rangeVar만 표시

### ✅ 부모 상대 경로

```csharp
[System.Serializable]
public class Item
{
    public int kind;
    [ShowIfAny("../kind", 1)]
    public int data; // kind가 1일 때만 표시
}

public class Inventory : MonoBehaviour
{
    public Item[] items;
}
```

### ✅ 점 경로

```csharp
[System.Serializable]
public class Config
{
    public bool isActive;
    public int power;
}

public class Weapon : MonoBehaviour
{
    public Config config;

    [ShowIfAny("config.isActive", true)]
    [SerializeField] private int bonusPower;
}
```

### ✅ Enum Flags

```csharp
[Flags]
public enum Tags { None=0, Fire=1<<0, Ice=1<<1 }

[ShowIfFlagsAny(nameof(tags), (int)Tags.Fire, (int)Tags.Ice)]
public float elementalPower;
```


---
## 🧩 SnapTo Attribute System

### ✅ 주요 기능
- 입력값을 지정된 단위(예: 90f, 5f 등)로 자동 정렬
- float, int 타입 모두 지원
- 회전, 스케일, 위치, 타일 단위 등 다양한 수치에 사용 가능

### 💡 사용 예시
```csharp
public class SnapExample : MonoBehaviour
{
    [SnapTo(90f)] public float rotationY;
    [SnapTo(5f)]  public int gridSize;
}
```

### 입력 결과 예시
- rotationY: 47 → 0, 136 → 90, 271 → 270
- gridSize: 3 → 5, 12 → 10, 28 → 30


---
## ✅ 지원 정보
- 동작 대상:
    - Conditional: 모든 Serialized 타입
    - SnapTo: float, int