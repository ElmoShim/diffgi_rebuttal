# Reviewer hEpV

- 모든 method 에서 DMS 로 mesh reconstruction 한 것은 unfair 
- GarmageNEt 은 VAE 를 from scratch 로 학습했다. 본 논문은 SD1.5 pretrained VAE 인데, proposed method 의 효과가 맞는건지: 성능 향상이 DiffGI representation 덕분인지, 아니면 SD1.5 pretrained라는 더 강력한 VAE 덕분인지 구분이 안 된다
- 정성평가에서 Seam 이 보인다.

+ TSDF 계산 방법이 픽셀-표면 거리인지 3D점-표면 거리인지 설명 필요
  
hEpV: Weak Reject (2), Low Confidence


#### My thoughts and responses
- DMS 는 occupancy 든 TSDF 든  pixel 정보를 왜곡없이 그대로 triangle 로 reconstruction 하는 방법이다 (marching square 인데 differentiable 할 뿐임). baseline 논문들에 있는 다양한 system engineering 을 배제하고 순수한 Diffusion 결과물을 fair 하게 비교하기 위해 통일한 것일 뿐이고, 어떤 method 에 유리한 방법이라고 생각하진 않음.
- SD1.5 VAE 는 애초에 in-the-wild RGB 이미지 데이터셋으로 학습한거라서 geometry image 에 대한 어떤 유의미한 prior 를 가지고 있진 않음. 우리는 단지 RGB 채널의 기본적인 reconstruction baseline 을 활용해서 조금 수렴을 빠르게 하기 위해서 이 pre-trained VAE 를 사용했을 뿐이고 크게 성능에 영향은 없음. (실제로 fine-tuning 하지 않은 SD1.5 geometry image reconstruction 결과를 보면 surface 가 지글지글한 것을 볼 수 있음. 필요하면 시각화 자료 첨부 가능함)
- Seam 이 보이는 건 geometry image 기반 method 의 근본적인 한계임. 기존 방법들도 그러한 것들을 해결하기 위해서 boundary snappign 알고리즘을 적용하거나 resolution 을 극단적으로 높이는 등의 방법을 적용했다고 생각하는데 여전히 seam 부분이 noisy 한 결과임, 우리는 TSDF + normal rendering loss 로 좀 더 기술적으로 유려한 방법으로 더 훨씬 개선된 output 을 만들어냈다고 생각함 


# Reviewer ZMhC
- 이 method 자체가 resasonable UV parameterization 을 필요로 한다는 한계가 있음
- low-resolution 이라는 한계가 있음 (detailed geometry 표현 한계)
 
+ implementation detail 이 약함
+ mesh->DiffGI 변환에 걸리는 시간? (per mesh?)
+ parameterization 에 얼마나 민감한지 - UV 가 안 좋은 mesh 가 들어오면 어떻게 되는지?

ZMhC: Weak Accept (5), Moderate Confidence

#### My thoughts and responses
- UV 가 깔끔한 mesh 가 target 이라는 건 팩트임. UV 가 안좋은 mesh 라도 UV 가 정의되어 있기만 하면 기술적으로 이 방법을 적용 가능하긴 함. 하지만 이러한 방법의 진짜 장점은 UV 가 깔끔하게 정리된 mesh dataset 으로 학습함으로써 UV 가 깔끔하게 정리된 mesh 를 생성할 수 있는 그 잠재성에 있다고 생각함. mesh-diffgi round-trip 자체는 UV 가 깔끔하든 안 깔끔하든 기술적으로 빠르게 가능하긴 하지만, 유의미한 UV 의 mesh 를 사용하는 것이 더 유의미한 적용 방법이라고 생각함. UV 가 없는 경우 parameterization 을 하긴 해야겠지만, 그럴 경우 parameterization quality 에 종속적이긴 할 것임

- Resolution 은 추후에 좀 더 높일 생각이 있음. 이미 이미지 기반 method 에서 1024 이상의 high resolution 에 대한 성능이 보장되어 있기도 하고, diffgi 의 resolution 을 높이는 것 자체는 쉽게 가능하기 때문에 promising 한 결과를 보여줄 것으로 기대함. 다만 이 논문에서는 256 이라는 극단적으로 작은 resolution 에서도 3D 의상의 주름까지도 reasonable 하게 보여준다는 점에 집중해서 서술했음.



# Reviewer rPcg
1. 이건 최초의 미분가능한 differentiable geometry-image-based framework 이 아니다 (omages, gimd, garmageNet 등이 존재), 최초로 compressed latent space 를 사용한 것도 아니다 (gimd, garmageNEt 도 LDM 이고, garmageNet 은 더 빠름) - 결국 주요한 novelty 는 TSDF + normal rendering loss 인데, 아주 큰 novelty 라고 볼 수 있을지?

2. 결국 속도에 관련한 이점은 GarmageNet 에 뒤쳐지는 게 아닌지?
3. GiMD 가 가장 관련성이 높은 연구인데 이 연구와 직접적인 비교가 빠진 것은 설명이 필요

 + citation 이 좀 틀린 게 있음
 + 그냥 geometry image 가 아니라, multi-chart geometry image 라는 정확한 표현을 사용할 필요가 있음
 + non-garment 에 대한 결과를 더 보고싶다

rPcg: Borderline Accept (4), High Confidence

#### My thoughts and responses
- 우리가 서술한 "미분가능한" 파이프라인은 mesh 까지 end-to-end 로 미분가능한 것을 말한것임. 우리는 단지 geometry image 를 diffusion 으로 생성한 게 아니라, 이걸 DMS 적용해서 3D 까지 미분가능하게 만들었다는 것에 의미를 두었음. 그렇기 때문에 3D 기반으로 normal rendering loss 를 적용할 수 있었고, 이 end-to-end pipeline 을 3D 로 설계한 것이 본래 목적인 3D 생성에서 큰 성능향상을 보여주었다고 해석하고 있음. baseline 논문들인 omages, gimd, garmagenet 모두 geometry image representation 을 사용하긴 했지만, 그냥 이미지 기반 pixel loss 만으로 학습되었기 때문에 differentiable 3D pipeline 이라고까지는 보기 힘들다고 생각하고, 이 지점에서 성능 차이가 났다고 생각함.

- GarmageNet 은 극단적으로 높은 resolution 을 극단적으로 낮은 latent 로 압축했음. 압축률이 아주 높기 때문에 diffusion 자체의 속도는 빠르긴 하지만, 이로 인한 성능 trade-off 가 분명하게 드러나고 있음. 실제로 image condition 을 제대로 따라가지 못하기도 하고, 높은 resolution 에 비해서 3D 형상 자체가 globally, locally 다 깨져있는 경우가 너무 많이 드러남 - 다만 garmageNEt 연구는 이게 다가 아니고 이후 system engineering 으로 어느정도 해결하긴 하지만, 우리는 diffusion model 만 비교했음.

- GiMD 가 우리가 생각해도 가장 연관성이 높은 연구라고 생각했지만, 코드나 weight, Dataset 이 모두 공개되어 있지 않은 상태라서 직접적인 비교가 쉽지 않았음. 사실 우리의 main contribution 인 TSDF + DMS + Normal Rendering Loss 를 제외하면 GiMD 와 거의 동일한 세팅이라고 생각하기 때문에 Ablation Study 의 결과가 GiMD 와 간접적으로 비교해준다고 생각하긴 함. - 물론 gimd 에서는 collaborative control 을 사용한 부분이 있긴 하지만 이는 texture 생성에 특화된 method 임. 우리는 geometry 생성에 특화해서 비교했기 때문에 ablation 결과로 갈음할 수 있지 않을까 싶긴 함. 아무튼 물리적으로 직접 비교할 수는 없었기 떄문에 직접적으로 비교한 것 처럼 논문에 서술하긴 애매했음.

- ABO 에 대한 결과들도 첨부가 되어있는데 부족한건지?
