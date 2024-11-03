PyMCDA를 처음 공부하는 한국인 학생을 위한 튜토리얼: 자동차 선택 예제

### 기본 예제: 자동차 선택하기

먼저 노트북을 설정하여 그래프를 그릴 수 있도록 하고(첫 번째 줄), 자동 완성 기능도 활성화합니다.

```python
%matplotlib inline
%config Completer.use_jedi = False
```

### 문제 정의
모든 문제 해결의 시작은 문제를 명확히 하는 것입니다. 다기준 의사결정 분석(MCDA)에서는 여러 대안 중 하나를 선택하는 것이 문제일 수 있습니다.

```python
alternatives = ["a01", "a02", "a03", "a04", "a05"]
criteria = ["c01", "c02", "c03"]
```

### 성과표 정의

성과표는 2차원 행렬로, 첫 번째 차원은 대안을, 두 번째 차원은 기준을 나타냅니다. 각 셀에는 특정 기준에 대한 대안의 성과 값을 나타내는 값이 들어갑니다.

성과표를 정의하려면 다음과 같이 데이터프레임을 사용합니다:

```python
from mcda import PerformanceTable

perfTable = PerformanceTable(
    [
        [0, "medium", "+"]
        [0.5, "good", "++"],
        [1, "bad", "-"],
        [0.2, "medium", "0"],
        [0.9, "medium", "+"]
    ],
    alternatives=alternatives,
    criteria=criteria
)
```

대안 및 기준 값을 다음과 같이 접근할 수 있습니다:

```python
perfTable.alternatives_values["a01"].data
perfTable.criteria_values["c02"].data
```

`PerformanceTable`을 반복하여 각 대안과 기준 값을 확인할 수 있습니다:

```python
for alternative_values in perfTable.alternatives_values.values():
    print(alternative_values.data)

for a, values in perfTable.alternatives_values.items():
    print(f"{a}: {list(values)}")

for alternative_values in perfTable.alternatives_values.values():
    print(f"{alternative_values.name}: {list(alternative_values.data)}")

for criterion_values in perfTable.criteria_values.values():
    print(criterion_values.data)
```

성과표의 내부 표현을 `pandas.DataFrame`으로 접근할 수도 있습니다:

```python
perfTable.data
perfTable.data.to_dict()
```

### 기준 척도 정의

특정 기준에 대해 성과가 무엇을 의미하는지 더 잘 이해하려면 기준 척도를 정의할 수 있습니다. 기준 척도는 3가지 유형으로 나눌 수 있습니다:

#### 양적 척도 (Quantitative Scales)

양적 척도는 사용 가능한 모든 값을 포함하는 수치적 범위를 나타냅니다. 또한, `preference direction`을 사용하여 선호되는 값을 명시할 수 있습니다: 작은 값이 선호될 경우 `PreferenceDirection.MIN`, 큰 값이 선호될 경우 `PreferenceDirection.MAX`를 사용합니다.

```python
from mcda.scales import *

scale1 = QuantitativeScale(0, 1, preference_direction=MIN)
```

#### 질적 척도 (Qualitative Scales)

질적 척도는 가능한 라벨의 이산적 집합을 나타내며, 선호 순서를 설정하기 위해 해당 라벨에 대응하는 수치 값을 설정합니다.

```python
scale2 = QualitativeScale({"bad": 1, "medium": 2, "good": 3})
```

#### 명명 척도 (Nominative Scales)

명명 척도는 순서가 없는 가능한 라벨의 이산적 집합을 나타냅니다. 다기준 의사결정 분석에서는 명명 척도를 사용하는 것이 권장되지 않으며, 가능한 한 빨리 질적 척도로 대체해야 합니다.

```python
scale3 = NominalScale(["--", "-", "0", "+", "++"])
```

#### 기준 척도 묶기

여러 기준 척도를 사전 형태로 묶어 MCDA 문제의 각 기준에 하나씩 할당할 수 있습니다:

```python
scales = {
    criteria[0]: scale1,
    criteria[1]: scale2,
    criteria[2]: scale3
}
```

성과표를 생성할 때 기준 척도를 설정하지 않았다면 모듈이 이를 추론하려고 시도합니다. 그러나 양적 척도의 선호 방향이나 문자열 라벨에서 질적 값을 추론하지는 못합니다.

```python
perfTable.scales = scales
```

### 성과표 결합하기

두 개의 성과표를 결합하여 새로운 대안 값을 추가하거나 새로운 기준 값을 추가할 수 있습니다. 예를 들어 새로운 대안 값을 추가하려면 다음과 같이 합니다:

```python
perfTable2 = PerformanceTable(
    [
        [0.25, "medium", "++"],
        [0.75, "bad", "+"]
    ],
    alternatives=["b1", "b2"],
    criteria=criteria,
    scales=scales
)
PerformanceTable.concat([perfTable, perfTable2]).data
```

새로운 기준 값을 추가하려면 다음과 같이 합니다:

```python
new_scales = {
    "c04": QuantitativeScale(0, 10),
    "c05": QuantitativeScale(-100, 100, preference_direction=PreferenceDirection.MIN)
}

perfTable3 = PerformanceTable(
    [
        [0, 0],
        [2, -20],
        [5, -100],
        [10, 50],
        [3, -25]
    ],
    alternatives=alternatives,
    criteria=list(new_scales.keys()),
    scales=new_scales
)
PerformanceTable.concat([perfTable, perfTable3], axis=1).data
```

### 성과표 계산

#### 성과 값 확인하기

성과표의 모든 성과가 해당 기준 척도 내에 있는지 확인할 수 있습니다:

```python
perfTable.is_within_scales
```

#### 레이블을 수치 값으로 변환하기

성과표에 포함된 모든 값을 수치 값으로 변환할 수 있습니다. 이 방법은 새로운 `PerformanceTable` 객체를 반환합니다:

```python
scale3b = QualitativeScale({"--": 0, "-": 1, "0": 2, "+": 3, "++": 4})
scales = {
    criteria[0]: scale1,
    criteria[1]: scale2,
    criteria[2]: scale3b
}
perfTable = PerformanceTable(perfTable.data, scales)
numeric_table = perfTable.to_numeric
numeric_table.data
```

#### 수치 값 정규화하기

성과표의 수치 값을 정규화할 수 있습니다. 이 방법은 새로운 `PerformanceTable` 객체를 반환합니다:

```python
from mcda import normalize

normalized_table = normalize(perfTable)
normalized_table.data
```
아래는 주어진 내용에 대한 더 상세하고 친절한 설명입니다.

---

### 1. **데이터 원본으로 정규화하기 (Normalization on Raw Data)**

기준 척도를 제공하지 않고도 성능 테이블을 정규화할 수 있습니다. 이 경우, 테이블 자체에서 각 기준의 최소값과 최대값을 자동으로 가져와 사용하게 됩니다. 결과는 새로운 `PerformanceTable` 객체로 반환됩니다.

**예제 코드:**

```python
from mcda import PerformanceTable, normalize

# 성능 테이블 정의
table = PerformanceTable(
    [[0, 25000, -5], [1, 18000, -3], [0, 68000, -1]]
)

# 정규화 수행
normalized_table = normalize(table)

# 정규화된 데이터 확인
normalized_table.data
# 출력 예시:
#     0      1      2
# 0  0.0   0.14   0.0
# 1  1.0   0.00   0.5
# 2  0.0   1.00   1.0
```

이 코드에서 `normalize` 함수는 각 열의 값(즉, 각 기준)을 해당 열의 최소값과 최대값 사이의 비율로 변환합니다. 따라서 모든 값은 `0`과 `1` 사이로 변환되며, 각각의 기준에 대해 `0`은 최소값, `1`은 최대값을 나타냅니다.

---

### 2. **기준 함수 적용하기 (Apply Criteria Functions to Performances)**

성능 테이블의 각 기준 값에 대해 함수를 적용할 수 있습니다. 기준 함수는 **람다 함수**를 사용해 정의할 수 있으며, 이를 통해 기본 산술 연산부터 복잡한 함수까지 다양한 형태의 함수 적용이 가능합니다.

예를 들어, `lambda x: 2 * x - 0.5`는 각 값 `x`를 두 배로 만들고 0.5를 빼는 함수입니다.

#### 예제 코드

```python
f2 = lambda x: 2 * x - 0.5
```

#### 더 복잡한 함수 정의하기

복잡한 연산이 필요할 경우 `pymcda.functions` 모듈에서 다양한 함수 클래스를 사용할 수 있습니다.

```python
from mcda.functions import DiscreteFunction, PieceWiseFunction, Interval

# 불연속 함수 정의
f3 = DiscreteFunction({"--": 1, "-": 2, "0": 3, "+": 4, "++": 5})

# 함수 호출
f3("+")  # 출력: 4
```

#### 구간별 함수 정의하기

구간별로 나뉜 함수(구간별 함수)를 정의할 수도 있습니다. 예를 들어, 특정 범위에서는 단순히 `x`의 값을 그대로 반환하고, 다른 범위에서는 다른 연산을 적용하는 식입니다.

```python
f = PieceWiseFunction(
    {
        Interval(0, 2.5, max_in=False): lambda x: x,
        Interval(2.5, 5): lambda x: -0.5 * x + 2.0,
    }
)
```

이 함수는 `x`가 `0`과 `2.5` 사이일 때는 그대로 반환하고, `2.5`에서 `5` 사이일 때는 `-0.5 * x + 2.0`을 반환합니다.

#### 구간별 선형 함수 정의

구간별 선형 함수는 다양한 구간을 정의하여 각 구간에 대한 선형 함수를 적용할 수 있습니다.

```python
f1 = PieceWiseFunction(segments=[
    [[0, 1], [0.3, -2]],
    [[0.3, -2], [0.6, 0.5]],
    [[0.6, 0.5], [1, 5]]
])
```

#### 함수 호출 예제

정의한 구간별 선형 함수 `f1`을 `f1(0.8)`처럼 호출할 수 있습니다. 이는 `0.8`이 속한 구간에 맞는 함수 값을 반환합니다.

```python
f1(0.8)  # 출력: 2.75
```

이러한 함수는 척도와 유사하게 특정 객체에 묶어서 사용할 수 있습니다.

---

### 3. **기준 함수 묶기**

여러 기준 함수를 함께 묶어 하나의 `CriteriaFunctions` 객체로 만들 수 있습니다. 이렇게 하면 성능 테이블에 대해 여러 기준 함수를 한꺼번에 적용할 수 있습니다.

#### 예제 코드

```python
from mcda import transform
from mcda.functions import CriteriaFunctions

# 정의한 함수들을 기준별로 묶기
functions = CriteriaFunctions(
    {
        criteria[0]: f1,  # 구간별 함수
        criteria[1]: f2,  # 람다 함수
        criteria[2]: f3   # 불연속 함수
    },
    in_scales={
        criteria[0]: scales[criteria[0]],
        criteria[1]: scales[criteria[1]].numeric,
        criteria[2]: scales[criteria[2]]
    }
)

# 함수 적용 전 데이터 변환
input_table = transform(perfTable, functions.in_scales)

# 함수 적용 후 테이블 결과
nTable = functions(input_table)
nTable.data
# 출력 예시:
#        c01     c02     c03
# a01   1.000000  3.5     4
# a02  -0.333333  5.5     5
# a03   5.000000  1.5     2
# a04  -1.000000  3.5     3
# a05   3.875000  3.5     4
```

이렇게 하면 `input_table`의 각 기준에 대해 정의한 함수가 적용된 새로운 성능 테이블이 생성됩니다.

---

### 4. **성능 테이블의 값이 숫자인지 확인하기 (Check Performance Table Numerics)**

성능 테이블의 모든 값이 숫자인지 확인할 수 있습니다. 숫자형 값만 포함되어 있는지 확인하려면 `is_numeric` 속성을 사용합니다.

```python
nTable.is_numeric  # 출력 예시: True
```

### 5. **성능 값 합산하기 (Sum Performances)**

성능 테이블의 값들을 합산할 수 있으며, `axis` 파라미터를 사용하여 테이블 전체, 열별, 행별로 합산할 수 있습니다.

- **전체 합산**

```python
nTable.sum()
# 출력 예시: 44.04166666666667
```

- **열별 합산** (대안별로 각 기준의 합계)

```python
nTable.sum(axis=0).data
# 출력 예시:
# c01     8.541667
# c02    17.500000
# c03    18.000000
```

- **행별 합산** (기준별로 각 대안의 합계)

```python
nTable.sum(axis=1).data
# 출력 예시:
# a01     8.500000
# a02    10.166667
# a03     8.500000
# a04     5.500000
# a05    11.375000
```

이 튜토리얼을 통해 PyMCDA에서의 데이터 정규화, 함수 적용, 합산에 대한 이해가 깊어지기를 바랍니다. 각 함수와 속성 사용법을 하나하나 직접 실행해보면서 학습하면 더욱 효과적일 것입니다.



이 구문은 **구간별 선형 함수**를 정의하고, 이 함수에서 특정 입력 값에 따른 출력을 구하는 예시입니다. 여기서 `PieceWiseFunction`은 입력 범위에 따라 다른 선형 방정식을 적용하는 구조입니다.

함수를 정의한 코드를 천천히 살펴보면, 각 구간이 특정 범위에서 작동하도록 설정되어 있습니다.

### 구간별 정의

```python
f1 = PieceWiseFunction(segments=[
    [[0, 1], [0.3, -2]],  # 구간 1
    [[0.3, -2], [0.6, 0.5]],  # 구간 2
    [[0.6, 0.5], [1, 5]]  # 구간 3
])
```

여기서 `segments` 목록의 각 항목은 **두 개의 점**을 통해 구간을 정의합니다. 각 구간은 다음과 같이 해석됩니다:

1. **구간 1**: `[0, 1]`에서 `[0.3, -2]`까지
2. **구간 2**: `[0.3, -2]`에서 `[0.6, 0.5]`까지
3. **구간 3**: `[0.6, 0.5]`에서 `[1, 5]`까지

각 구간마다 두 점을 기준으로 **선형 방정식**이 만들어집니다. 이를 통해 `0.8`이 속한 구간의 방정식을 사용해 출력을 계산합니다.

### `0.8`이 속하는 구간 찾기

`0.8`은 **구간 3** `[0.6, 0.5]`에서 `[1, 5]`에 속합니다. 따라서 구간 3의 방정식을 사용하여 계산합니다.

### 구간 3의 선형 방정식 구하기

구간 3에 대해, 두 점 `[0.6, 0.5]`와 `[1, 5]`를 통해 선형 방정식을 구합니다. 두 점을 `(x1, y1) = (0.6, 0.5)`와 `(x2, y2) = (1, 5)`라고 하면, 기울기(슬로프) `m`과 절편 `b`를 구할 수 있습니다:

\[
m = \frac{y2 - y1}{x2 - x1} = \frac{5 - 0.5}{1 - 0.6} = \frac{4.5}{0.4} = 11.25
\]

\[
b = y1 - m \times x1 = 0.5 - (11.25 \times 0.6) = 0.5 - 6.75 = -6.25
\]

따라서, 구간 3의 선형 방정식은 다음과 같습니다:

\[
y = 11.25 \times x - 6.25
\]

### `0.8`을 대입하여 계산하기

이제 이 방정식에 `x = 0.8`을 대입하면:

\[
y = 11.25 \times 0.8 - 6.25 = 9 - 6.25 = 2.75
\]

따라서, `f1(0.8)`의 결과는 `2.75`가 됩니다.
