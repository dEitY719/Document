# Dash 관련 질문
## Q1. 
``` python
# 현재 구조
data_loader = DataLoader()

# layout 관리
app = Dash(__name__, external_stylesheets=["bootstrap"])

app.layout = ... # something

if __name__ == "__main__":
    app.run_server(debug=True, host=Config.HOST, port=Config.PORT)
	
# 질문, 
app.run_server 이후에 웹에서 F5	(리프레쉬)를 하면 DataLoader() 를 다시 수행할 것을 기대했으나,
다시 실행되지 않는다. 어떻게 구조를 잡아야 하나?

# 요청
1. OOP, SOLID 원칙을 만족하고, 파이써닉한 코드를 알려줘
2. 예제도 자세하게 설명해줘
```

## A1-1. chatGPT
## A1-2. perflexity
Dash 애플리케이션에서 페이지 리프레시 시 데이터를 재로드하려면 **애플리케이션 라이프사이클과 레이아웃 생성 방식을 재설계**해야 합니다. 다음은 OOP와 SOLID 원칙을 준수하는 파이써닉한 솔루션입니다:

---

### 1. 핵심 원리
- **문제 진단**: `DataLoader` 인스턴스가 앱 시작 시 1회만 초기화되므로 페이지 리프레시 시 재실행되지 않음
- **해결 전략**: 레이아웃을 함수로 구성하여 매 페이지 로드 시 데이터 재조회
- **SOLID 준수**: 단일 책임 원칙(SRP), 개방-폐쇄 원칙(OCP) 적용

---

### 2. 개선된 구조
```python
from dash import Dash, html
from data_loader import DataLoader  # 사용자 정의 모듈

class AppInitializer:
    """애플리케이션 초기화 담당 클래스"""
    def __init__(self):
        self.app = Dash(__name__, external_stylesheets=["bootstrap"])
        self._configure_layout()
        
    def _configure_layout(self):
        def serve_layout():
            # 매 레이아웃 요청 시 새 DataLoader 인스턴스 생성
            data_loader = DataLoader()  
            raw_data = data_loader.load()
            return html.Div([
                # 데이터를 사용한 레이아웃 구성
                html.H1("Dynamic Dashboard"),
                # ... 다른 컴포넌트들
            ])
            
        self.app.layout = serve_layout

def create_app():
    """팩토리 함수를 통한 애플리케이션 생성"""
    initializer = AppInitializer()
    return initializer.app

if __name__ == "__main__":
    app = create_app()
    app.run_server(debug=True, host=Config.HOST, port=Config.PORT)
```

---

### 3. 고급 최적화 버전 (의존성 주입)
```python
from dataclasses import dataclass
from dash import Dash, html

@dataclass
class DataLoader:
    """데이터 로드 책임을 가진 클래스"""
    source: str  # 의존성 주입을 위한 설정
    
    def load(self):
        # 실제 데이터 로드 로직
        return f"Data from {self.source}"

class LayoutBuilder:
    """동적 레이아웃 생성 전담 클래스"""
    def __init__(self, data_loader: DataLoader):
        self.data_loader = data_loader
        
    def build(self) -> html.Div:
        data = self.data_loader.load()
        return html.Div([
            html.H1("Advanced Dashboard"),
            html.P(data),
            # ... 데이터 기반 컴포넌트
        ])

class DashAppFactory:
    """애플리케이션 생성 팩토리"""
    @staticmethod
    def create(config: dict) -> Dash:
        app = Dash(__name__, **config)
        
        # 의존성 주입
        data_loader = DataLoader(source="AWS Athena")
        layout_builder = LayoutBuilder(data_loader)
        
        # 레이아웃 함수 바인딩
        app.layout = lambda: layout_builder.build()
        return app

if __name__ == "__main__":
    config = {
        "external_stylesheets": ["bootstrap"],
        "suppress_callback_exceptions": True
    }
    app = DashAppFactory.create(config)
    app.run_server(host=Config.HOST, port=Config.PORT)
```

---

### 4. 동작 방식
1. **페이지 리프레시 발생** → Dash 서버가 새로운 HTTP 요청 수신
2. `serve_layout()` 함수 재실행 → 새 `DataLoader` 인스턴스 생성
3. 최신 데이터로 레이아웃 다시 구성
4. 콜백이 데이터를 참조할 경우 `@callback` 함수 내에서 DataLoader 사용

---

### 5. 추가 최적화 기법
```python
# 데이터 캐싱 전략 (리소스 절약용)
from functools import lru_cache

class CachedDataLoader(DataLoader):
    @lru_cache(maxsize=1)  # 1회 캐싱
    def load(self):
        return super().load()

# 주기적 리프레시 (dcc.Interval 조합)
import dash_core_components as dcc
from datetime import datetime

def build_layout():
    return html.Div([
        dcc.Interval(id='refresh-trigger', interval=5*60*1000),  # 5분 간격
        html.Div(id='live-content')
    ])
```

---

### 6. 장점
- **확장성**: 데이터 소스 변경 시 `DataLoader` 구현체만 교체
- **테스트 용이성**: 모듈별 단위 테스트 가능
- **유지보수성**: 레이아웃/데이터/로직이 명확히 분리
- **리소스 관리**: 캐싱 전략으로 부하 감소

이 구조는 페이지 리프레시 시 항상 최신 데이터를 로드하며, 의존성 주입을 통해 다양한 데이터 소스에 유연하게 대응할 수 있습니다[1][6][7].

Citations:
[1] https://stackoverflow.com/questions/63225707/plotly-dash-refreshing-global-data-on-reload
[2] https://community.plotly.com/t/best-method-for-triggering-a-backend-update-data-refresh/38585
[3] https://community.dataiku.com/discussion/38604/update-dataset-with-dash
[4] https://community.plotly.com/t/introducing-dash-oop-components-write-modular-composable-configurable-dash-code/46481
[5] https://python.plainenglish.io/building-a-dockerised-structured-multipage-plotly-dash-app-3fb53c8703a0
[6] https://community.plotly.com/t/refresh-dataframe-on-page-load/55139
[7] https://stackoverflow.com/questions/71802750/dash-app-refresh-underlying-pandas-dataframe-hourly/71804518
[8] https://github.com/plotly/dash-recipes/blob/master/live-reload.py
[9] https://stackoverflow.com/questions/61359017/is-there-a-user-controlled-way-to-refresh-data-in-a-dash-app
[10] https://community.plotly.com/t/behavior-changes-after-update/72448
[11] https://github.com/plotly/dash/issues/1293
[12] https://www.faa.gov/air_traffic/publications/media/PCG_Bsc_dtd_2-20-25_POST.pdf
[13] https://community.plotly.com/t/structuring-a-large-dash-application-best-practices-to-follow/62739
[14] https://community.plotly.com/t/sharing-examples-of-partial-update-allow-duplicate-true-from-dash-2-9-2/74131
[15] https://www.youtube.com/watch?v=YU7bCEcsBK8
[16] https://stackoverflow.com/questions/66698348/python-parent-child-design-pattern
[17] https://dash.plotly.com/layout

---
Perplexity로부터의 답변: pplx.ai/share

웹 브라우저에서 F5(리프레시)를 실행할 경우, Dash 애플리케이션은 기본적으로 새 HTTP 요청을 처리하고 레이아웃을 다시 그립니다. 그러나 Dash의 레이아웃이 **함수 기반**으로 설정되어 있지 않다면, 초기화된 객체(`DataLoader`)는 다시 실행되지 않습니다. 위에서 제안한 구조는 F5 이벤트가 발생할 때 레이아웃을 다시 생성하며 `DataLoader`를 새로 호출하는 방식으로 동작합니다.

### 동작 확인
- Dash의 `app.layout`을 **함수로 정의**하면, 매번 브라우저에서 새로고침(F5) 시 레이아웃을 새로 생성합니다.
- 함수 내부에서 `DataLoader`를 호출하므로, 데이터 로드가 매번 새롭게 이루어집니다.

### 추가 검증
Dash 공식 문서에 따르면, 함수 기반 레이아웃은 매 요청마다 실행되므로 F5를 눌렀을 때 `DataLoader`가 재실행됩니다. 이를 통해 최신 데이터를 반영할 수 있습니다.

### 예제 동작 흐름
1. 브라우저에서 F5를 누르면 서버에 새로운 HTTP 요청이 전달됩니다.
2. Dash는 `serve_layout()` 함수를 호출하여 새로운 레이아웃을 생성합니다.
3. `DataLoader`가 다시 실행되며 데이터를 로드합니다.
4. 새로운 데이터로 구성된 레이아웃이 사용자에게 렌더링됩니다.

따라서 위 코드 구조는 F5 이벤트 발생 시 기대한 대로 동작합니다.

Citations:
[1] https://lts0606.tistory.com/482
[2] https://help.tableau.com/current/pro/desktop/ko-kr/shortcut.htm
[3] https://www.checkmal.com/manual/detail/?lang=ko
[4] https://help.tableau.com/current/online/ko-kr/to_connections_modify.htm

---
Perplexity로부터의 답변: pplx.ai/share


## A1-3. gemini

## Dash 앱에서 DataLoader 재실행 문제 해결 및 개선된 구조 제안

현재 Dash 앱 구조에서는 `app.run_server`가 실행된 후 웹 페이지에서 새로고침(F5)을 하더라도 `DataLoader()`가 다시 실행되지 않습니다. 이는 Dash 앱이 서버 모드로 실행되는 동안 초기화된 객체(`DataLoader` 인스턴스)가 메모리에 유지되기 때문입니다.

요청하신 대로 OOP, SOLID 원칙을 만족하고 파이써닉한 코드를 통해 이 문제를 해결하고 구조를 개선하는 방법을 설명드리겠습니다.

### 1. 문제 해결 및 개선된 구조

1.  **데이터 로딩 및 관리 분리:**
    * `DataLoader` 클래스를 데이터 로딩 및 관리 역할을 수행하는 별도의 클래스로 분리합니다.
    * 이 클래스는 데이터 로딩 로직을 캡슐화하고, 필요한 경우 데이터를 다시 로딩하는 기능을 제공합니다.
2.  **데이터 관리자 클래스 도입:**
    * 데이터 로딩 및 관리 로직을 총괄하는 `DataManager` 클래스를 도입합니다.
    * 이 클래스는 `DataLoader` 인스턴스를 관리하고, 필요에 따라 데이터를 다시 로딩하거나 업데이트하는 기능을 제공합니다.
3.  **콜백 함수 활용:**
    * Dash 콜백 함수를 사용하여 웹 페이지 새로고침 이벤트 또는 특정 버튼 클릭과 같은 사용자 인터랙션에 따라 `DataManager`를 통해 데이터를 다시 로딩합니다.
4.  **싱글톤 패턴 적용 (선택 사항):**
    * `DataManager`를 싱글톤 패턴으로 구현하여 앱 전체에서 하나의 인스턴스만 사용하도록 관리할 수 있습니다.

### 2. 코드 예시

```python
import dash
from dash import Dash, html, dcc, Input, Output
import time

class DataLoader:
    """데이터 로딩 및 관리 클래스"""
    def __init__(self):
        self.data = None
        self.load_data()

    def load_data(self):
        """데이터 로딩 로직 (예시: 시간 정보 로딩)"""
        print("데이터 로딩 시작...")
        time.sleep(1)  # 데이터 로딩 시뮬레이션
        self.data = f"현재 시간: {time.time()}"
        print("데이터 로딩 완료.")

    def get_data(self):
        """로드된 데이터 반환"""
        return self.data

class DataManager:
    """데이터 관리자 클래스"""
    def __init__(self):
        self.data_loader = DataLoader()

    def reload_data(self):
        """데이터 다시 로딩"""
        self.data_loader.load_data()

    def get_data(self):
        """로드된 데이터 반환"""
        return self.data_loader.get_data()

# Dash 앱 초기화
app = Dash(__name__)
data_manager = DataManager()

# 레이아웃 정의
app.layout = html.Div([
    html.Button("새로고침", id="refresh-button"),
    html.Div(id="data-output"),
])

# 콜백 함수 정의
@app.callback(
    Output("data-output", "children"),
    Input("refresh-button", "n_clicks"),
)
def update_data(n_clicks):
    """새로고침 버튼 클릭 시 데이터 업데이트"""
    if n_clicks:
        data_manager.reload_data()
    return data_manager.get_data()

if __name__ == "__main__":
    app.run_server(debug=True)
```

### 3. 코드 설명

* **`DataLoader` 클래스:**
    * 데이터 로딩 로직을 캡슐화합니다.
    * `load_data()` 메서드를 통해 데이터를 로딩하고, `get_data()` 메서드를 통해 로드된 데이터를 반환합니다.
* **`DataManager` 클래스:**
    * `DataLoader` 인스턴스를 관리하고 데이터 로딩 및 관리 로직을 총괄합니다.
    * `reload_data()` 메서드를 통해 `DataLoader`의 `load_data()` 메서드를 호출하여 데이터를 다시 로딩합니다.
    * `get_data()` 메서드를 통해 `DataLoader`의 `get_data()` 메서드를 호출하여 로드된 데이터를 반환합니다.
* **Dash 앱:**
    * `DataManager` 인스턴스를 생성하여 데이터 로딩 및 관리를 담당합니다.
    * 새로고침 버튼 클릭 시 `update_data()` 콜백 함수가 호출되어 `DataManager`를 통해 데이터를 다시 로딩하고 화면을 업데이트합니다.

### 4. SOLID 원칙 및 파이써닉 코드

* **단일 책임 원칙 (SRP):** `DataLoader`와 `DataManager`는 각각 데이터 로딩과 데이터 관리라는 단일 책임을 수행합니다.
* **개방-폐쇄 원칙 (OCP):** `DataLoader`를 상속하여 다양한 데이터 소스에 대한 로딩 로직을 쉽게 확장할 수 있습니다.
* **리스코프 치환 원칙 (LSP):** `DataLoader`를 상속하는 하위 클래스는 `DataLoader`를 사용하는 코드에서 문제없이 작동해야 합니다.
* **인터페이스 분리 원칙 (ISP):** 필요한 기능만 제공하는 작은 인터페이스를 사용하여 클라이언트가 불필요한 기능에 의존하지 않도록 합니다.
* **의존성 역전 원칙 (DIP):** `DataManager`는 `DataLoader`의 구체적인 구현에 의존하지 않고 추상화된 인터페이스에 의존합니다.
* **파이써닉 코드:** 클래스 및 메서드 이름은 명확하고 이해하기 쉽도록 작성되었으며, 필요에 따라 독스트링을 추가하여 코드의 가독성을 높였습니다.

이러한 구조를 통해 Dash 앱에서 데이터를 효율적으로 관리하고, 필요에 따라 데이터를 다시 로딩할 수 있습니다.


### 재차 확인 질문한 결과,

제시된 방식에서 웹 브라우저의 F5 새로 고침(refresh) 이벤트는 Dash 애플리케이션의 동작 방식과 약간 다른 방식으로 작동합니다. F5 새로 고침은 브라우저에서 서버로 새로운 요청을 보내고, 서버는 해당 요청에 대한 응답으로 전체 HTML 페이지를 다시 전송합니다. 하지만 Dash 애플리케이션은 이러한 전체 페이지 새로 고침 없이도 콜백 함수를 통해 특정 부분만 업데이트할 수 있도록 설계되어 있습니다.

다음은 F5 새로 고침과 Dash 애플리케이션의 동작 방식에 대한 자세한 설명입니다.

**웹 브라우저의 F5 새로 고침:**

* 브라우저는 서버에 새로운 HTTP 요청을 보냅니다.
* 서버는 전체 HTML 페이지를 다시 생성하여 브라우저에 전송합니다.
* 브라우저는 새로 받은 HTML 페이지를 기반으로 전체 페이지를 다시 렌더링합니다.
* 이 과정에서 JavaScript, CSS 등 모든 리소스가 다시 로드됩니다.

**Dash 애플리케이션의 동작 방식:**

* Dash는 React.js를 기반으로 하는 단일 페이지 애플리케이션(SPA)입니다.
* 초기 로드 시 전체 레이아웃과 필요한 JavaScript, CSS 리소스가 로드됩니다.
* 사용자 인터랙션(버튼 클릭, 입력 변경 등)이 발생하면 Dash는 콜백 함수를 통해 서버에 필요한 데이터만 요청합니다.
* 서버는 요청된 데이터를 JSON 형식으로 응답하고, Dash는 해당 데이터를 기반으로 페이지의 특정 부분만 업데이트합니다.
* F5 새로고침을 하게 되면 Dash앱의 레이아웃이 다시 그려지면서 처음부터 동작하게 됩니다.

**제시된 코드의 경우:**

* 제공된 코드에서 새로고침 버튼을 누르면 콜백 함수가 실행되고, `DataManager`를 통해 데이터를 다시 로드하여 화면을 업데이트합니다.
* 이 방식은 전체 페이지를 다시 로드하지 않고 필요한 데이터만 업데이트하는 효율적인 방식입니다.
* F5 새로 고침을 하게 되면 초기 레이아웃이 다시 그려지고, 초기 레이아웃을 구성할때 데이터 로더가 작동하게 됩니다.

**요약:**

* F5 새로 고침은 전체 페이지를 다시 로드하는 반면, Dash는 콜백 함수를 통해 필요한 부분만 업데이트합니다.
* 제시된 코드에서 새로고침 버튼을 사용하는 것은 Dash의 효율적인 업데이트 방식을 활용하는 것이며, F5 새로고침은 레이아웃전체를 다시 그리면서 데이터 로더를 다시 실행하게 됩니다.


## A4. gemini thinking 답변
## 웹 페이지 새로고침 시 DataLoader() 재실행 문제 해결 및 구조 개선

웹 애플리케이션 개발 시, 특히 Dash와 같은 프레임워크를 사용할 때, 웹 페이지 새로고침 시 데이터 로딩 로직이 다시 실행되기를 기대하는 것은 자연스러운 생각입니다. 하지만 현재 구조에서는 `app.run_server` 실행 후 웹 브라우저에서 F5 키를 눌러 새로고침을 해도 `DataLoader()`가 재실행되지 않는 현상이 발생합니다. 이는 Dash 애플리케이션의 실행 방식과 웹 브라우저의 동작 방식 때문에 발생하는 일반적인 현상입니다.

**문제 원인 분석**

Dash 애플리케이션은 `app.run_server()`가 호출될 때 웹 서버를 시작하고, 애플리케이션 레이아웃 및 초기 상태를 **딱 한 번** 설정합니다. `DataLoader()`는 현재 코드 구조상 애플리케이션이 시작될 때 **단 한 번만** 실행되어 데이터를 로드합니다.

웹 브라우저에서 F5 키를 눌러 새로고침을 하는 것은 **웹 서버에 새로운 HTTP 요청을 보내는 것**일 뿐, **파이썬 코드를 다시 처음부터 실행하는 것이 아닙니다**. 이미 실행 중인 Dash 애플리케이션은 새로운 요청에 대해 기존에 설정된 레이아웃과 상태를 기반으로 응답하며, `DataLoader()`를 다시 호출하지 않습니다.

**해결 방안: 데이터 로딩 시점 및 구조 변경**

웹 페이지 새로고침 시 데이터를 다시 로드하려면, 데이터 로딩 로직이 **웹 요청 처리 과정**에 포함되도록 구조를 변경해야 합니다. 즉, 사용자가 웹 페이지를 새로고침하거나 특정 액션을 취할 때마다 데이터를 다시 로드하는 방식을 구현해야 합니다.

**요청 사항에 따른 해결 방안 및 예제**

요청하신 사항 (1. OOP, SOLID 원칙 만족, 파이썬다운 코드, 2. 구체적인 예제 3가지 및 상세 설명) 에 맞춰 다양한 해결 방안과 예제를 제시하며, 각 예제가 어떻게 문제점을 해결하고 요청 사항을 만족하는지 자세하게 설명해 드리겠습니다.

### 예제 1: 콜백 함수와 `dcc.Interval` 컴포넌트 활용 (주기적 데이터 업데이트)

**개념:**

*   `dcc.Interval` 컴포넌트를 사용하여 주기적으로 콜백 함수를 실행합니다.
*   콜백 함수 내에서 `DataLoader()`를 호출하여 데이터를 다시 로드하고, 필요한 컴포넌트의 속성을 업데이트합니다.
*   **장점:** 주기적인 데이터 업데이트를 구현하여, 웹 페이지 새로고침 없이도 최신 데이터를 반영할 수 있습니다.
*   **단점:** 새로고침 시점에 정확히 맞춰 데이터를 로드하는 것은 아니며, 설정된 간격에 따라 데이터가 업데이트됩니다. 실시간성이 중요한 경우 간격 조절이 필요합니다.

**코드 예제:**

```python
import dash
from dash import Dash, html, dcc, Input, Output
import pandas as pd
import plotly.express as px

# Config 클래스 (예시)
class Config:
    HOST = '127.0.0.1'
    PORT = 8050

# DataLoader 클래스 (예시)
class DataLoader:
    def __init__(self):
        pass

    def load_data(self):
        # 실제 데이터 로딩 로직 (CSV, DB, API 등)
        data = pd.DataFrame({
            'x': range(10),
            'y': [i**2 for i in range(10)]
        })
        return data

data_loader = DataLoader() # 초기화 (하지만 콜백에서 재로드)

app = Dash(__name__, external_stylesheets=["https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"])

app.layout = html.Div(className="container", children=[
    html.H1("주기적 데이터 업데이트 예제", className="my-4"),
    dcc.Interval(  # 주기적으로 콜백 트리거
        id='interval-component',
        interval=5*1000, # 5초마다 (milliseconds 단위)
        n_intervals=0
    ),
    dcc.Graph(id='graph-output')
])

@app.callback(
    Output('graph-output', 'figure'),
    Input('interval-component', 'n_intervals') # interval 컴포넌트의 트리거 속성
)
def update_graph(n):
    loaded_data = data_loader.load_data() # 콜백 실행 시마다 데이터 재로드
    fig = px.line(loaded_data, x='x', y='y', title='주기적으로 업데이트되는 그래프')
    return fig

if __name__ == "__main__":
    app.run_server(debug=True, host=Config.HOST, port=Config.PORT)
```

**코드 설명:**

1.  `dcc.Interval` 컴포넌트가 레이아웃에 추가되었습니다. `interval` 속성을 통해 콜백이 트리거되는 주기를 설정할 수 있습니다 (예: `5*1000` milliseconds = 5초).
2.  `update_graph` 콜백 함수는 `dcc.Interval` 컴포넌트의 `n_intervals` 속성을 입력으로 받습니다. `n_intervals`는 `dcc.Interval`이 트리거된 횟수를 나타냅니다.
3.  콜백 함수 내에서 `data_loader.load_data()`를 **매번 호출**하여 데이터를 다시 로드합니다.
4.  로드된 데이터를 기반으로 그래프를 생성하고, `graph-output` 컴포넌트의 `figure` 속성을 업데이트합니다.

**SOLID 원칙 및 파이썬 스타일:**

*   **단일 책임 원칙 (SRP):** `DataLoader` 클래스는 데이터 로딩이라는 단일 책임, 콜백 함수는 데이터 로딩 및 그래프 업데이트라는 단일 책임을 가집니다.
*   **개방-폐쇄 원칙 (OCP):** `DataLoader` 클래스를 상속하거나 인터페이스를 구현하여 다양한 데이터 소스를 지원하도록 확장할 수 있습니다.
*   **파이썬다운 코드:** 클래스, 함수, 컴포넌트 활용, 명확한 변수명 등을 사용하여 가독성이 높은 파이썬 코드를 작성했습니다.

### 예제 2: 콜백 함수와 브라우저 Location 활용 (웹 페이지 로드 시 데이터 로딩)

**개념:**

*   `dcc.Location` 컴포넌트를 사용하여 브라우저의 주소 표시줄 정보를 감지합니다.
*   콜백 함수는 `dcc.Location`의 `pathname` 속성 (URL 경로) 변화를 감지하여 트리거됩니다.
*   웹 페이지가 처음 로드되거나 새로고침될 때 URL 경로가 변경되므로, 콜백 함수가 실행되어 데이터를 로드합니다.
*   **장점:** 웹 페이지가 로드될 때마다 데이터를 로드하여, 새로고침 시 데이터가 업데이트되는 효과를 얻을 수 있습니다.
*   **단점:** URL 경로 변화에 의존하므로, URL 경로가 변경되지 않는 단순 새로고침의 경우 콜백이 트리거되지 않을 수 있습니다 (브라우저 동작 방식에 따라 다름).  또한, URL 경로 변화가 데이터 로딩과 직접적인 연관이 없는 경우 부적절할 수 있습니다.

**코드 예제:**

```python
import dash
from dash import Dash, html, dcc, Input, Output
import pandas as pd
import plotly.express as px

# Config, DataLoader 클래스 (예제 1과 동일)
class Config:
    HOST = '127.0.0.1'
    PORT = 8050

class DataLoader:
    def __init__(self):
        pass

    def load_data(self):
        data = pd.DataFrame({
            'x': range(10),
            'y': [i**2 for i in range(10)]
        })
        return data

data_loader = DataLoader() # 초기화 (하지만 콜백에서 재로드)

app = Dash(__name__, external_stylesheets=["https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"])

app.layout = html.Div(className="container", children=[
    html.H1("웹 페이지 로드 시 데이터 로딩 예제", className="my-4"),
    dcc.Location(id='url', refresh=False), # 브라우저 URL 정보 감지
    dcc.Graph(id='graph-output')
])

@app.callback(
    Output('graph-output', 'figure'),
    Input('url', 'pathname') # URL 경로 변화 감지
)
def update_graph(pathname): # pathname은 필수로 사용하지 않아도 됨 (트리거 역할)
    loaded_data = data_loader.load_data() # 콜백 실행 시마다 데이터 재로드
    fig = px.line(loaded_data, x='x', y='y', title='웹 페이지 로드 시 데이터 로딩 그래프')
    return fig

if __name__ == "__main__":
    app.run_server(debug=True, host=Config.HOST, port=Config.PORT)
```

**코드 설명:**

1.  `dcc.Location` 컴포넌트가 레이아웃에 추가되었습니다. `refresh=False` 설정은 페이지 새로고침 시 전체 페이지를 다시 로드하지 않고 콜백만 업데이트하도록 설정합니다 (성능 향상).
2.  `update_graph` 콜백 함수는 `dcc.Location` 컴포넌트의 `pathname` 속성을 입력으로 받습니다. `pathname`은 URL 경로를 나타냅니다.
3.  콜백 함수는 `pathname`이 변경될 때마다 실행됩니다. 웹 페이지가 처음 로드될 때나 새로고침될 때 `pathname` 변화가 감지될 수 있습니다.
4.  콜백 함수 내에서 `data_loader.load_data()`를 **매번 호출**하여 데이터를 다시 로드합니다.

**SOLID 원칙 및 파이썬 스타일:**

*   예제 1과 마찬가지로 SOLID 원칙을 준수하며, 파이썬다운 코드를 유지합니다.
*   `dcc.Location` 컴포넌트를 활용하여 웹 브라우저의 동작 방식과 Dash 콜백 시스템을 연동하는 방법을 보여줍니다.

### 예제 3: 함수 기반 데이터 로더 및 레이아웃 함수 활용 (간결한 구조, 초기 로딩 시점 변경)

**개념:**

*   `DataLoader` 클래스 대신 간단한 **함수 기반 데이터 로더**를 사용합니다.
*   `app.layout`에 **함수**를 할당하여, 레이아웃이 **매번 호출될 때마다** 데이터 로딩 함수를 실행합니다.
*   **장점:** 코드가 간결해지고, 초기 데이터 로딩 시점을 레이아웃 생성 시점으로 옮길 수 있습니다. 간단한 애플리케이션에 적합합니다.
*   **단점:** `app.layout` 함수가 매번 호출될 때마다 전체 레이아웃을 다시 생성하므로, 애플리케이션이 복잡해질수록 성능 저하가 발생할 수 있습니다. 데이터 로딩 로직과 레이아웃 정의가 혼합되어 코드 구조가 복잡해질 수 있습니다 (SOLID 원칙 중 단일 책임 원칙 위반 가능성). 웹 페이지 새로고침 시점에 정확히 맞춰 데이터를 로드하는 것은 아니며, 레이아웃 생성 시점에 데이터가 로드됩니다.

**코드 예제:**

```python
import dash
from dash import Dash, html, dcc
import pandas as pd
import plotly.express as px

# Config 클래스 (예제 1과 동일)
class Config:
    HOST = '127.0.0.1'
    PORT = 8050

# 함수 기반 데이터 로더
def load_data():
    data = pd.DataFrame({
        'x': range(10),
        'y': [i**2 for i in range(10)]
    })
    return data

app = Dash(__name__, external_stylesheets=["https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"])

# 레이아웃 함수 (app.layout에 함수 할당)
def serve_layout():
    loaded_data = load_data() # 레이아웃 함수 호출 시마다 데이터 로드
    fig = px.line(loaded_data, x='x', y='y', title='레이아웃 함수 기반 데이터 로딩 그래프')
    layout = html.Div(className="container", children=[
        html.H1("레이아웃 함수 기반 데이터 로딩 예제", className="my-4"),
        dcc.Graph(id='graph-output', figure=fig) # figure를 레이아웃 생성 시점에 바로 전달
    ])
    return layout

app.layout = serve_layout # app.layout에 함수 할당

if __name__ == "__main__":
    app.run_server(debug=True, host=Config.HOST, port=Config.PORT)
```

**코드 설명:**

1.  `DataLoader` 클래스 대신 `load_data()` 함수를 사용하여 데이터 로딩 로직을 구현했습니다.
2.  `serve_layout()` 함수는 레이아웃을 정의하고 반환하는 함수입니다. 이 함수 내에서 `load_data()`를 호출하여 데이터를 로드하고, 그래프를 생성합니다.
3.  `app.layout = serve_layout` 와 같이 `app.layout`에 **함수**를 할당합니다. 이렇게 하면 Dash는 웹 요청이 들어올 때마다 `serve_layout()` 함수를 호출하여 레이아웃을 생성합니다.

**SOLID 원칙 및 파이썬 스타일:**

*   **간결성:** 클래스 대신 함수를 사용하여 코드가 간결해졌습니다.
*   **단일 책임 원칙 (SRP) 위반 가능성:** `serve_layout()` 함수는 레이아웃 생성과 데이터 로딩이라는 두 가지 책임을 동시에 가지게 되어 SRP를 위반할 가능성이 있습니다. 복잡한 애플리케이션에서는 클래스 기반 구조가 더 유지보수에 유리할 수 있습니다.
*   **파이썬다운 코드:** 함수 활용, 명확한 변수명 등을 사용하여 가독성이 높은 파이썬 코드를 유지합니다.

**예제 선택 및 추가 고려 사항**

*   **애플리케이션 복잡도:** 간단한 애플리케이션에는 예제 3과 같이 함수 기반 레이아웃 함수가 간결하고 효율적일 수 있습니다. 하지만 애플리케이션이 복잡해질수록 예제 1 또는 예제 2와 같이 콜백 함수와 컴포넌트를 활용하는 방식이 더 유지보수 및 확장에 유리합니다.
*   **데이터 업데이트 주기:** 실시간 데이터 업데이트가 필요한 경우 예제 1의 `dcc.Interval` 컴포넌트가 적합합니다. 웹 페이지 새로고침 시 데이터 로딩을 원하는 경우 예제 2의 `dcc.Location` 컴포넌트를 활용하거나, 예제 3과 같이 레이아웃 함수 기반 방식을 사용할 수 있습니다 (새로고침 시 레이아웃 함수가 다시 호출되므로).
*   **SOLID 원칙 준수:** SOLID 원칙을 최대한 준수하려면 예제 1 또는 예제 2와 같이 클래스 기반 데이터 로더와 명확하게 분리된 콜백 함수를 사용하는 것이 좋습니다.

**결론**

제시된 3가지 예제는 각각 다른 방식으로 웹 페이지 새로고침 시 데이터 로딩 문제를 해결하고, 요청하신 OOP, SOLID 원칙, 파이썬 스타일을 고려하여 작성되었습니다. 애플리케이션의 요구 사항, 복잡도, 데이터 업데이트 주기 등을 고려하여 적절한 구조를 선택하시면 됩니다.