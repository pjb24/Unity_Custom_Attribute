
# 🎯 Conditional Inspector System for Unity

## 📘 개요
이 시스템은 Unity Inspector에서 **조건부 필드 표시(Show/Hide)** 를 지원한다.  
`[ShowIfAny]`, `[HideIfAll]`, `[ShowIfFlagsAny]` 등 Attribute를 이용해  
enum, bool, int, float, string, enum flags 값에 따라 필드를 자동으로 표시하거나 숨길 수 있다.

---

## 🗂️ 폴더 구조 및 배치
```plaintext
Assets/
 ├─ Scripts/
 │   ├─ ConditionalAttribute.cs ← Attribute 정의 (런타임 코드)
 │   └─ Weapon.cs               ← 테스트용 스크립트 (예시)
 └─ Editor/
     ├─ ConditionalDrawer.cs    ← PropertyDrawer (에디터 전용)
     └─ CondPathUtil.cs         ← 경로 해석 유틸리티
````

* **Scripts 폴더**

  * `[ShowIfAny]`, `[HideIfAll]` 등 Attribute 클래스는 반드시 여기 위치해야 한다.
  * Editor 폴더에 넣으면 Inspector에서 인식되지 않음.

* **Editor 폴더**

  * `ConditionalDrawer`, `CondPathUtil`은 반드시 Editor 폴더에 배치해야 한다.
  * 그렇지 않으면 빌드 포함 오류가 발생한다.

---

## ⚙️ 주요 기능

* Enum, Bool, Int, Float, String, Enum Flags 비교 지원
* 점(`.`), 부모 상대(`../`), 절대(`$`) 경로 지원
* 다중 조건(AND / OR) 결합 가능
* `ShowIf` / `HideIf` / `ShowIfFlags` / `HideIfFlags` 시리즈 제공
* 멀티 오브젝트 편집 및 프리팹 오버라이드 호환

---

## 💡 사용 예시

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

* `_type == Melee` → `_meleeVar`만 표시
* `_type == Range` → `_rangeVar`만 표시

---

## 🔗 확장 기능

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

## ⚠️ 주의 사항

* **Attribute는 `Scripts/`**, **Drawer와 유틸리티는 `Editor/`** 폴더에 반드시 분리 배치해야 한다.
  경로가 섞이면 `ShowIfAny`를 인식하지 못한다.
* Drawer는 `SerializedProperty` 기반이므로 숨겨진 상태에서도 값은 유지된다.
* 문자열 비교는 기본적으로 **대소문자 구분(`Ordinal`)** 모드로 수행된다.
  필요 시 `OrdinalIgnoreCase`로 수정 가능.

---

## 🧩 참고 및 확장

* 다중 Attribute 부착 시 기본적으로 **AND 결합**
* Enum Flags는 비트 단위로 Any / All 비교 가능
* `[ShowIfAll]`, `[HideIfAny]`, `[ShowIfFlagsAll]` 등 조합 가능
* 문자열, float, enum index 비교까지 완전 지원

---

## 📄 구성 요약

| 구성 요소        | 파일명                       | 폴더      | 역할                      |
| ------------ | ------------------------- | ------- | ----------------------- |
| Attribute 정의 | `ConditionalAttribute.cs` | Scripts | 조건부 표시 Attribute 정의     |
| Drawer 구현    | `ConditionalDrawer.cs`    | Editor  | 조건 평가 및 Inspector 표시 제어 |
| 경로 유틸리티      | `CondPathUtil.cs`         | Editor  | 점/상대/절대 경로 해석 처리        |

---

## 🧰 복사 후 바로 사용하기

이 시스템은 프로젝트 간 재사용을 고려해 설계되었다.
`Assets/Scripts/` 와 `Assets/Editor/` 폴더 구조만 유지하면 즉시 동작한다.

```plaintext
Assets/
 ├─ Scripts/
 │   └─ ConditionalAttribute.cs
 └─ Editor/
     ├─ ConditionalDrawer.cs
     └─ CondPathUtil.cs
```

필요 시 원하는 필드에 다음과 같이 Attribute를 추가하면 된다:

```csharp
[ShowIfAny(nameof(isActive), true)]
[SerializeField] private int powerLevel;
```

---