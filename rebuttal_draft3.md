# DiffGI Rebuttal Draft (#12200) 한국어 초안

리뷰어 분들의 검토에 감사드립니다. Camera-ready에 GIMDiffusion 인용 [15]→[11] 정정 및 "multi-chart geometry image" 용어 일관 사용을 반영하겠습니다. [R3 minor]

**[R3] DMS 기반 mesh-까지-end-to-end differentiability가 핵심 기여입니다.** 기존 GI 연구(Omages, GIMDiffusion, GarmageNet)는 image-domain pixel loss까지만 differentiable이며 mesh 추출은 non-differentiable post-processing이어서 3D-도메인 supervision이 latent까지 도달할 수 없습니다. DMS가 이 gap을 닫음으로써 Normal Rendering Loss를 3D에서 계산하고 VAE encoder/decoder까지 backpropagate할 수 있게 됩니다 (Supp.~Sec.~6: "$\mathcal{L}_\text{Normal}$ can propagate gradients $\dots$ *only because the DMS module is fully differentiable*"). Tab.~2가 두 효과를 분리해 입증합니다. Representation 단독(Occ→TSDF)은 CD $1.503 \rightarrow 0.595 \times 10^{-3}$ ($-60\%$)을 보입니다. 추가로 DMS 없이는 사용 불가능한 Normal Loss를 결합하면 NC $0.921 \rightarrow 0.961$, JSD $2.169 \rightarrow 1.244 \times 10^{-3}$로 더 개선됩니다. TSDF + DMS + $\mathcal{L}_\text{Normal}$은 단순한 component sum이 아니라 3D end-to-end 학습을 가능케 하는 **결합된** contribution입니다.

**[R1] SD 1.5 VAE 초기화의 영향 분리.** 본 논문은 학습 수렴 가속을 위해 SD 1.5 VAE를 사용했으나, 이는 in-the-wild RGB로 학습되어 geometry-specific prior를 갖지 않습니다. R1의 우려 — *"성능 향상이 representation 때문인지 더 강력한 VAE 때문인지 분리되지 않는다"* — 에 두 단계로 답합니다. **(i)** Tab.~2 ablation은 **VAE init을 동일하게 고정**한 채 representation만 Occ→TSDF로 변경했을 때 CD $1.503 \rightarrow 0.595 \times 10^{-3}$ ($-60\%$)을 보여, representation의 독립적 기여를 입증합니다. **(ii)** 추가로 from-scratch VAE와 SD-pretrained VAE가 수렴 속도만 다르고 최종 성능은 동등함을 보이는 미니 실험을 진행 중이며, 결과를 supplementary에 포함하겠습니다. <!-- TODO: from-scratch vs pretrained VAE 수렴/최종성능 비교 실험 결과 추가 -->

**[R1] DMS 통합 평가 프로토콜의 근거.** Mesh-extraction 방식이 결과에 영향을 줄 수 있다는 우려에 동의합니다. 이 때문에 모든 baseline에 동일한 추출기(DMS)를 적용했으며, DMS는 binary occupancy 입력에 대해 **standard Marching Squares와 동등한 결과**를 산출하므로 (Supp.~Sec.~3) 어느 baseline에도 구조적으로 불리하지 않습니다. 통합 프로토콜의 목적은 *representation·compression의 효과를 mesh-extraction engineering으로부터 분리*하는 것입니다 (Supp.~Sec.~5) — 각 baseline의 native pipeline은 서로 다른 boundary heuristic을 포함해 representation 자체의 기여를 가립니다. 본 방법은 이 통합 평가 하에서도 Tab.~1의 ABO·GarmageSet 두 도메인 모두에서 baseline 대비 우위를 유지합니다. 추가로, 동일 occupancy 입력에 대해 DMS와 standard MS로 추출한 mesh가 시각·정량적으로 일치함을 보이는 미니 비교를 진행 중이며 supplementary에 포함하겠습니다. <!-- TODO: DMS ≈ standard MS equivalence (occupancy 입력) 미니 실험/시각화 결과 추가 -->

**[R3] GIMDiffusion 직접 비교 부재 사유.** GIMDiffusion은 코드·weights·preprocessed dataset이 모두 비공개라 동일 조건의 재현이 불가능했습니다. 다만 GIMDiffusion의 geometry pipeline (binary-mask GI + SD-init VAE + LDM; Collaborative Control은 texture 전용)은 본 논문 ablation의 **"Occ w/o $\mathcal{L}_\text{Normal}$"** 행과 구조적으로 등가이며, 이 baseline 대비 본 방법(TSDF + $\mathcal{L}_\text{Normal}$)은 CD $1.503 \rightarrow 0.461 \times 10^{-3}$ (3.3×), JSD $4.539 \rightarrow 1.244 \times 10^{-3}$ (3.6×)로 개선됩니다 (Tab.~2).

**[R3] 속도-품질 trade-off와 GarmageNet의 구조적 한계.** GarmageNet은 패널당 72-dim의 극단 압축으로 diffusion 자체는 빠르지만 품질 비용이 명확합니다 — Image-to-3D BCD $5.64$ vs Ours $2.91 \times 10^{-2}$ ($\sim 2\times$ 악화, Tab.~5), reconstruction CD on GarmageSet $2.19$ vs $0.46 \times 10^{-3}$ (4.7× 악화, Tab.~1). Fig.~6 (main paper)에서 GarmageNet은 다수 row에서 구조적 condition mismatch를 보입니다 — garment type 변경 (row 1: skirt→t-shirt; row 4: shorts→skirt), sleeve 유무·길이 불일치, row 5의 형체 결여·비정상 cuff 등. 또한 **fixed per-panel 구조** 때문에 패널 수가 가변인 ABO 등 도메인에는 적용 자체가 불가능합니다 (l.~274–275). 본 방법은 단일 GPU $\sim 1$s latency를 유지하면서 품질·일반성에서 우위를 갖습니다.

**[R2] UV-parameterization 의존성에 대해.** UV 의존은 multi-chart GI 계열의 공통 제약이며 본 방법도 이를 공유합니다. 본 논문이 타겟하는 garment·furniture 도메인에서는 산업 표준 UV가 보편적으로 제공됩니다 (ABO 7,900개 commercial furniture, GarmageSet 14,801개 professionally designed garments). 부수적 이점으로, 잘 정돈된 UV로 학습한 결과 생성된 mesh도 UV가 정돈된 채로 출력되어 simulation·texturing 파이프라인에 직접 활용 가능합니다.

**[R2] Resolution과 latent 압축률.** Supp.~Sec.~1에서 reconstruction error가 해상도 256 이상에서 수렴함을 보였습니다 — 8× 공간 압축한 32×32×4 latent로도 압축 없는 Omages(64×64×4)를 CD/JSD/NC 모든 지표에서 능가하며, 이는 TSDF의 redundant boundary encoding이 latent compression에 robust하기 때문입니다. 더 높은 해상도로의 확장은 latent grid 차원만 늘리면 됩니다.

**[R1] Fig.~6의 seam.** Multi-chart GI 계열의 구조적 한계로 모든 baseline이 공유합니다. 본 방법은 Tab.~5에서 boundary error(BCD)를 정량적으로 크게 개선했으며, 완전 제거는 Sec.~5 Limitations에 명시한 dual contouring 통합을 future work로 남깁니다.

**[R1] TSDF는 3D 거리가 아니라 UV-image 공간의 2D 부호 거리입니다 (l.~139–141).** 각 픽셀에서 가장 가까운 patch contour까지의 부호화된 Euclidean 거리(chart 내부 양수, 외부 음수)이며 $1024 \times 1024$ grid 기준 15 px로 truncate됩니다 (Sec.~3.1, Step~4). main text에서 더 명확히 기술하겠습니다.

**[R3] Non-cherry-picked ABO generation.** ABO 결과는 main paper Tab.~1, Tab.~4, Fig.~5에 이미 포함되어 있습니다. cherry-picking 우려를 해소하기 위해 Fig.~1에 두 무작위 sample 간 latent interpolation 시퀀스를 추가합니다 — 경로 전 구간에서 globally consistent하고 plausible한 furniture geometry가 생성됨을 확인할 수 있습니다.

<!--
검토용 메모 (LaTeX 변환 시 제거):
- ICCV 2023 reference rebuttal 스타일: opener에 minor editorial 모두 bundling, 본문은 [Rx, Ry] 태그 + 굵은 lead-in으로 시작하는 dense 단락 9개.
- 영어 직역 시 ~700–800 단어 + figure 1개. ECCV 1쪽 적정.
- item-based(draft.md)와의 차이: 응답 단위가 "테마"가 아닌 "질문"이므로 reviewer traceability가 더 강함 (Rx 태그 명시). 대신 표면적 응답 수가 많아 (9개) 시각적으로 더 dense함.
- 1쪽 초과 시 축약 우선순위: Resolution → UV → Seam 순으로 한 줄씩 압축.
- 핵심 보호: [R3] differentiable end-to-end (가장 길고 중요), [R1] DMS fairness, [R1] init 분리, [R3] GIMD/GarmageNet 두 개.
-->
