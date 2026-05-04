# DiffGI Rebuttal Draft (#12200)

> 1쪽 한도 기준 한국어 item-based 초안. 모든 수치는 main paper Table 1–5 및 Supp. Sec 1–7에서 직접 인용. 영어 LaTeX 변환 시 그대로 직역 가능하도록 문장 단위 압축.

리뷰어 분들의 자세한 피드백에 감사드립니다. 우려 사항을 5개 주제로 묶어 답변드립니다 (괄호 안 ID는 각 리뷰어 의견과의 대응).

**Minor 정정 (R3 minor).** Introduction의 GIMDiffusion 인용은 [15]→[11]로 수정하고, "multi-chart geometry image" 용어를 일관되게 사용하도록 camera-ready에 반영하겠습니다.

---

## 1. Representation의 일반성 — UV·Resolution·Domain Coverage (R2.1, R2.2, R3.2)

**[UV 의존성은 한계가 아니라 강점입니다.]** Multi-chart GI 계열 공통의 본질적 속성이며, 본 논문이 타겟하는 garment·furniture 자산은 산업 표준 UV가 이미 부착된 형태로 유통됩니다 (ABO: 7,900개 commercial furniture, GarmageSet: 14,801개 professionally designed garments). 잘 정돈된 UV로 학습한 결과, **생성된 mesh가 UV가 정돈된 채로 출력**되어 simulation·texturing 파이프라인에 직접 투입 가능한 downstream 이점이 발생합니다.

**[Resolution과 압축률.]** Supp.~Sec.~1에서 reconstruction error는 해상도 256 이상에서 수렴함을 보였습니다. 8× 공간 압축한 32×32×4 latent로도, 압축이 없는 Omages(64×64×4)를 CD/JSD/NC 모든 지표에서 능가합니다 — TSDF의 redundant boundary encoding이 latent compression에 robust하기 때문입니다. 더 높은 해상도로의 확장은 latent grid 차원만 늘리면 됩니다.

**[Domain coverage와 GarmageNet의 trade-off.]** GarmageNet은 패널당 72-dim의 극단적 압축으로 diffusion 자체는 빠르지만, 품질 비용이 명확합니다 — Image-to-3D BCD $5.64$ vs 본 논문 $2.91$ ($\times 10^{-2}$, 약 2× 악화, Tab.~5), reconstruction CD on GarmageSet $2.19$ vs $0.46$ ($\times 10^{-3}$, 4.7× 악화, Tab.~1). 또한 **fixed per-panel** 구조 때문에 패널 수가 가변인 ABO 등 도메인에는 적용 자체가 불가능합니다 (l.~274–275). 본 방법은 단일 GPU 1초 수준의 latency를 유지하면서 품질·일반성에서 우위를 갖습니다.

---

## 2. Differentiable End-to-End Pipeline이 핵심 기여입니다 (R3.1, R1.2)

**["Differentiable"의 범위는 image가 아니라 mesh까지의 end-to-end입니다.]** 기존 GI 연구(Omages, GIMDiffusion, GarmageNet)는 image-domain pixel loss까지만 differentiable이며, mesh 추출은 non-differentiable post-processing이라 3D-도메인 supervision이 latent까지 도달할 수 없습니다. DMS가 이 gap을 닫음으로써 Normal Rendering Loss를 3D에서 계산하고 VAE encoder/decoder까지 backpropagate할 수 있게 됩니다 (Supp.~Sec.~6: "$\mathcal{L}_\text{Normal}$ can propagate gradients … *only because the DMS module is fully differentiable*"). TSDF + DMS + $\mathcal{L}_\text{Normal}$은 단순 component sum이 아니라 3D end-to-end 학습을 가능케 하는 **결합된** contribution입니다.

**[SD 1.5 init이 성능 향상의 원인이 아닙니다.]** SD 1.5 VAE는 in-the-wild RGB로 학습되어 geometry-specific prior를 갖지 않으며, 단지 학습 수렴 가속의 목적으로 사용했습니다. Tab.~2의 ablation은 **동일한 init**에서 representation만 Occ→TSDF로 교체했을 때 CD $1.503 \rightarrow 0.595$ ($\times 10^{-3}$, $-60\%$), JSD $4.539 \rightarrow 2.169$ ($-52\%$)을 보입니다 — init이 원인이라면 Occ 행도 동등하게 향상되어야 하지만 그렇지 않습니다. 추가로 Normal Loss 결합 시 NC $0.921 \rightarrow 0.961$, JSD $2.169 \rightarrow 1.244$로 더 개선되며, 이는 **DMS 없이는 사용 자체가 불가능한** supervision입니다.

---

## 3. Evaluation Fairness와 Baseline 비교 (R1.1, R3.3)

**[통합 DMS 평가는 공정합니다.]** DMS는 binary occupancy 입력에 대해 **standard Marching Squares와 동등한 결과**를 산출하며 (Supp.~Sec.~3), 어느 baseline에도 추가적인 boundary refinement·chart merging 같은 후처리를 가하지 않습니다. 통합 프로토콜의 목적은 Supp.~Sec.~5에 명시한 대로 *representation·compression의 효과를 mesh-extraction engineering으로부터 분리*하는 것입니다 — 각 baseline의 native pipeline은 서로 다른 boundary heuristic을 포함해 representation 자체의 기여를 가립니다. 본 방법은 이 통합 평가 하에서 ABO의 Omages(CD $0.83$ vs $0.89 \times 10^{-3}$)와 GarmageSet의 GarmageNet(CD $0.46$ vs $2.19 \times 10^{-3}$)을 모두 능가합니다 (Tab.~1).

**[GIMDiffusion 직접 비교 부재 사유.]** GIMDiffusion은 코드·weights·preprocessed dataset이 모두 비공개라 동일 조건의 재현이 불가능했습니다. 다만 GIMDiffusion의 geometry pipeline (binary-mask GI + SD-init VAE + LDM; Collaborative Control은 texture 전용)은 본 논문 ablation의 **"Occ w/o $\mathcal{L}_\text{Normal}$"** 행과 구조적으로 등가입니다. 본 방법(TSDF + $\mathcal{L}_\text{Normal}$)은 이 baseline 대비 CD $1.503 \rightarrow 0.461$ ($\times 10^{-3}$, 3.3×), JSD $4.539 \rightarrow 1.244$ (3.6×)로 개선됩니다 (Tab.~2). 이 관계를 camera-ready에 명시하겠습니다.

---

## 4. Implementation Detail과 Artifact (R1.4, R2.3, R1.3)

- **TSDF 정의 (R1.4):** TSDF는 3D 거리 함수가 아니라 **UV-image 공간의 2D 부호 거리**입니다 (Sec.~3.1, Step~4). 각 픽셀에서 가장 가까운 patch contour까지의 부호화된 Euclidean 거리(chart 내부 양수, 외부 음수)이며 1024×1024 grid 기준 15 px로 truncate됩니다. main text에서 더 명확히 기술하겠습니다.
- **Mesh→DiffGI 변환 (R2.3):** 결정론적·학습 불요·GPU batch-parallel 절차입니다 (Sec.~3.1, Steps~1–4: UV chart 추출 → $1024^2$ grid sampling → 256 downsample → 2D TSDF). camera-ready에 의사코드 추가.
- **Seam (R1.3):** Multi-chart GI 계열의 구조적 한계로 모든 baseline이 공유합니다. 본 방법은 image-to-3D BCD $5.64 \rightarrow 2.91$ ($\times 10^{-2}$, $-48\%$, Tab.~5)로 경계 품질을 정량적으로 크게 개선했으며, 완전 제거는 Sec.~5 Limitations에 명시한 대로 dual contouring 통합을 future work로 남깁니다.

---

## 5. Non-Curated ABO Generation Results (R3.4)

ABO 결과는 이미 main paper Tab.~1, Tab.~4 (label-conditioned P-FID $31.14 \rightarrow 19.91$), Fig.~5에 포함되어 있습니다. cherry-picking 우려를 해소하기 위해 Fig.~1에 두 무작위 sample 간 latent interpolation 결과를 추가했습니다 — interpolation 경로 전 구간에서 globally consistent하고 plausible한 furniture geometry가 생성됨을 확인할 수 있습니다.

<!--
검토용 메모 (LaTeX 변환 시 제거):
- 각 항목 헤더의 (R..., R...) 매핑은 영어 변환 시 그대로 살리는 것을 권장 — 리뷰어가 자기 의견의 응답 위치를 즉시 찾을 수 있음.
- 분량 추정 (영어 직역 후): ~700 단어 + figure 1개. ECCV 10pt 2-column 1쪽에 적정.
- 1쪽 초과 시 축약 우선순위: 4번 [Conversion] 한 줄로 → 5번 한 줄로 → 1번 [Resolution] 마지막 문장 삭제.
- 핵심 보호 (절대 축약 금지): 1번 [Domain coverage], 2번 두 단락 모두, 3번 [Unified DMS], 4번 [Seam] 수치.
-->
