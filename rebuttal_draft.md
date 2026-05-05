# DiffGI Rebuttal Draft (#12200) 한국어 초안

리뷰어 분들의 검토에 감사드립니다. Camera-ready에 GIMDiffusion 인용 [15]→[11] 정정, "multi-chart geometry image" 용어 일관 사용, 그리고 TSDF (본 논문 l.~139–141)의 정의 — UV 공간에서 각 픽셀에서 가장 가까운 patch contour까지의 부호화 2D 거리 (chart 내부 양수, 외부 음수; $1024 \times 1024$ grid에서 15 px로 truncate) — 를 명시할 예정입니다.

**[rPcg] DMS 기반 mesh-까지-end-to-end differentiability가 핵심 기여입니다.** 기존 GI 연구는 mesh 추출이 non-differentiable이어서 3D-domain supervision이 latent까지 도달하지 못합니다. DMS는 이 gap을 닫아 Normal Rendering Loss를 3D에서 계산하고 VAE까지 backpropagate할 수 있게 합니다. 본 논문 Tab.~2에서 DMS 없이는 사용 불가능한 Normal Loss를 추가하면 JSD가 $43\%$ 감소하고 NC가 $0.92\to 0.96$로 향상됩니다. DMS가 mesh 단위 end-to-end supervision을 가능케 한 결과입니다.

**[hEpV] SD 1.5 VAE 사용에 대해.** GarmageNet의 VAE는 panel을 64-dim 단일 벡터로 극단 압축하는 panel-wise 구조라, 표준 image VAE와 구조적으로 호환되지 않습니다. 본 방법의 multi-chart GI는 standard image 형태라 SD 1.5와 동일한 VAE 구조를 그대로 사용할 수 있고, GIMDiffusion과 마찬가지로 pretrained weight 활용의 이득을 취했습니다. 다만 이는 단지 수렴 가속 수단입니다. Fig.~\ref{fig:vae}에서 from-scratch VAE도 동등 capacity로 수렴하나 훨씬 느리고 불안정함을 확인했습니다. 한편 본 논문 Tab.~2의 ablation은 VAE init을 고정한 채 representation만 Occ→TSDF로 바꿔도 reconstruction이 크게 개선됨을 보여, representation의 기여가 init과 독립적임을 이미 입증하고 있습니다.

**[hEpV] 통합 평가 프로토콜.** 본 논문 Supp.~Sec.~5의 *"all methods evaluated using DMS"* 서술은 부정확합니다 (camera-ready 정정). 실제 protocol은 TSDF에는 DMS를, occupancy에는 Omages의 native triangulation을 미분가능하게 구현한 Differentiable Tessellation을 사용하며, 본 논문 Sec.~4.2 (l.~275–277)가 후자를 이미 명시하고 있습니다. 본 방법의 Differentiable Tessellation은 Omages의 native triangulation과 bit-identical함을 확인했습니다 (Fig.~\ref{fig:tess}). Mesh quality 향상을 위한 별도 engineering이 없는 단순 tessellation이므로, geometry image 기반 3D 형상 평가에 어느 baseline에도 구조적 유불리를 도입하지 않습니다.

**[rPcg] 속도-품질 trade-off와 GarmageNet의 구조적 한계.** GarmageNet은 dense mesh를 패널당 72-dim으로 극단적으로 압축하여 빠른 diffusion을 얻지만, 그만큼 품질 측면의 trade-off가 동반됩니다. reconstruction CD on GarmageSet은 $4.7\times$ 악화되고 (본 논문 Tab.~1), image-to-3D BCD는 $\sim\!2\times$ 악화됩니다 (Tab.~5). 본 논문 Fig.~6의 image-to-3D 결과에서도 garment type 변경, sleeve 불일치, 형체 결여 등 row별 실패가 관측됩니다. 또한 fixed per-panel 구조는 패널 수가 가변인 ABO 등 도메인을 지원하지 않습니다 (l.~274–275). 본 방법은 mesh를 과도하게 압축하지 않으면서도 단일 GPU $\sim 1$s latency와 더 높은 품질·일반성을 달성하여 이 trade-off를 보다 효과적으로 해결합니다.

**[rPcg] GIMDiffusion 직접 비교 부재 사유.** GIMDiffusion은 코드·weights·preprocessed dataset이 모두 비공개라 동일 조건의 재현이 불가능했습니다. 다만 GIMDiffusion의 geometry pipeline (binary-mask GI + SD-init VAE + LDM; Collaborative Control은 texture 전용)은 본 논문 ablation의 *"Occ w/o $\mathcal{L}_\text{Normal}$"* 행과 구조적으로 등가이며, 이 baseline 대비 본 방법(TSDF + $\mathcal{L}_\text{Normal}$)은 CD를 $3.3\times$, JSD를 $3.6\times$ 개선합니다 (본 논문 Tab.~2).

**[ZMhC, hEpV] Multi-chart GI 계열 diffusion 모델의 특성.** 본 방법은 이 계열의 특성을 공유합니다. (i) Seam: 모든 baseline이 공유하는 구조적 한계지만, 본 방법은 본 논문 Tab.~5에서 boundary error (BCD) 를 크게 개선했으며, 추가 개선은 dual contouring 같은 대안 representation 도입을 future work로 두었습니다 (본 논문 Sec.~5). (ii) UV 의존: 산업 표준 UV로 학습하면 생성된 mesh의 UV도 정돈되어 simulation·texturing 등 downstream에 활용 가능합니다. (iii) Resolution: image 기반 diffusion이라 자유롭게 확장 가능하며, 본 논문은 Supp.~Sec.~1의 수렴 분석에 따라 256을 채택했습니다.

**[rPcg] Non-cherry-picked ABO generation.** Fig.~\ref{fig:abo}에 두 무작위 sample 간 latent interpolation 시퀀스를 첨부합니다.

<!--
검토용 메모 (LaTeX 변환 시 제거):
- ICCV 2023 reference rebuttal 스타일: opener에 minor editorial 모두 bundling, 본문은 [Rx, Ry] 태그 + 굵은 lead-in으로 시작하는 dense 단락 9개.
- 영어 직역 시 ~700–800 단어 + figure 1개. ECCV 1쪽 적정.
- item-based(draft.md)와의 차이: 응답 단위가 "테마"가 아닌 "질문"이므로 reviewer traceability가 더 강함 (Rx 태그 명시). 대신 표면적 응답 수가 많아 (9개) 시각적으로 더 dense함.
- 1쪽 초과 시 축약 우선순위: Resolution → UV → Seam 순으로 한 줄씩 압축.
- 핵심 보호: [rPcg] differentiable end-to-end (가장 길고 중요), [hEpV] DMS fairness, [hEpV] init 분리, [rPcg] GIMD/GarmageNet 두 개.
-->
