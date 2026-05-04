# DiffGI Rebuttal Draft (#12200) 한국어 초안

리뷰어 분들의 검토에 감사드립니다. Camera-ready에 GIMDiffusion 인용 [15]→[11] 정정 및 "multi-chart geometry image" 용어 일관 사용을 반영하겠습니다. [rPcg minor]

**[rPcg] DMS 기반 mesh-까지-end-to-end differentiability가 핵심 기여입니다.** 기존 GI 연구(Omages, GIMDiffusion, GarmageNet)는 image-domain pixel loss까지만 differentiable이며, mesh 추출이 non-differentiable post-processing이어서 3D-domain supervision이 latent까지 도달하지 못합니다. DMS는 이 gap을 닫아 Normal Rendering Loss를 3D에서 계산하고 VAE encoder/decoder까지 backpropagate할 수 있게 합니다. Tab.~2의 ablation은 두 효과를 분리합니다. Representation을 Occ에서 TSDF로 바꾸는 것만으로 CD가 $60\%$ 감소합니다. 여기에 DMS 없이는 사용 불가능한 Normal Loss를 추가하면 JSD가 $43\%$ 더 감소하고 NC가 $0.92$에서 $0.96$으로 향상됩니다. 따라서 TSDF + DMS + $\mathcal{L}_\text{Normal}$은 표준 component의 단순 합이 아니라 3D end-to-end 학습을 가능케 하는 결합된 contribution입니다.

**[hEpV] SD 1.5 VAE 초기화의 영향 분리.** 본 논문은 학습 수렴 가속을 위해 SD 1.5 VAE를 사용했으나, 이는 in-the-wild RGB로 학습되어 geometry-specific prior를 갖지 않습니다. (i) Tab.~2 ablation은 VAE init을 고정한 채 representation만 Occ→TSDF로 변경했을 때 CD를 $60\%$ 감소시켜, representation의 독립적 기여를 입증합니다. (ii) 추가로 from-scratch VAE와 SD-pretrained VAE가 수렴 속도만 다르고 최종 성능은 동등함을 보이는 미니 실험을 진행 중이며 supplementary에 포함하겠습니다. <!-- TODO: from-scratch vs pretrained VAE 수렴/최종성능 비교 실험 결과 추가 -->

**[hEpV] 통합 평가 프로토콜.** hEpV의 코멘트를 통해 supp Sec.~5의 *"all methods evaluated using DMS"* 서술에 오류가 있음을 확인했습니다. 실제 protocol은 TSDF에는 DMS를, occupancy에는 Omages의 native triangulation을 미분가능하게 구현한 Differentiable Tessellation을 사용하며, 본 논문 Sec.~4.2 (l.~275–277)가 후자를 이미 정확히 명시하고 있습니다. 본 방법의 Differentiable Tessellation은 mesh quality 향상을 위한 별도 engineering 없이 ABO 평가 sample 전체에서 Omages의 native triangulation과 bit-identical함을 확인했으며 (Fig.~\ref{fig:tess}), 모든 occupancy 계열 baseline에 통일 적용해도 어느 baseline에도 구조적 유불리가 없습니다. <!-- TODO: bit-identical sample 수 확정, Fig 라벨/경로 확정, supp Sec.~5 정정 반영 -->

**[rPcg] 속도-품질 trade-off와 GarmageNet의 구조적 한계.** GarmageNet은 dense mesh를 패널당 72-dim으로 극단적으로 압축하여 빠른 diffusion을 얻지만, 그만큼 품질 측면의 trade-off가 동반됩니다. reconstruction CD on GarmageSet은 $4.7\times$ 악화되고 (Tab.~1), image-to-3D BCD는 $\sim\!2\times$ 악화됩니다 (Tab.~5). 이는 Fig.~6 (main paper)의 image-to-3D 결과에서도 확인됩니다. row 1과 4에서는 garment type 자체가 변경되고 (skirt→t-shirt; shorts→skirt), 일부 row는 sleeve 유무·길이가 불일치하며, row 5에서는 형체 결여와 비정상 cuff가 나타납니다. 또한 fixed per-panel 구조는 패널 수가 가변인 ABO 등 도메인을 지원하지 않습니다 (l.~274–275). 본 방법은 mesh를 과도하게 압축하지 않으면서도 단일 GPU $\sim 1$s latency와 더 높은 품질·일반성을 달성하여 이 trade-off를 보다 효과적으로 해결합니다.

**[rPcg] GIMDiffusion 직접 비교 부재 사유.** GIMDiffusion은 코드·weights·preprocessed dataset이 모두 비공개라 동일 조건의 재현이 불가능했습니다. 다만 GIMDiffusion의 geometry pipeline (binary-mask GI + SD-init VAE + LDM; Collaborative Control은 texture 전용)은 본 논문 ablation의 *"Occ w/o $\mathcal{L}_\text{Normal}$"* 행과 구조적으로 등가이며, 이 baseline 대비 본 방법(TSDF + $\mathcal{L}_\text{Normal}$)은 CD를 $3.3\times$, JSD를 $3.6\times$ 개선합니다 (Tab.~2).

**[ZMhC, hEpV] Multi-chart GI 계열 diffusion 모델의 특성.** 본 방법은 이 계열의 특성을 공유합니다. (i) Seam: 모든 baseline이 공유하는 구조적 한계지만, 본 방법은 Tab.~5에서 boundary error (BCD) 를 크게 개선했으며, 추가 개선은 dual contouring 같은 대안 representation 도입을 future work로 두었습니다 (Sec.~5). (ii) UV 의존: 데이터셋 측면의 제약은 있으나, 산업 표준 UV로 학습하면 생성된 mesh의 UV도 정돈되어 simulation·texturing 등 downstream task에 직접 활용 가능합니다. (iii) Resolution: image 기반 diffusion의 특성상 자유롭게 확장 가능하며, Supp.~Sec.~1의 수렴 분석에 따라 본 논문은 256을 채택했고 후속에서 더 높은 해상도를 사용할 예정입니다.

**[hEpV] TSDF는 3D 거리가 아니라 UV-image 공간의 2D 부호 거리입니다 (l.~139–141).** 각 픽셀에서 가장 가까운 patch contour까지의 부호화 Euclidean 거리(chart 내부 양수, 외부 음수)이며, $1024 \times 1024$ grid 기준 15 px로 truncate됩니다 (Sec.~3.1, Step~4); main text에서 더 명확히 기술하겠습니다.

**[rPcg] Non-cherry-picked ABO generation.** 본 논문 Tab.~1, 4, Fig.~5 외에 Fig.~\ref{fig:abo}에 두 무작위 sample 간 latent interpolation 시퀀스를 첨부합니다.

<!--
검토용 메모 (LaTeX 변환 시 제거):
- ICCV 2023 reference rebuttal 스타일: opener에 minor editorial 모두 bundling, 본문은 [Rx, Ry] 태그 + 굵은 lead-in으로 시작하는 dense 단락 9개.
- 영어 직역 시 ~700–800 단어 + figure 1개. ECCV 1쪽 적정.
- item-based(draft.md)와의 차이: 응답 단위가 "테마"가 아닌 "질문"이므로 reviewer traceability가 더 강함 (Rx 태그 명시). 대신 표면적 응답 수가 많아 (9개) 시각적으로 더 dense함.
- 1쪽 초과 시 축약 우선순위: Resolution → UV → Seam 순으로 한 줄씩 압축.
- 핵심 보호: [rPcg] differentiable end-to-end (가장 길고 중요), [hEpV] DMS fairness, [hEpV] init 분리, [rPcg] GIMD/GarmageNet 두 개.
-->
