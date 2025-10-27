붙여주신 HTML/JS 안에서 **페이지가 내부적으로 호출하는 Ajax(API) 엔드포인트**를 그대로 뽑아보면 아래 두 가지가 핵심입니다.

---

### 1) 지표/표 데이터(JSON) — cF4002.aspx

코드:

```javascript
$.getJSON('../company/cF4002.aspx', param, function (res) { ... })
```

**절대경로**: 
  
    https://navercomp.wisereport.co.kr/v2/company/cF4002.aspx
    
    (현재 문서가 /v2/company/c1040001.aspx 이므로 ../ 적용)
    
**요청 파라미터(예시)**:
    
    - cmp_cd: 종목코드 (예: 005930)
        
    - rpt: 탭별 리포트 코드
        
        - 투자분석 상단 탭에서 data-rpt로 전달 (예: 수익성=1, 성장성=2, 안정성=3, 활동성=4 … — 페이지마다 다를 수 있음)

    - finGubun: 재무 기준 (MAIN, IFRSS, IFRSL, GAAPS, GAAPL)
        
    - frqTyp: 주기 (0=연간, 1=분기)
        
    - frq: 주기(추가 필드, 쿠키에서 덮어쓸 수 있음 — 페이지는 둘 다 보냄)
        
    - cn: 공란('')
        
    - encparam: 페이지가 넣어주는 토큰(예: T2lHUVhiZkcyWmtvc1E4YzJXY0NWdz09)
        
**샘플 URL**:

```
https://navercomp.wisereport.co.kr/v2/company/cF4002.aspx
  ?cmp_cd=005930
  &rpt=1
  &finGubun=IFRSL
  &frqTyp=0
  &frq=0
  &cn=
  &encparam=T2lHUVhiZkcyWmtvc1E4YzJXY0NWdz09
```

---

### 2) 차트 데이터(JSON) — c1030001.aspx

코드:

```javascript
$.ajax({
  url: "/company/chart/c1030001.aspx",
  data: { cmp_cd, rpt: 6, finGubun, frq, chartType: '' },
  ...
})
```

**절대경로**: 

    https://navercomp.wisereport.co.kr/company/chart/c1030001.aspx
    
    (/company/chart/... 는 루트 기준 절대경로)
    
**요청 파라미터(예시)**:
    
    - cmp_cd: 종목코드 (예: 005930)
        
    - rpt: 6 (이 페이지의 “가치분석(주당지표/가치지표)” 차트용으로 고정)
        
    - finGubun: 재무 기준 (MAIN, IFRSS, IFRSL, GAAPS, GAAPL)
        
    - frq: Y(연간) 또는 Q(분기)
        
    - chartType: 공란 또는 svg/image
        
**샘플 URL**:

```
https://navercomp.wisereport.co.kr/company/chart/c1030001.aspx
  ?cmp_cd=005930
  &rpt=6
  &finGubun=IFRSL
  &frq=Y
  &chartType=
```

---

### 참고(이미지 차트 파일)

차트 이미지를 직접 불러오는 경로도 있습니다(데이터가 아니라 PNG).

코드 예:

```
'/chartimg/page3_4/chart1/' + cmp + frq + rpt + finGubun + '.png?20251024'
```

- 예) https://navercomp.wisereport.co.kr/chartimg/page3_4/chart1/005930Y2IFRSL.png?20251024

---

### 바로 써볼 수 있는 cURL 예시

**표/리스트 데이터(JSON)**

```
curl -G 'https://navercomp.wisereport.co.kr/v2/company/cF4002.aspx' \
  --data-urlencode 'cmp_cd=005930' \
  --data-urlencode 'rpt=1' \
  --data-urlencode 'finGubun=IFRSL' \
  --data-urlencode 'frqTyp=0' \
  --data-urlencode 'frq=0' \
  --data-urlencode 'cn=' \
  --data-urlencode 'encparam=T2lHUVhiZkcyWmtvc1E4YzJXY0NWdz09' \
  -H 'User-Agent: Mozilla/5.0'
```

**차트 데이터(JSON)**

```
curl -G 'https://navercomp.wisereport.co.kr/company/chart/c1030001.aspx' \
  --data-urlencode 'cmp_cd=005930' \
  --data-urlencode 'rpt=6' \
  --data-urlencode 'finGubun=IFRSL' \
  --data-urlencode 'frq=Y' \
  --data-urlencode 'chartType=' \
  -H 'User-Agent: Mozilla/5.0'
```

---

### 요약

- **직공 대상 API**:
    
    1. /v2/company/cF4002.aspx (표/그리드 데이터)
        
    2. /company/chart/c1030001.aspx (차트 데이터)
        
    
- **필수 파라미터**: cmp_cd, rpt, finGubun, frq/frqTyp, (그리고 cF4002.aspx에는 encparam)
    
- **전략**: 종목코드만 바꿔서 300개 루프 돌리면 됩니다. 쿠키/토큰(encparam)이 바뀌는 경우가 있으니, 실제 요청 직전에 **페이지에서 읽어온 최신 값**을 넣어주세요.