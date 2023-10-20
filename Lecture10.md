## 10강. 백테스팅 프로세스

### 소개

과거 데이터를 기반으로 투자모델의 유용성을 파악하는 것을 백테스팅이라고 합니다. 백테스팅은 퀀트 기반의 주식투자 모델을 만드는 첫번째 과정으로 새로운 투자 아이디어가 얼마나 유용했는지를 파악할 때 사용합니다. 백테스팅의 결과는 전략이 과거에 얼마나 유용하게 잘 작동했고 앞으로 잘 작동할만 할것 같은지를 알려주게 됩니다.  본 수업에서는 백테스팅을 하는 데 필요한 과정을 살펴보겠습니다.

1) 데이터와 소프트웨어
2) 전략 테스트 기간
3) 투자유니버스와 벤치마크
4) 생존편향 관리
5) 미래참조 편향 관리
6) 백테스팅 기간 설정
7) 데이터 마이닝 관리
8) 거래비용과 회전율 
9) 성과측정

### 데이터와 소프트웨어

**대표 데이터 제공 업체**

백테스트에 필요한 데이터 선택은 어떤 팩터를 사용할지에 따라 결정됩니다. 펀더멘털 데이터, 가격 및 수익률 데이터, 애널리스트 예측 데이터, 거시 경제 지표등 5가지의 주요 범주가 있고 추가로 대안 데이터 영역까지 포함하면 6가지의 범주 데이터가 있습니다. 펀더멘털 데이터를 제공해주는 업체는 Refinitiv, CRSP, S&P Compustats, FactSet 등이 유명합니다. 이들이 제공해주는 데이터에는 기업의 재무상태표, 손익계산서, 현금흐름표 항목이 있고 자주 사용하는 재무비율등을 따로 계산해서 제공해주기도 합니다. (두물머리는 Refinitiv를 사용하고 있습니다.) 가격 및 수익률 데이터는 CRSP가 가장 긴 역사적 데이터를 제공해주고 있습니다만 커버리지가 거의 미국 한정인게 아쉽습니다. 가격관련 데이터에는 단순히 가격뿐만 아니라 배당 및 기업의 합병, 분할등의 이벤트 데이터도 포함되어 있습니다. 그래야 수정주가를 계산할 수 있기 때문입니다.애널리스트 정보는 IBES가 가장 유명하고, Social 데이터와 경제지표등은 제공해주는 회사와 출처가 매우 다양합니다. 기관이라면 이런 단순 시계열 데이터는 블룸버그 단말기를 통해서 수집하는게 가장 효율적입니다. 다만 블룸버그 단말기는 데이터 적재가 원칙적으로 금지되어 있고 API 사용량을 블룸버그측에서 마음대로 결정하기 때문에 퀀트 모델링에는 적합하지 않습니다.

**시계열 특징**

아주 긴 기간을 백테스트 할수 있으면 좋겠지만 어떤 데이터는 데이터 제공 업체의 한계에 의해 시계열의 길이가 한계가 있습니다. 예를 들어  애널리스트 데이터는 1993년 이후가 되어야 사용가능하고 산업분류코드는 1994년 이전의 데이터가 없습니다. 한국 기업 재무데이터는 2000년은 지나야 그나마 가용하고 그마저도 IFRS가 도입된 2011년 이전에는 분기 데이터가 누락된 경우가 많습니다. 이럴때는 연간 데이터로 대체해서 사용하고 2011년 이후 데이터부터 분기를 사용합니다.
미국은 장기 백테스팅이 가능한 시계열 데이터를 보유중이지만 한국은 2010년 이전의 데이터에 어떤 업체 데이터를 사용하더라도 누락이 상당합니다. 당시에 있었던 회계기준 변경때문입니다.

**종목 코드**

여러 출처의 데이터를 병합하기 위해서는 코드 관리가 중요합니다. 유명한 코드에는 SEDOL, CUSIP, ISIN 이나 TICKER를 사용하기도 하는데 시간이 지남에 따라 이러한 식별자 코드들은 계속 변하며, TICKER 같은 경우는 종종 재활용되어 식별자로 사용할수 없습니다. 이런 이유로 데이터 업체들은 주식에 대해 자체 식별자를 사용합니다. 사실 영구적으로 변하지 않는 식별자를 연결하는 것은 매우 어려운 작업입니다. 한국은 거래소가 하나밖에 없어서 큰 이슈가 없지만 해외 같은 경우 주식의 종류가 다양하고 여러 거래소에 상장되어 있어 이런 종목 식별자들이 매우 자주 바뀌게 됩니다.  

쉽게말해 테슬라의 TSLA, 애플의 AAPL 같은 티커는 백테스팅에 사용할 수 없습니다. 과거에 어떤 종목들이 이런 티커를 사용했던 적이 있기 때문입니다. 또한 같은 티커로 중복된  종목이 존재하기도 합니다. 

CUSIP 같은 경우 첫 여섯자리는 회사를 의미하고 다음 두자리는 증권유형을 나타냅니다. CRSP는 Compustats와 병합되어 같은 유형의 코드를 사용하고 있습니다. KLD 데이터는 티커와 7글자 이름을 활용해 코드를 생성합니다. 

**백테스팅을 위한 툴**

백테스팅을 위한 소프트웨어는 주로 파이썬과 R을 모델링으로 사용하지만 주 데이터 핸들링에는 SQL을 사용합니다. 과거에는 Matlab이나 STATA, SAS를 사용했던 적도 있습니다만 라이센스 가격때문에 요즘은 파이썬과 R로 통합되는 추세입니다. Matrix 연산에는 확실히 Matlab과 R이 효율적이기 때문에 회귀분석등은 두 프로그램을 사용하는게 장점이 있습니다.

### 전략 테스트 기간

사실 전략의 유용성을 확인해 보는 가장 좋은 방법은 백테스팅이 아니라 실시간 테스팅방식입니다. 실시간 테스팅은 향후 몇 년동안 투자 아이디어를 실제 적용해보고 기록하는 것을 의미합니다. 이는 전략에 대한 보다 즉각적인 평가결과를 제공하는 백테스트와는 다릅니다. 문제는 시간과 돈이라는 자원이 많이 소비된다는게 한계 때문에 백테스팅을 먼저 수행하고 그중에 일부를 실시간 테스팅으로 전환하는 프로세스를 밟습니다.


일반적으로 팩터모델 테스트에는 다음과 같은 데이터 세그먼트가 포함됩니다. T0부터 현재까지인 T2까지의 기간은 백테스트이고 T2부터 T3까지는 실시간 테스르를 의미합니다. T0에서 T1은 표본데이터이고 T1부터 T2는 표본 외 테스트 데이터입니다. 

**회귀분석이나 머신러닝을 활용한 모델을 만들때는 이런 구조로 모델링을 하지만 가중치가 필요없는 단순 주식 선택모델의 유용성을 볼 때는 T1없이 T0-T2까지 테스트 합니다.**

![](/pic/2023-08-28-14-41-19.png)

분석가는 여러가지 모델을 테스트 할 때 각 테스트에서 팩터를 제거하거나 추가하면서 순차적으로 테스트를 수행합니다. 주의할 점은 모델을 테스트 하고 폐기하는 것은 데이터 마이닝의 한 형태라는 것입니다. 원래의 모델이 건전한 경제 이론에 기초해서 가설을 세우고 만들어 졌음에도 최종 모델은 그렇지 않을지도 모릅니다. 순차적 검정을 통해 좋은 결과를 보이는 모델만 선택하게 되어 최초의 가정과 완전히 다른 모델이 되는 경우가 있기 때문입니다. 따라서 분석가는 순차적 검정을 표본 내 데이터로 제한한 다음 만족스러운 모형을 찾으면 표본 외 데이터에 해당 모형을 검정해야 합니다. 예를들면 2015년까지의 데이터만 써서 좋은 성과를 내는 모델을 찾아 본 다음 2022년 현재까지도 잘 되면 통과시키고 아니면 탈락시키는 것입니다. 물론 표본 외 데이터에 대해 모형이 실패하여 탈락시키면서 다시 모델링을 수행하면 또 데이터 마이닝이 발생하기도 합니다. 100% 데이터 마이닝 Bias를 피할 수 있는 방법은 없다고 봐도 되기 때문에 건전한 경제 이론적 가정이 중요한 이유입니다.


모델을 선택한 다음 실제 투자를 집행한 뒤에 매니저에게 가장 좌절을 줄 때는 팩터와 주식 수익률 사이의 관계가 자꾸 변하는 것입니다. 시간이 지나도 안정적인 가중치를 가지는 매개 변수를 찾는 모델을 찾는 것이 중요한 이유이죠. 너무 자주 관계가 변하거나 특정 구간에만 의미 있었던 지표를 잘못 선택하게 되는 경우를 피하려면 조금 다른 방식의 접근법이 필요합니다.

모수가 시간이 지남에 따라 변하는 것처럼 보이면 데이터를 롤링 윈도우 방식으로 모델링 하여 안정성을 확인해 볼 수 있습니다. 분석가는 롤링 표본 내 창을 생성하고 다음 그림과 같이 시간에 따라 매개변수를 동적으로 재추정해 봅니다.

![](/pic/2023-08-28-14-42-19.png)

예를들어 10년치 데이터를 보유하고 있다고 가정합니다. 이 때 최초 표본 데이터는 첫 1년치 입니다. 이 1년치를 가지고 매개 변수를 추정합니다. 2010년 1월~2011년 1월 까지의 데이터로 모델링을 하고 2011년 2월 수익률을 예측합니다. 그리고 모델의 성능을 점검합니다. 그 다음 단계는 2010년 2월 ~ 2011년 2월 데이터로 모델링을 하고 2011년 3월 수익률을 예측합니다. 이는 표본 외 데이터의 끝인 2020년 1월 데이터까지 반복됩니다.

주식수익률과 팩터간의 변화를 모델링을 통해 반영할 수 있다는 점에서 장점이 있는 롤링 모델이지만 짧은 시계열을 가지고 테스트 하기 때문에 잡음이 섞여들 여지도 있어 단점이 있습니다. 분석가가 데이터를 보고 어떤 방식이 적합한지 선택해야 합니다.

### 투자 유니버스와 벤치마크 선택

분석가는 투자 분석의 대상을 미리 특정 주식 풀로 제한하고 그것을 유니버스라고 말합니다. 투자 유니버스는 몇 가지 단순한 기준을 만족하는 종목들로 정의되는 경우도 있고 어떤 스크리닝과 특별한 선별 절차에 따라 정해지기도 합니다. 단순 조건이냐  좀 심화된 조건이냐 정도의 차이 입니다. 조건을 단순화 시켜서 더 넓고 많은 주식을 유니버스로 만들면 벤치마크를 능가할 수 있는 더 많은 기회를 가질 수 있습니다만 분석해야 할 데이터가 많아지거나 너무 소형주가 들어가서 투자가능하지 않은 주식들로 구성 될 수도 있습니다.

투자 유니버스를 정하는 가장 대표적인 조건은 유동성입니다. 주식이란 투자자(여기서는 펀드 매니저)가 필요할 때마다 거래할 수 있을 정도로 유동적이어야 합니다. 보통 미국같은 경우 개인투자자들이 큰 가격 움직임을 만들지 않고 투자할만한 주식의 숫자가 3000~4000개 정도 되지만 기관은 그것보다 훨씬 적은 1000개 안팎으로 알려져 있습니다. 

좋은 벤치마크는 투자 유니버스를 잘 투영해야 합니다. 미국 주식 투자자라면 미국 주식을 대표하는 지수를 선택해야 합니다. (S&P 500, NASDAQ, 러셀3000 등) 벤치마크는 투자 가능한 주식으로 구성되어 있어야 하고 복제 가능해야 합니다. 펀드 매니너가 벤치마크의 성과를 쉽게 복제할 수 있어야 하고, 벤치마크 안에 매수할 수 없거나 유동성이 떨어지는 주식이 거의 없어야 합니다. 벤치마크는 정확하고 신뢰할 수 있어야 하고 정보가 적시에 제공되어야 투자전략의 성과 비교 및 분석이 쉽게 이루어 질 수 있습니다. 벤치마크의 구성종목은 투명하게 공개되어야 하고 벤치마크를 대상으로 하는 선물 상품이 존재하면 매우 바람직 합니다. 

마지막으로 벤치마크는 거래 회전율이 낮아야 합니다. 높은 거래비용을 발생하게 만들 정도로 자주 구성종목이 변하게 되면 이를 비교대상으로 삼는 포트폴리오들의 수익률에 지장을 초래 합니다.

**미국 벤치마크**

미국 투자자들을 위한  주요 벤치마크는 S&P500, S&P1500, Russell 1000, Russell 3000, NASDAQ 100, NASDAQ 지수등이 대표적입니다. 그 밖에 특정 스타일을 추종하는 벤치마크로 S&P 500 Barra Value(or Growth), Russel Value(or Growth) 등도 있습니다.

S&P500이 아마도 가장 많이 사용되는 벤치마크일 것입니다. S&P 지수는 Standard & Poor's 사가 정기적으로 회의를 열어 지수에서 어떤 종목을 추가하거나 떨어뜨려야 할지 결정합니다. 상장폐지, 합병 등 기업 이벤트가 있을 때, 기본 편입기준을 더 이상 충족하지 못할때는 비정기적으로 조정하기도 합니다.  지수에 속한 기업들은 유동주식의 비율이 높아야 하고 4분기 연속 수익성이 있어야 합니다. S&P500 지수의 종목들은 넓은 주식 시장을 대표해야 하기 때문에 다양한 산업으로 구성됩니다. 지수는 변경전에 사전 발표되고 변경 후에도  발표되기 때문에 미리 준비할 수 있습니다.

Frank Russell Company가 운영하는 Russell 지수는 미국 주식 포트폴리오 매니저들에게 매우 인기 있는 벤치마크 입니다. 러셀 3000 지수(Russell 3000)는 미국의 주식 시장 시가총액 상위 3000개 종목에 대한 지수입니다. 러셀 1000은 러셀 3000과 러셀 2000의 상위 1000개 주식으로 구성됩니다. Russell Small-Cap은 시총 하위 2000 종목을 포함합니다. 러셀 지수에 대한 포함 기준은 S&P 지수에 대한 것보다 덜 주관적 입니다. 프랭크 러셀 컴퍼니는 1년에 한 번, 재건일에 미국에서 거래되는 모든 보통주를 시가총액으로 순위를 매기고, 1달러 미만의 주식, 비 미국 법인 주식, 외국 주식, 미국 예탁 증권(ADR)을 포함한 특정 종목을 순위에서 제외한 상황에서 재 구성합니다. 재구성 날짜 사이의 한 해 동안 모든 주식은 다음 재구성 날짜 까지 변경 되지 않습니다.  분사 및 기타 기업 활동으로 인해 발행되거나 변경된 새로운 주식들도 지수에 그대로 포함되어 있습니다. 러셀 지수의 주식은 시가총액 가중치와 유사한 유통주식수 기준 가중치 입니다. 

나스닥 100 지수는 나스닥에서 거래되는 가장 시총이 큰 100개 회사를 포함합니다. 매년 12월에 한번  나스닥 주식은 시가총액 기준으로 순위가 매겨집니다. 순위의 상위 100위는 지수가 약간 변형된 시가총액 가중 방식으로 가중치가 부여 되는데 이는 시총이 너무 높은 기업들에게 쏠리는 현상을 완화하고자 적용한 방식입니다. 무조건 시총으로 계산하지는 않고 지수 재구성 시 원래 편입되었던 종목이 시총 상위 100위권에서  만약 101-150위 까지 하락하는 경우 지수 잔류가 허용되나, 그 이하에서는 신 주식으로 대체됩니다. 지수는 분기별로 재조정됩니다.

다우존스 산업평균지수(DJ)IA)는 개인 투자자들에게는 매우 인기있고 유명하지만 전문 포트폴리오 관리자들에게는 의미 없는 취급을 받습니다. 다우존스 컴퍼니가 미국 경제를 대표할 종목으로 선정한 30개 종목이 포함돼어 있습니다. 주식은 가격 가중치로 비중이 결정되는데 지수를 주식분할 같은 기업의 펀더멘털과 무관한 이벤트로 인한 변동에 취약하게 만드는 특성이 있어 포트폴리오 매니저들에게 인기 있는 벤치마크가 되지는 못하고 그냥 상징성으로만 남아 있습니다.

Wilshire 5000은 이름과 달리 거의 8000개가 넘는 주식들이 포함됩니다. 1974년에 시작하여 미국에 본사를 둔 거의 대부분의 기업을 포함합니다. 시가총액 가중 방식이며 가장 시장과 유사한 인덱스로 평가 받습니다.

모든 인덱스들 중에서 미국에서는 S&P500이 대표 인덱스로 불리고 있습니다. 종목수가 수천개인 지수들 보다 관리 측면에서 용이하고 그럼에도 전체 주식을 포함한 Wilshire 인덱스와 상관관계가 0.996 이상이기 때문에 대표성도 있기 때문입니다. 또한 브랜드 인지도도 높습니다.

S&P 500 지수의 주요 단점은 지표로서의 인기때문에 주식시장 수익률에 왜곡을 일으킬 수 있다는 것이 꼽힙니다. 요즘같이 ETF가 활성화 된 경우 지수의 변화를 반영하는 거래를 통해 단순히 지수에 추가되거나 지수에서 떨어지는 결과로 거래되는 유가증권의 가격이 굉장히 많이 변동하는 경우가 생겼기 때문입니다.

**한국 벤치마크**

KOSPI 200 지수는 한국거래소가 관리하며, 한국 주식 시장의 대표적인 지수 중 하나입니다. 이 지수는 한국의 주식 시장에서 시가총액이 큰 상위 200개 종목을 대상으로 구성됩니다. 그러나 이를 결정할 때 특정 기준, 예를 들어, 유동성, 거래 활성도 등도 고려됩니다. 종목의 비중은 시가총액에 비례합니다. KOSPI 200 지수의 리밸런싱은 연간 2회, 3월과 9월에 이루어집니다. 한국 매니저들 대부분이 사용하는 벤치마크입니다.

MSCI KOREA 지수는 MSCI가 관리하는 한국의 주식 시장 지수입니다. 이 지수는 한국의 주식 시장에서 가장 대표적인 기업들을 포함하며, 전 세계의 투자자들이 한국 시장에 투자할 때 자주 참고하는 지수 중 하나입니다. MSCI KOREA 지수는 MSCI의 방대한 데이터베이스와 방법론을 기반으로 선정된 한국의 주식 종목들로 구성됩니다. 선정 기준에는 시가총액, 유동성, 거래 활성도, 그리고 MSCI의 글로벌 투자 가능 지수에 포함될 수 있는 기준을 충족해야 합니다. 이 로직이 투명하게 공개되진 않습니다.  종목의 비중은 시가총액에 비례하며, 추가적으로 유동성 및 자유유동성 조정이 적용됩니다. MSCI KOREA 지수는 분기별로 리밸런싱이 이루어집니다. 일반적으로 2월, 5월, 8월, 11월에 리밸런싱이 진행됩니다. 글로벌 펀드 매니저들이 한국에 투자할때 염두에 두는 벤치마크입니다.

### 생존 편향 관리

생존 편향은 많은 투자자들이 일반적으로 범하는 오류 중 하나로, 대다수는 이를 인지하고 있지만 그 중요성을 제대로 이해하지 못하고 있습니다. 이 문제는 학계에서 광범위하게 논의되고 있지만, 실무자들 사이에서도 여전히 흔하게 발생하고 있습니다. 특히, 실무자들은 종종 현재 상장된 회사들만을 이용해 특정 투자 전략을 백테스트하는 경향이 있습니다. 이는 결국 파산, 상장 폐지, 또는 인수로 인해 투자 유니버스에서 사라진 주식들을 백테스트에 포함하지 않게 됩니다. 이러한 생존 편향은 지나치게 긍정적인 결과를 초래하거나, 때로는 완전히 반대의 결과를 가져올 수 있습니다.

예를 들어, 1986년 12월 31일에 Russell 3000 지수에 포함되어 있었지만, 오늘날까지 존재하는 회사들만을 대상으로 하면, 수년간 지수에서 제외된 회사들을 배제하게 됩니다. 

![](/pic/2023-08-28-14-54-26.png)

Figure3을 보면 투자 유니버스는 시간이 흐를수록 점차 축소되는 것을 확인할 수 있습니다. 실제로, 지난 28년 동안 지수의 3000개 주식 중에서 500개 미만의 주식만이 존속하고 있습니다. 이러한 살아남은 주식들의 성과(동일가중 평균)를 동일가중 Russell 3000 지수와 비교해보면, 지수에서 제외된 주식들이 대부분 파산, 상장 폐지, 또는 장기간의 성과 부진으로 인한 것이기 때문에, Figure4를 보면 살아남은 주식들의 평균 성과가 전체 지수에 비해 앞도적으로 우수한 것을 확인할 수 있습니다.

이러한 내용을 고려하면, 생존 편향의 문제는 투자 전략을 평가하고 결정할 때 중요한 요소로 간주되어야 합니다.

생존 편향은 때로는 완전히 상반된 결과를 초래하기도 합니다. 

![](/pic/2023-08-28-14-58-21.png)

Figure 5와 6은 Russell 3000 지수의 전체 주식과 생존 편향이 반영된 주식을 대상으로, 머튼의 부도 팩터를 기반으로 구성된 상위 및 하위 포트폴리오의 성과를 보여줍니다. 올바른 투자 유니버스를 사용한 적절한 백테스트(Figure 5) 는, 부도 확률이 낮은 회사의 주식이 부도 확률이 높은 주식보다 성과가 훨씬 좋다는 것을 보여주며, 이는 우량성 프리미엄이 존재한다는 것을 시사합니다. 그러나, 생존 편향이 반영된 유니버스(Figure 6)를 사용한 백테스트는 정반대의 결과를 보여줍니다. 신용 위험이 가장 높은 기업의 주식이 우량성이 좋은 주식에 비해 7배나 높은 누적 수익률을 보이는 것은 놀랍습니다.

이를 감안하면, 생존 편향의 문제는 투자 전략을 평가하고 결정할 때 매우 중요한 요소로 간주되어야 합니다. 그림 5와 6에서 볼 수 있는 것처럼, 생존 편향이 반영된 데이터를 사용하면 전혀 다른 결론에 도달할 수 있으므로, 투자 전략을 평가하고 결정할 때 이를 반드시 고려해야 합니다.

**생존 편향을 피하는 방법**

- 백테스트를 위한 유니버스는 인덱스에서 제외되거나, 파산, 상장 폐지, 인수 등으로 인해 상장이 폐지된 모든 기업들을 포함해야 합니다.
- 지수(예: S&P 500 또는 MSCI World)를 사용하는 경우, 특정 시점을 기준으로 지수에 포함된 종목만을 사용하고, 그 이후에 지수에 추가된 종목은 제외해야 합니다.
누락된 값을 적절하게 처리해야 합니다. 
- 특정 주식의 팩터 점수를 사용할 수 없는 경우, 해당 주식을 유니버스에서 제외하기보다는 NA(사용할 수 없음)로 표시해두어야 합니다.

이러한 조치들은 생존 편향을 방지하고, 백테스트의 정확성을 높이는 데 중요합니다.

### 미래 참조 편향 관리

백테스팅을 수행할 때 두 번째로 흔히 발생하는 오류는 '미래참조 편향'입니다. 이것은 백테스팅이 수행되는 시점에 알려지지 않았거나 사용할 수 없는 정보나 데이터를 사용함으로써 발생하는 편향입니다.

특히, 기업들의 재무제표 데이터에는 미래참조 편향이 존재할 가능성이 높습니다. 기업들은 일반적으로 분기별, 반기별, 또는 연간별로 재무 상태를 보고합니다. 대부분의 기업들은 회계연도가 종료된 후 재무제표를 작성하여 공시하는데 평균적으로 1~2개월이 소요되며, 때로는 더 오래 걸리는 경우도 있습니다(그림 12 및 그림 13 참조). 따라서, 일반적으로 백테스트를 수행할 때는 시간 지연을 고려해야 합니다. 즉, 회계 기간이 종료된 후 x개월 동안은 재무제표를 알지 못한다고 가정해야 합니다. 일반적으로 백테스트에서는 3개월의 시간 지연을 적용합니다.

![](/pic/2023-08-28-15-02-09.png)

이러한 접근 방식은 백테스팅 과정에서 미래참조 편향을 방지하고, 결과의 정확성을 높이는 데 중요합니다.

**재작성 편항**
더 복잡한 형태의 미래참조 편향 중 하나는 '재작성 편향'입니다. 기업들은 다양한 이유로 재무제표를 재작성하는 경우가 종종 있습니다. 일반적으로 많이 사용되는 데이터 소스들은 최종적으로 수정된 데이터만을 저장합니다. 우리의 이전 연구에서는, 경제 데이터가 자주 재작성되며, 이러한 재작성이 통계적으로 중요하다는 것을 보여주었습니다. 중요한 점은, 첫 번째로 보고된 데이터를 사용할 것인지, 아니면 재작성된 데이터를 사용할 것인지에 따라, 모델의 결과가 매우 달라질 수 있다는 것입니다.

이러한 점을 감안할 때, 백테스팅을 수행할 때는 이러한 재작성 편향을 반드시 고려해야 하며, 사용하는 데이터가 최초로 보고된 데이터인지, 아니면 재작성된 데이터인지를 명확히 해야 합니다.

이상적인 해결책은 <u>공급업체가 원래 보고된 데이터와 날짜를 보관하고, 데이터가 수정될 때 마다 이를 추적하는 PIT(Point In Time, 시점) 데이터베이스를 사용하는 것입니다.</u> 예를 들어, Compustat, Capital IQ 및 Worldscope는 모두 미국 및 글로벌 기업을 위한 PIT 재무 데이터를 제공합니다.

PIT가 아닌 데이터는 모든 재무 제표 데이터 항목이 회계 기간이 끝나는 날에 사용 가능하다고 가정합니다. 반면, Figure 14는 PIT가 아닌 데이터, Figure 15는 적절한 PIT 데이터를 사용하여 밸류 팩터의 성과를 보여줍니다. PIT가 아닌 데이터, 즉 편향된 데이터는 밸류 팩터의 성과를 약 60% 가량 부풀리는 것이 확인됩니다.

![](/pic/2023-08-28-15-06-33.png)

**PIT 데이터가 없다면?**

PIT(Point-In-Time) 데이터를 사용할 수 없는 경우, 투자자들은 일반적으로 어떤 형태의 보고 지연을 적용하곤 합니다. 그러나, 정확한 지연 시간을 설정하기 위한 명확하고 간단한 기준은 없습니다. 지연 시간을 너무 짧게 설정하면, 잠재적으로 미래참조 편향에 노출될 위험이 있습니다. 반면, 보고 지연을 너무 길게 설정하면, 사용하는 데이터가 너무 오래되어 실질적인 가치가 없을 수 있습니다.

데이터 커버리지도 고려할 필요가 있는 중요한 요소입니다. 예를 들어, Worldscope는 국제 기업들의 재무제표를 제공하는 서비스 업체로, 최근 몇 년 동안 PIT(Point-In-Time) 데이터베이스를 도입했습니다. 그러나, 1999년 이후로만 PIT 데이터와 비 PIT 데이터가 동일한 범위의 종목을 다룹니다. Figure 16은 S&P BMI UK 유니버스에서 ROE(Return on Equity)를 예시로 들어, PIT 데이터를 포함하는 종목의 수를 보여줍니다.

공정한 비교를 위해, 1999년 이후의 데이터만을 사용하여 영국 유니버스에서 ROE 팩터를 백테스트하기 위해 다음과 같은 설정을 했습니다.

- PIT 데이터: 가장 현실적인 경우
- 비 PIT 데이터 및 보고 지연이 없는 경우: 상당한 미래 참조 편향
- 비 PIT 데이터 및 한달의 보고 지연: 기업의 회계기간 종료 후 한달 뒤 ROE를 안다고 가정
- 비 PIT 데이터 및 두달의 보고 지연
- 비 PIT 데이터 및 세달의 보고 지연

![](/pic/2023-08-28-15-09-58.png)

Figure 17에서 보여지는 것처럼, PIT(Point-In-Time)가 아닌 데이터를 사용하고 보고 지연 가정을 적용하지 않으면, ROE(Return on Equity) 팩터의 성과가 10% 상승합니다. 만약 PIT 데이터가 없어서 비 PIT 데이터를 사용한다면, 보고 지연 가정을 1개월에서 3개월로 늘리면 ROE 팩터의 성과는 9% 감소합니다. 이러한 보고 지연 가정을 3개월로 설정하는 것은 다소 보수적인 접근으로 보일 수 있습니다. 오래된 데이터를 사용하면 팩터의 성과가 적절한 PIT 데이터를 사용하는 것에 비해 낮아질 수 있습니다. "이상적인" 보고 지연은 약 2개월로 보입니다. 그러나 대부분의 연구에서는, 비 PIT 데이터를 사용할 때 보수적으로 3개월의 보고 지연을 적용하는 것이 일반적입니다.

백테스팅에서 미래참조 편향을 방지하기 위한 몇 가지 방법은 다음과 같습니다.

- 가능하다면 PIT(Point-In-Time) 데이터를 사용하세요.
- PIT 데이터를 사용할 수 없는 경우, 보고 지연을 보수적으로 설정하세요. 
- 백테스팅 결과가 직관적으로 이해되지 않을 때는 결과를 다시 검토하세요.
- 전략을 실제 세계에 적용해서 성과가 백테스트와 일치하는 테스트 해보세요.

### 벡테스팅 기간 설정

새로운 전략을 개발하고, 데이터를 확보할때, 자주 제기하는 비판 중 하나는 "데이터의 기간이 너무 짧고, 역사적으로 충분하지 않다"는 것입니다. 이것은 어떤 면에서는 정당한 의견입니다. 물리학과 달리, 경제학과 금융학에서는 시간에 따라 관측치가 하나뿐입니다. 즉, 시계열 데이터를 다룹니다. 데이터가 정상성을 가지고 있다 해도, 매우 긴 기간의 월간 데이터나, 적절한 기간의 일간 또는 틱 단위 데이터가 필요합니다. 그렇다면 얼마나 긴 기간의 데이터가 필요할까요?

작은 표본에서 발생하는 편향을 극복하기 위해, 최소한 몇 번의 경제 사이클이나 국면 변화를 포함하는 길이의 기간에 백테스트를 수행하는 것이 필요합니다.  국면 변화는 금융 분야에서 흔한 현상입니다. 특히 1990년대 후반의 인터넷 버블, 2007년 퀀트 위기, 그리고 2008년 글로벌 금융 위기는 새로운 체제로의 전환점으로 자주 인용됩니다. 되도록이면 이런 기간이 포함되는게 좋습니다.

예를 들어, 가치와 모멘텀은 학계와 실무자 모두에게 가장 널리 인정받는 팩터들입니다. 1997년 말에 투자를 시작하여 미국 주식의 PER(Price to Earnings Ratio) 팩터를 백테스트했다고 가정해봅시다. Figure 22에서 보여지는 것처럼, 가치는 명백히 우수한 전략입니다. 이 전략에 대한 설명이 설득력이 있기 때문에, 가치주에 투자할 것입니다.

그런데 1997년 말부터 2000년 중반까지의 2년 반 동안 기술 섹터의 가치 전략의 성과를 살펴보겠습니다. 결과는 참담했습니다. 매우 짧은 기간 동안 거의 70%의 손실이 발생했습니다(Figure 23 참조). 2000년 초, 많은 이들이 기존의 사업 모델이 곧 이른바 '새로운 경제'로 완전히 대체될 것이라고 말했습니다. 이것은 '닷컴' 시대와 함께 가치가 더 이상 중요하지 않다는 의미였습니다. 불행하게도, '새로운 경제' 이론은 오래 지속되지 않았습니다. 

2000년 중반부터 2002년 후반까지의 2년 반 동안, 전통적이고 지루한 가치 스타일이 강력하게 회복되었습니다(Figure 24 참조).

그런데 또한 1995년부터 2014년까지의 성과(Figure 25 참조)를 보면 가치 전략은 엄청난 변동성에도 불구하고 수익률은 소수점 이하입니다.

![](/pic/2023-08-28-15-22-27.png)

Figure 26에서 볼 수 있듯이 기술 섹터에서 전형적인 가치 전략의 샤프 비율은 상당히 나쁘기도 하며 상당히 좋기도 합니다.

답은 사실 정해져 있습니다.

- 가능한 한 오랜 기간을 사용하여, 가능한 한 많은 경제 및 정책 주기에 백테스트를 한다.
- 다양한 경제 사이클에서 성과가 어떻게 달라지는지 이해하고 정부 정책의 개입이 잠재적으로 새로운 레짐을 만드는지를 조사한다.

### 데이터 마이닝(데이터 스누핑) 편향 관리

데이터 마이닝이란 용어는 매우 흥미롭습니다. 컴퓨터 과학과 통계학에서는 이것이 정교한 통계 기법, 계산 알고리즘 및 대규모 데이터베이스 시스템과 관련된 대규모 데이터 세트에서 패턴을 찾는 계산 프로세스를 나타냅니다. 그러나 금융 분야에서는 분석가들이 원하는 패턴을 찾기 위해 데이터나 모델을 '조작'하는 것을 의미하는 경우가 많습니다.

금융에서 데이터 마이닝의 의미는 데이터 스누핑, 즉 표본 내 모델에 완벽하게 맞는 패턴이나 규칙을 광범위하게 검색하는 행동입니다. 분석가들은 종종 모델의 변수를 미세하게 조정하고 백테스트에서 수익률이 높은 변수를 선택합니다.

데이터를 충분히 조작하면, 샘플에서 매우 잘 작동하는 모델을 만들 수 있습니다.

![](/pic/2023-08-29-08-12-23.png)

데이터 스누핑 편향은 아마도 다루기 가장 어려울 것이며, 아마도 이를 완전히 피할 수 없을 것입니다. 그러나 몇 가지 기본적인 검사를 통해 이러한 편향을 약간이나마 피할 수 있습니다.

- 최소한으로, 우리는 미래참조 편향을 피해야 합니다. 모델과 백테스트 전략을 구축할 때, 해당 시점에서 사용 가능한 point-in-time 데이터만을 사용해야 합니다.
- 팩터 자체보다는 미리 정의된 일련의 규칙에 따라 백테스트를 실시하는 것이 중요합니다. 사전에 계획대로 결과가 나온 팩터에 이런 저런 수치를 바꿔보는 추가적인 작업을 해서는 안됩니다.
- 한 국가에서 잘 작동한 전략을 아웃오프 샘플로 다른 국가나 지역에도 적용해 봅니다.
- 절대로 표본 외 데이터인 "검증 샘플"을 만지지 말고 최종 모델 성능을 확인하는 데만 사용해야한다.
- 진정한 테스트는 실제 성과입니다.

### 거래비용과 회전율 

백테스팅은 종종 거래 비용이 없고, 회전율 제약이 없으며, 무제한으로 롱숏이 가능한 이상적인 세계에서 수행됩니다. 그러나 현실적으로 모든 투자자들에게는 제약 사항이 존재합니다. 이 장에서는 가장 중요한 제약 중 하나인 회전율과 관련된 거래 비용의 문제에 대해 논의합니다.

**거래 비용의 영향**

우선, 회전율이나 신호 감소가 왜 중요한지 궁금할 것입니다. 헤지펀드에서 자산을 관리한다면, 아마도 당신이 원하거나 당신이 적합하다고 생각하는 만큼 당신의 포트폴리오를 바꿀 수 있는 상당한 유연성을 가지고 있을 것입니다. 문제는 신호가 빠르게 감소하는 팩터에서 매력적인 수익을 거두기 위해서는 상당한 회전율이 필요하다는 것입니다. 높은 회전율은 더 많은 거래를 의미하며, 거래에는 비용(수수료, 비드-애스크 스프레드, 시장 충격, 롱숏 포트폴리오의 경우 주식 대여 수수료)이 발생합니다.

전형적인 예는 단기적인 반전, 즉 최근에 실적이 좋았던 주식(예: 지난 달)이 그 다음 달에 하락할 가능성이 더 높다는 전략에서 발견 됩니다. 반전 팩터는 수수료를 고려하지 않으면 일본 주식 시장에서 상당히 훌륭한 주식 선택 전략입니다. 우리의 최근 연구 논문에서, 일본의 배당금 지급 주식들 사이에서 반전이 훨씬 더 강력한 예측력을 가지고 있다는 것을 발견했습니다. 그러나 이는 거래비용을 감안하지 않을 때만 유의미 합니다.

![](/pic/2023-08-29-08-31-48.png)

Figure 31에서 볼 수 있듯이 거래 비용이 없는 이상적인 세계에서 단순한 롱숏 전략(일본의 배당을 주는 주식 중 전월 실적이 가장 나쁜 상위 20 % 주식을 매수하고 이전 달 수익률이 가장 높은 하위 20% 주식을 공매도)은 연간 수익률 12%를 기록하여 고전적인 가치 전략인 PBR 팩터를 능가합니다. 그러나 거래 당 비용을 0bps에서 30bps로 늘리면 PBR 팩터의 초과 수익은 완만하게 떨어지는 반면, 반전 전략의 알파는 음수로 떨어집니다.

이런 경우 현실에서는 직접적인 거래비용은 줄이기가 힘들기 때문에 매매를 줄이는 회전율 제약 조건을 넣습니다.

현실적으로 모든 매니저가 원하는 만큼 거래할 수 있는 것은 아닙니다. 대부분의 기관 포트폴리오 관리자들은 어느 정도의 회전율에 대한 제약이 있습니다. 회전율 제약 조건은 모델의 성과에 상당한 영향을 미칠 수 있습니다. 

예를들어 N-LASR 글로벌 주식 선택 모델을 예로 들어 회전율 제약의 영향을 살펴보겠습니다. N-LASR은 AdaBoost라는 기계 학습 기술을 사용합니다. N-LASR 모델은 아웃 오브 샘플에서 훌륭한 성과를 보여줍니다. 그러나 이를 달성하려면 포착하려면 상당히 높은 수준의 회전율이 필요합니다.

![](/pic/2023-08-29-08-35-53.png)

회전율이 떨어질수록 성과가 안좋아지는 것을 확인할 수 있습니다.

단순히 이상적인 세계에서의 백테스팅 결과를 보여주는 전략이 아닌, 거래비용을 최소화 하면서 성과를 극대화 할수 있는 최적의 리밸런싱 주기를 찾는 것이 제일 중요하다고 할수 있습니다.

**최적의 리밸런싱 주기**

회전율 제약이 엄격하다고 해서 반드시 리밸런싱 빈도가 매우 느려야 한다는 뜻은 아닙니다. "우리는 장기 가치 투자자입니다. 우리는 3년에서 5년 동안 주식을 보유합니다. 그러므로 우리는 일년에 한 번 리밸런싱을 합니다.“라고 말하는 사람들이 많습니다. 새로운 정보는 끊임없이 들어오고 우리는 그에 따라 우리의 모델과 생각을 조정해야 합니다. 비록 우리가 엄격한 회전율 제약이 있더라도, 여전히 포지션을 자주 조정하고 싶어할 수 있습니다.

잦은 리밸런싱의 이점을 설명하기 위해 간단한 예제를 사용하겠습니다. 연간 36% 이하의 상당히 엄격한 회전율 제약이 있는 롱-온리이면서 가치주에 틸트된 공모 펀드 포트폴리오를 운용한다고 가정해봅시다. Russell 3000 지수를 추종하면서 PER 팩터에 틸트를 하며, 연간 추적 오차가 4%인 두 개의 롱 온리 포트폴리오를 구성합니다.

- 연간 리밸런싱: 매년 12월 31일에 포트폴리오를 리밸런싱하며, 회전율을 36%로 제한합니다.
- 월간 리밸런싱: 매월 말 포트폴리오를 리밸런싱하며, 월간 회전율을 3%로 제한합니다. 따라서 연간 회전율은 36%로 동일합니다.

![](/pic/2023-08-29-08-40-00.png)

두 전략 모두 벤치마크를 능가하지만 더 자주 리밸런싱을 하는 전략(월간 리밸런싱)은 난 26년 동안 연간 리밸런싱 전략보다 성과가 1.5배 좋습니다.

회전율 제약이 있더라도 가능한 한 자주 포지션을 조정하는 것이 유리할 수 있습니다.
만약 각 리밸런싱 주기마다 종목 교체가 많지 않은 전략이라면 잦은 리밸런싱이 더 나을수도 있다는 뜻입니다.

사실 회전율, 리밸런싱 주기 및 거래 비용을 고려해 최적으로 가중치를 부여하는 방법은 어느 정도 과학이면서도 예술의 영역에 가깝습니다.

### 팩터 성과 측정

팩터 성과를 측정할 때 일반적으로 다음 두 가지 측정 기준을 고려합니다.

**롱/숏 수익률의 차이**
- 팩터 기준에 따라 상위 순위의 주식을 매수하고 동시에 최하위 순위의 주식을 공매도하여 롱/숏 헤지 포트폴리오를 구성합니다.
- 매수 (그리고 공매도) 포트폴리오의 주식은 동일 비중 혹은 시가총액 비중으로 가중합니다.
- 분위수의 선택은 데이터 샘플의 크기에 따라 다르며, 3 분위수, 4 분위수, 5 분위수, 10 분위수 및 때로는 백분위수를 사용합니다.
- 롱/숏 포트폴리오는 주기적으로 리밸런싱합니다.

**순위 정보 계수 (Rank Information Coefficient, RIC)**
- RIC는 주어진 날짜의 팩터 점수를 기반으로 한 주식의 순위와, 다음 기간(일반적으로 다음달)의 수익률을 기반으로 한 주식의 순위 간의 상관관계를 살펴봅니다.
이 작업을 매 기간마다 반복합니다.

Figure 60과 Figure 61은 각각 Earning yield와 Price Momentum에 대한 매수와 공매도 포트폴리오의 초과 수익률을 보여줍니다. Earning yield의 알파는 매수 포지션에 집중되는 반면 Price Momentum의 초과수익률은 공매도 포지션에 의해 좌우됩니다. 공매도 포지션 없이 매수만 하려는 투자자에게는 Earning Yield 팩터가 더 적절하다고 볼수 있습니다.

![](/pic/2023-08-29-08-58-10.png)

