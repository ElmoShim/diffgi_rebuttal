Dear Authors of Submission #12200,

Thanks to the efforts of all reviewers, emergency reviewers, ACs, and SACs, preliminary reviews are now available. Below you will find the preliminary reviews for your ECCV 2026 submission, DiffGI: Differentiable Geometry Images for High-Fidelity Thin-Shell 3D Generation (#12200).

## Reviews

You can access the reviews for your submission by clicking on the corresponding submission in OpenReview: https://openreview.net/group?id=thecvf.com/ECCV/2026/Conference/Authors

A very small number of papers received two reviews. We are still chasing the remaining reviews and will keep the impacted authors posted.

A small number of papers received more than three reviews. If your paper is one of these, it is simply due to the Area Chair’s strenuous efforts in securing emergency reviewers and does not indicate anything special about your paper.

In a few exceptional cases, papers received five or more reviews. **We'll reach out to those authors separately.**

### Rebuttal

- If you have questions or concerns regarding the reviews, we encourage you to address them in your rebuttal, where they will be carefully considered by the reviewers and Area Chairs. To ensure a fair and consistent process, the rebuttal is the appropriate channel for such discussions. **The Program Chairs will generally not respond to individual inquiries about specific reviews at this stage**, except in cases involving conflicts of interest or ethical concerns.

- If you wish to respond to the reviews, you may upload a one-page PDF document using the template (rebuttal.tex) that can be found in the ECCV Author Kit (https://www.overleaf.com/project/696680793124a7ce34e9739e). The entire rebuttal must fit on one page, including any references. We also removed the need for a title in the template to give you more space. To upload your rebuttal, click the “Rebuttal” button on the page of the paper, upload the PDF, and submit the form.

- **Rebuttals longer than one page will not be reviewed**. The same goes for rebuttals, where the margins and formatting are deemed to have been altered from the template.

- The rebuttal **CANNOT** include external links to videos, code repositories, etc (anonymous or otherwise).

- The rebuttal must maintain anonymity.

- The goal of the rebuttal is to refute any factual errors in reviews or to supply additional information or clarifications requested by the reviewers. Rebuttals may include minor additional experiments or analysis requested by reviewers. They may also include figures, graphs, or proofs to better illustrate your arguments. Rebuttals **MUST NOT** add new contributions (theorems, algorithms, experiments) that were absent in the original submission and were not specifically requested by the reviewers. We allow reviewers to request small experiments that could be reasonably run during the rebuttal phase with academic resources. If reviewers request experiments that require too many resources or are otherwise impossible to run, the authors should explain why they are not feasible. Authors should refrain from including significant new experimental results in the rebuttal not specifically requested by the reviewers.

### Confidential Comment to AC

- In addition to the PDF, the rebuttal submission form includes an optional box for authors to write a confidential comment to the Area Chair, to address any significant concerns related to policies, ethics, etc. Please do so only in exceptional circumstances. Note that overly lengthy or charged comments are unlikely to benefit the outcome of your paper.

### Withdrawal

If you wish to withdraw your paper after reading the reviews, you may do so by pressing the “Withdraw” button on the paper’s page, confirming the withdrawal, and submitting the form.

Best,
Paolo Favaro, Zuzana Kukelova, Atsuto Maki, Anna Rohrbach, Konrad Schindler, Federico Tombari
ECCV 2026 Program Chairs

----- 

# Reviewer hEpV
#### Title
Review

#### Choose The Contribution Type You Are Using For Your Review
Algorithms/General

#### Justify The Above Choice
I agree

#### Paper Summary
The paper aims for a 3D surface representation for reconstruction and generation, especially for non-watertight meshes. The proposed representation, DiffGI, consists of 3D position map and a truncated SDF map unwrapped in the 2D space. The key difference between this representation with previous works is the truncated SDF map, while previous works usually use occupancy map (0 or 1 indicating "valid" or not). Additionally, it proposes a differentiable marching cube algorithm that converts DiffGI to a 3D mesh. Therefore, the representation can be supervised by losses applied to the mesh, e.g., the normal loss.

It experiments on 3D garments dataset and 3D object dataset, showing the quality of the reconstruction and generation with the proposed representation as an intermediate step.

#### Paper Strengths
- A differentiable representation is essential for 3D reconstruction and generation, as it enables the application of loss functions directly to both the mesh and the rendered images.

- The proposed method is simple and it shows good results in the experiments.

- It conducts experiments on both 3D garments and general 3D objects, with garments presenting additional challenges due to their non-watertight geometry.

#### Major Weaknesses
- My biggest concern is about the fairness compared to the baselines. It mentions in Sec.5 of Supplementary that all methods are evaluated with the differentiable marching squares algorithm instead of their native baselines and claims that it is for fair comparison on the representation side. However, the way to extract meshes is highly coupled with the choice of 3D representation. And the baseline methods may not design for DMS. Without comparing to the original pipeline, it's hard to evaluate the effectiveness of the method.
- Besides, the comparisons on 3D reconstruction are not fair as well. While the focus is on the 3D representation, Tab.1 does not keep other parts identical for fair comparison. For example, GarmageNet uses a U-Net-based VAE and trains it from scratch. However, this method uses a more powerful VAE, pretrained on Stable Diffusion 1.5. It is difficult to disentangle the representation from the architecture, making it challenging to fairly evaluate the method.
- In the qualitative results, the mesh from the proposed method has seams (Fig. 6). For example, there is a seam on the dress between the upper part and the bottom part. Such errors in the geometry introduce extra challenges to physical simulation.

#### Minor Weaknesses
How is the TSDF map computed? Since pixels outside the image boundary are not associated with any 3D position, how is the distance defined in those regions? Specifically, is the TSDF value the distance from a given 3D point to its closest surface, or the distance between a pixel and the nearest surface pixel? Lines 139–141 are unclear to me.

#### Preliminary Recommendation
2: Weak Reject

#### Justification Of Preliminary Recommendation
My main concern is the unfair comparisons. Without fair comparisons, it's hard to evaluate the effectiveness of the method.

#### Suggestions For Rebuttal
I hope that the authors can clarify them during the rebuttal and meanwhile provide the fair comparisons.

#### Ethics Review Flag
No

#### Confidence Level
2: Low Confidence - The reviewer has some familiarity with the topic but does not have deep expertise. They might understand the paper but feel unsure about their evaluation.


-----------
# Reviewer ZMhC
#### Title
Interesting method

#### Choose The Contribution Type You Are Using For Your Review
Algorithms/General

#### Justify The Above Choice
I agree

#### Paper Summary
This paper introduces an end-to-end 3D-to-2D DiffGI framework for representing and generating surfaces in 2D UV spaces. At the core of the proposed method is a DiffGI-VAE that maps sampled 2D truncated signed distance function (TSDF) values to latent codes in a lower-dimensional space.

The key contribution of this paper is the proposed DiffGI framework.

#### Paper Strengths
- The paper tackles the generation of surfaces---an important problem with a wide array of applications.

- The proposed method is overall novel, and the idea of operating in the UV space is interesting.

- The proposed method produces higher-quality surface geometries than the baseline methods.

#### Major Weaknesses
- The mesh-to-DiffGI conversion step seems to require meshes with reasonable UV parameterization (e.g., no overlapping), which could limit the amount of feasible data.

- The proposed method operates on low-resolution images/latent codes, which seems to limit its ability to capture detailed geometric structures.

#### Minor Weaknesses
The paper lacks implementation details, making it nontrivial to reproduce.

#### Preliminary Recommendation
5: Weak Accept

#### Justification Of Preliminary Recommendation
This paper presents a new representation for the generation of surface geometries. By operating in the UV space, the proposed method is capable of producing higher-quality results compared with the existing methods.

Overall, I think the proposed method is novel and offers practical value (despite potential limitations). Thus, I lean toward acceptance.

#### Suggestions For Rebuttal
Providing more details on the mesh-to-DiffGI step (e.g., how long it takes per mesh, how sensitive it is to the UV parameterization of input meshes) would be helpful.

#### Ethics Review Flag
No

#### Confidence Level
3: Moderate Confidence - The reviewer is reasonably knowledgeable about the topic. They understand the paper's methodology and results but may not be a leading expert in the specific subfield.

# Reviewer rPcg
#### Title
Overall a good paper but with some doubts on scope of the novelty, efficiency claim, and evaluation

#### Choose The Contribution Type You Are Using For Your Review
Algorithms/General

#### Justify The Above Choice
I agree.

#### Paper Summary
DiffGI addresses a specific and well-motivated problem for multi-chart geometry image representations of 3D surfaces. Specifically, prior works rely on binary occupancy maps to encode which pixels on a 2D UV grid belong to the surface, but these binary maps are resolution-limited and produce staircase artifacts at mesh boundaries. The paper's core technical contribution is replacing binary occupancy with a continuous 2D Truncated Signed Distance Function (TSDF) and introducing a Differentiable Marching Squares (DMS) algorithm with a normal rendering loss, operating on a compact latent space.

The method is evaluated mainly on garment datasets, showing improved boundary precision, better preservation of thin-shell and non-manifold geometry, and significantly lower computational cost compared to volumetric baselines like TRELLIS and geometry-image baselines like Omages, with the entire image-conditioned generation pipeline running in about one second on a consumer GPU.

#### Paper Strengths
1. Continuous surface representation with a 2D TSDF-based geometry image framework replacing binary occupancy maps.This leads to improved boundary handling and quality in reconstruction and generation.

2. Fully differentiable surface reconstruction using a tensor-based differentiable marching squares (DMS) algorithm.

3. A compressed latent space, leading to high efficiency, especially for non-watertight, open, and even non-manifold surfaces, such as those in garment data.

4. Clear qualitative improvements on garment data for both reconstruction and generation.

#### Major Weaknesses
1. I have some doubts over the scope of the novelty. First, DiffGI is not the first differentiable geometry-image-based framework for the tackled tasks, even though prior works such as Omages, GIMDiffusion [ICLR 2025], and GarmageNet all used binary occupancy maps. Also, it is not the first work that used a compressed latent space in this context, as GarmageNet also used a latent diffusion model and is apparently faster (see Table 3). So the main novelties are the TSDF and normal rendering losses (see the difference in ablation studies), but these are somewhat standard ideas and are only novel in the specific context of the problem setting.

2. The third claimed contribution of speed is compromised by GarmageNet.

3. Missing direct comparisons to GIMDiffusion [ICLR 2025] should be explained. It seems highly relevant.

#### Minor Weaknesses
The citation to GiMD [15] in the introduction is probably wrong. It points to the original GI paper from 2002 by Gu et al. Isn't this supposed to cite the ICLR 2025 paper on Geometry Image Diffusion [11]?

The paper should use the term "multi-chart" to distinguish what it is doing from the original geometry images of Gu et al. In that original paper, there is no gap in the geometry images, while handling these gaps using either binary occupancies or the 2D TSDFs is a key.

#### Preliminary Recommendation
4: Borderline Accept

#### Justification Of Preliminary Recommendation
I am leaning slightly positive mainly due to the improved results that were well demonstrated.

#### Suggestions For Rebuttal
See points raised under weaknesses sections. Also, I wonder how "garment-specific" the improvements are. Some visual results (not cherry-picked, I hope) on ABO would be informative.

#### Ethics Review Flag
No

#### Confidence Level
4: High Confidence - The reviewer has strong expertise in the area. They are highly familiar with the relevant literature and can critically evaluate the paper.