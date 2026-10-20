# HSSD 대역별 위치·yaw 정보 / 에너지–복소 coherency 결합 결과

## 결론

24kHz 직전까지 원본에 에너지가 존재하지만, 높은 주파수일수록 유효한
coherency 기반 pose 분별력이 커지지는 않았다. 가구100%에서 complex
coherency의 XY 정보는500–1500Hz, yaw/joint 정보는1500–4000Hz에서 상대적으로
높았다. 이는 이 scene에서의 관측이며 보편적인 최적 대역 선정 결과가 아니다.

전체 대역 에너지와 coherency의 단순 결합은 전체 대역 coherency 단독보다
좋았지만, 기존375–3000Hz coherency와 visual-only를 일관되게 넘지 못했다.
따라서 "24kHz까지 확장"이나 "진폭+위상 고정 결합"만으로 clutter 문제가
해결된다는 근거는 얻지 못했다. 고주파에서도 에너지 기반 위치 정보는 일부
남아 있어 "고주파에는 정보가 전혀 없다"고 결론 내리면 안 된다.

## 방법

승인된 새 접근은 에너지–복소 coherency 고정 결합이다.

- E: 마이크×대역×시간의 band-summed |STFT|², 전체 합 정규화.
  절대 공통 이득 제거, 상대 에너지 및 시간 패턴 보존.
- C: 인접6마이크 쌍의 normalized cross-spectrum, Re/Im 보존.
- D_E=L1(E_obs,E_candidate)/2.
- D_C=Re/Im L1/(2sqrt(2)×마이크쌍수×주파수bin수).
- D_combined=.5D_E+.5D_C. 이론적 범위 정규화이며 학습/가중치 튜닝 없음.
- MSC=|C|² 단독은 위상 제거 대조군.

같은 fixed direct sample7, 같은 Hann256/hop64, 같은 시간 구간으로 비교했다.
DC와 Nyquist를 제외하고187.5–23812.5Hz 양의127개bin을 사용했다. 최저 대역은
2개bin뿐이다. 전체 대역 에너지는8개 대역 축을 유지하며 C는127개bin을 균등
가중한다. 대역 경계는 STFTbin 선택이므로 분석창에 따른 leakage가 있다.
새 Energy ablation은 기존1746dim legacySTFT-L1의 완전 재현이 아니며 이를
구분해 표기했다. 원래375–3000Hz C 거리/argmin은 양 관측 조건에서 재현했다.

## 제한 후보 bank 진단: 가구100%, 2–50ms

80위치×36yaw 후보, 480관측(80위치×6물리yaw). XY는 GT yaw 고정, yaw는 GT
XY 고정, joint는 둘 다 검색. 수치는 정답cell/bin exact top1(%), 동점chance
credit. GT포함 제한 bank이며 dense-global 결과가 아니다.

| 대역 kHz | C XY | C yaw | C joint | 결합 XY | 결합 yaw | 결합 joint |
|---|---:|---:|---:|---:|---:|---:|
| 0–0.5 | 8.75 | 10.42 | 0.21 | 18.96 | 11.67 | 1.46 |
| 0.5–1.5 | 22.92 | 16.67 | 3.96 | 18.75 | 15.00 | 3.96 |
| 1.5–4 | 15.83 | 18.12 | 6.25 | 21.04 | 16.67 | 6.46 |
| 4–8 | 9.58 | 15.42 | 2.92 | 15.00 | 14.37 | 6.04 |
| 8–12 | 3.96 | 8.54 | 1.25 | 15.62 | 8.75 | 4.17 |
| 12–16 | 3.54 | 6.25 | 0.42 | 15.83 | 6.04 | 1.46 |
| 16–20 | 1.67 | 3.54 | 0.62 | 14.58 | 4.17 | 1.88 |
| 20–24 | 3.54 | 4.58 | 0.42 | 14.58 | 4.79 | 1.88 |
| 全 대역 | 8.54 | 17.50 | 2.50 | 17.50 | 16.25 | 5.42 |

Chance는 XY1.25%, yaw2.78%, joint0.0347%. 전체 대역 joint는 Energy2.92%,
C2.50%, MSC0.62%, Combined5.42%. Combined−C +2.92pp,
80물리 위치 cluster bootstrap95% CI[+0.42,+6.04],2000회seed42,다중비교보정없음.
그러나 기존375–3000Hz earlyC는 XY21.46/yaw18.33/joint6.04%였으므로 전체
대역 결합이 기존 좁은 대역 접근을 개선했다고 주장할 수 없다.

20–24kHz 에너지 단독은 XY15.83/yaw4.17/joint2.08%였다. 이 대역에서는
진폭·시간 패턴의 위치 단서가 C보다 강하고, yaw단서는 약하다.

## 독립 가구 없음 대조군 / 긴 구간

가구 없는 독립 관측에서도20–24kHz C는 XY5.42/yaw6.88/joint0.83%, 결합은
79.38/11.46/10.42%였다. 따라서 높은 대역 C의 약함을 clutter 하나로만 설명할
수 없다. 위상 wrapping/배열 공간 샘플링/반향 합성 및 독립 mono 채널 렌더의
영향은 가능한 설명이며 이번 실험이 원인을 분리해 증명한 것은 아니다.

2–128ms 가구100% 전체 대역은 C joint2.29%, 결합5.42%,
차이+3.125pp [0.833,6.042]. 이 경우도 기존 좁은 대역보다 낫지 않다.
독립empty 및 긴 구간의 모든 대역/4방법 지표는 JSON/HTML에 공개했다.

## 실제 visual top-K: 가구100%, 전체 대역, 2–50ms

동일한 이전 audit checkpoint의80yaw0query 및 고정 후보. Visual-onlyR@1m45%.
A는36yaw 최소화, C조건은 후보별 visual yaw고정. B(GTyaworacle)도HTML에 공개.
아래 C단독은 전체 대역 Complex C이며, 기존375–3000Hz와 구분해야 한다.

| K | 전체 대역 C단독 A | 결합 A | 결합 C조건 |
|---|---:|---:|---:|
| 5 | 42.50 | 46.25 | 43.75 |
| 10 | 38.75 | 45.00 | 46.25 |
| 20 | 32.50 | 42.50 | 45.00 |
| 50 | 28.75 | 35.00 | 35.00 |

결합A의K20은 C단독보다+10pp [1.25,18.75]이나 visual45%보다 낮다.
결합A의K50은 C보다+6.25pp [-1.25,13.75], visual보다−10pp [-21.25,1.25].
결합A의K5는 visual보다+1.25pp [-7.5,10]. 따라서 일관된 visual 보완 이득을
확인하지 못했다. 2–128ms 결합 C조건 K10=47.5%도 visual대비CI[-6.25,11.25]pp로
0을 포함한다. 가장 높은 결과만 골라 성공으로 해석하지 않는다.

## 검증·출력

원본RIR17796candidate+480independentempty+480furnished=18756파일SHA/paired
metadata를 확인했다. 작업319.7초. 가용CPU16개 작업자/거리계산4스레드,
GPU학습 변경 없음. 제한144조건×480=69120query, 실제topK7680query행.

저장된 float64 component 거리에서 제한144조건의 선택/공동정확도 및 실제
topK7680행의 선택/yaw/거리를 독립 재계산했고, 거리범위0–1과 후보동일성,
sourcehash를 확인했다. Combined는 저장된 E/C에서 정확히 재구성할 수 있다.
시각화는Firefox에서 조건/시간/K/query/method/A-B-C 변경 및 RGB/map/표를
확인했다. Scoped pytest258passed.

원본: `/ssd1/AV-FPLoc/exp4_acoustic_xy_observability/analysis/hssd_frequency_probe_20261017/`
HTML: http://10.20.20.113:8766/hssd-frequency-probe-20261017/index.html

RIRfeature은행, 모든 component거리, 조건별예측,입력SHA,CSV/규약/전체summary를
보존했다. 기존결과/학습/데이터split은변경하지않았다. 단일testscene 탐색이며
모든GT yaw가다양한v4실험이나실측일반화로해석하지않는다.
