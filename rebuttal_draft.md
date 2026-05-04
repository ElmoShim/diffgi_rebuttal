# Rebuttal 초안 (한글)

---

## Reviewer hEpV (Weak Reject, Low Confidence)

---

### [R1-W1] 모든 baseline에 DMS를 적용한 것은 unfair

**리뷰어 주장:** baseline 논문들은 DMS를 위해 설계된 게 아니므로, DMS로 통일 평가하는 것은 불공정하다.

> **[실험 placeholder] Omages native reconstruction vs. DMS 적용 결과 비교**
> - Omages의 native reconstruction pipeline output과, 동일한 Omages representation에 DMS를 적용한 output의 정량 비교 (CD, NC 등)
> - 정성 비교 (시각화 2~3개 샘플)

DMS를 binary occupancy에 적용하면 standard Marching Squares reconstruction과 동일한 mesh를 산출함 — 즉 Omages가 native pipeline에서 얻는 결과와 동일함. 두 결과가 거의 동일하다면 이를 직접 증명할 수 있음. 또한 Supplementary Sec. 5에 명시된 대로, DMS 통일 평가의 목적은 representation과 compression 효과를 mesh extraction engineering으로부터 분리하기 위함임 — 각 method의 native pipeline을 사용하면 서로 다른 boundary snapping, chart merging 등의 system engineering이 혼입되어 representation 자체의 효과를 분리하기 더 어려워짐.

---

### [R1-W2] SD1.5 pretrained VAE vs. GarmageNet from-scratch VAE — 불공정한 비교

**리뷰어 주장:** GarmageNet은 VAE를 from scratch로 학습, 본 논문은 SD1.5 pretrained VAE 사용 → 성능 향상이 representation 때문인지, stronger VAE 때문인지 불분명.

> **[실험 placeholder] From-scratch VAE 학습 결과 비교**
> - 동일한 아키텍처로, SD1.5 pretrained init 없이 random init으로 VAE를 학습
> - 학습 loss 수렴 curve 비교 (pretrained init vs. random init)
> - 최종 reconstruction 성능 비교 table (CD, NC 등)

SD1.5 VAE는 in-the-wild RGB 이미지로 학습된 것으로 geometry image에 대한 geometry-specific prior를 갖고 있지 않음. Ablation (Table 2)에서도, 동일한 SD1.5 init 아래 representation만 TSDF로 바꿨을 때 CD가 60% 감소 (Occ 1.503 → TSDF 0.595)함. 만약 성능 향상이 SD1.5 init 덕분이었다면 representation에 무관하게 Occ 결과도 좋아야 하지만 그렇지 않음.

---

### [R1-W3] 정성평가에서 Seam이 보임 (Fig. 6)

Seam은 multi-chart geometry image 방식 자체의 구조적 한계로, 기존 방법들도 완전히 해결하지 못하고 있음. 본 논문은 TSDF + DMS + Normal Rendering Loss의 조합으로 기존 대비 boundary quality를 정량적으로 크게 개선함:
- GarmageNet BCD: 5.64 × 10⁻²
- Ours BCD: **2.91 × 10⁻²** (약 48% 개선)

완전한 seam 제거는 목표로 하지 않았으며 Limitations에 명시. 미래 작업으로 dual contouring integration을 계획 중 (Conclusion 참조).

---

### [R1-minor] TSDF 계산 방법 불명확 (Lines 139–141)

**리뷰어 질문:** TSDF 값이 "3D 점 → 표면 거리"인지, "픽셀 → 가장 가까운 표면 픽셀 거리"인지 불명확.

**2D UV 이미지 공간에서의 거리**임 (논문 Sec. 3.1 Step 4, Supplementary Sec. 7):
- 각 픽셀로부터 가장 가까운 UV chart contour (patch 경계선)까지의 2D Euclidean distance
- Positive = UV chart 내부 픽셀, Negative = chart 외부 (background) 픽셀
- 3D 공간 거리가 아님. truncation은 1024×1024 기준 15 pixels

---

## Reviewer ZMhC (Weak Accept, Moderate Confidence)

---

### [R2-W1] Reasonable UV parameterization이 필요하다는 한계

UV 의존성은 본 방법 고유의 결함이 아니라 GI-family 전반의 intrinsic 속성임 (Omages, GiMD, GarmageNet 모두 동일). 본 논문이 타겟하는 garment / furniture 3D assets는 산업적으로 UV가 잘 정의된 경우가 대부분 (물리 시뮬레이션, 텍스처링, 게임 자산 등). 학습 데이터인 ABO (7,900개 상업용 furniture), GarmageSet (14,801개 professionally designed garments) 모두 고품질 UV를 포함.

오히려 UV가 잘 정의된 데이터로 학습함으로써 UV가 잘 정리된 mesh를 생성할 수 있음 — physics simulation, material editing 등 downstream application에서 직접 활용 가능한 강점.

UV가 없는 mesh에도 LSCM, ARAP 등 parameterization 알고리즘으로 전처리 후 적용 가능하나, parameterization quality에 종속적임.

---

### [R2-W2] Low-resolution 한계 (256 → 32×32 latent)

Supplementary Sec. 1에서 확인: 256은 TSDF 기반에서 reconstruction fidelity 수렴에 충분함 ("Beyond resolution 256, both representations converge to near-identical error"). 또한 256×256 → 32×32×4 (8× 공간 압축)에도 불구하고 CD/EMD/JSD 모두 Omages (64×64, VAE 없음)를 능가하는데, 이는 TSDF의 redundant boundary encoding이 압축에 robust하기 때문임 (Supplementary Sec. 1).

Resolution 확장은 기술적으로 어렵지 않으며, 64×64×4 latent로의 확장이 straightforward함.

---

### [R2-minor] mesh-to-DiffGI 변환 시간 및 UV 민감도

> **[실험 placeholder] Mesh-to-DiffGI 변환 시간 측정**
> - ABO / GarmageSet 데이터에 대해 per-mesh 변환 시간 측정 (평균, std)

> **[실험 placeholder] UV parameterization 품질에 따른 민감도**
> - 고품질 UV / 저품질 UV (자동 parameterization 결과) 예시에 대한 reconstruction 결과 시각화

---

## Reviewer rPcg (Borderline Accept, High Confidence)

---

### [R3-W1] 최초의 differentiable geometry-image framework가 아님 + novelty 의문

**리뷰어 주장:** Omages, GiMD, GarmageNet이 이미 존재하며, 주요 novelty는 TSDF + normal rendering loss뿐인데 이것이 충분한 novelty인가?

기존 방법들 (Omages, GiMD, GarmageNet)에서 diffusion 자체는 differentiable하지만, **geometry image → 3D mesh 변환 (mesh extraction)이 non-differentiable post-processing**임. 따라서 3D surface loss를 2D latent space까지 backpropagation하는 것이 구조적으로 불가능함.

본 논문의 DMS가 이 gap을 제거함으로써 Normal Rendering Loss를 3D에서 계산하고 gradient를 2D latent까지 propagate할 수 있게 됨 (Supplementary Sec. 6):
> "LNormal can propagate gradients to the VAE encoder and decoder **only because the DMS module is fully differentiable**. Without differentiable surface extraction, mesh reconstruction would be a non-differentiable post-processing step, blocking any 3D-space supervision from reaching the network weights."

Ablation (Table 2)에서 정량적으로 확인:
- TSDF w/o NL: CD 0.595 → TSDF w/ NL (Ours): CD 0.461, NC 0.921 → 0.961
- NL 자체가 DMS 없이는 사용 불가능함

우리가 최초로 주장하는 것은 "differentiable geometry image generation"이 아니라, "geometry image → 3D mesh까지 완전히 differentiable한 end-to-end pipeline"임. 이 차이가 성능 향상의 근본 원인.

---

### [R3-W2] 속도 이점이 GarmageNet에 뒤쳐지지 않는가?

LDM은 구조적으로 latent 압축률과 생성 품질 사이의 trade-off가 존재함. GarmageNet은 high-quality 3D mesh target을 72-dim이라는 극단적으로 작은 latent로 압축했고, 결과적으로 diffusion 속도는 빠르지만 생성된 3D geometry의 quality가 크게 저하됨 — boundary aliasing 및 globally/locally broken geometry가 Fig. 6에서 뚜렷하게 드러남 (BCD: GarmageNet 5.64×10⁻² vs. Ours 2.91×10⁻²). 또한 GarmageNet은 fixed panel count 구조 때문에 ABO 데이터셋에 아예 적용 불가 (논문 l.274–275)임.

---

### [R3-W3] GiMD와의 직접 비교 누락

GiMD는 코드, model weights, preprocessed dataset이 모두 공개되어 있지 않아 동일한 조건에서의 직접 재현 및 비교가 불가능했음.

본 논문의 Ablation이 간접 비교 역할을 함: GiMD의 geometry 부분은 binary mask 기반 geometry image + VAE (SD1.5 init) + diffusion으로, Collaborative Control은 texture 생성 전용이며 geometry pipeline 자체는 우리 ablation의 "Occ w/o NL" 행과 등가임. 본 방법 (TSDF + NL)은 이 구성 대비 CD 1.503 → 0.461 (3.3×), JSD 4.539 → 1.244 (3.6×) 개선함.

GiMD와의 관계를 논문에서 더 명확히 서술하겠음.

---

### [R3-minor] non-garment(ABO) 결과 추가 요청

논문에 ABO 결과가 이미 포함되어 있음 (Table 1, Table 4, Fig. 5). Rebuttal에 추가로 랜덤 샘플 기반의 ABO generation visual results를 첨부하겠음.

> **[figures/abo_interpolation.pdf]**
> 캡션 예시: "To demonstrate generation quality without cherry-picking, we provide latent interpolation results on ABO rather than individual samples."

---

### [R3-minor] Citation 오류

확인함. Introduction에서 GiMD를 언급할 때 [15] (Gu et al. 2002) 대신 [11] (GIMDiffusion, ICLR 2025)로 수정하겠음.

---

### [R3-minor] "multi-chart geometry image" 용어 사용

타당한 지적. Camera-ready에서 "geometry image" → "multi-chart geometry image"로 일관되게 수정하겠음.

---

## 전략 메모

### 우선순위
1. **rPcg** 최우선 (High Confidence + Borderline Accept)
2. **hEpV** 차순위 (Weak Reject — R1-W1, R1-W2 실험이 핵심)
3. **ZMhC** 이미 Weak Accept이므로 간략하게

### 진행해야 할 실험 목록
| 항목 | 목적 | 예상 소요 |
|------|------|----------|
| Omages native vs. DMS output 비교 | R1-W1 반박 | 낮음 (기존 코드로 가능) |
| From-scratch VAE 학습 | R1-W2 반박 | 높음 (재학습 필요) |
| Mesh-to-DiffGI 변환 시간 측정 | R2-minor | 낮음 (timing 코드) |
| UV 민감도 시각화 | R2-minor | 낮음 (샘플 선별) |
| ABO 추가 generation 결과 | R3-minor | 낮음 (기존 모델로 생성) |

### One-page 분량 전략
ECCV rebuttal은 1 page PDF로 매우 제한적. 공간 배분:
- rPcg W1 (novelty 명확화): 가장 많은 공간
- hEpV W1/W2 (실험 결과): 그 다음
- 나머지: bullet point로 압축
- Figure 1개 (from-scratch VAE 비교 또는 DMS equivalence 실험이 가장 임팩트 있음)
