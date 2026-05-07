### Reconstruction Method 비교

- DMS : TSDF recon 할 때 사용
- Omages : occupacny recon 할 때 사용
- GarmageNet reconstruction : Garmagenet 에서 사용한 reconstruction method, B-spline boundary extraction + pointcloud sampling + Delaunay triangulation

garmagenet 의 경우, 논문에서 제안된 reconstruction method 를 사용하면 boundary 가 좀 정리되고, surface smoothing 되는 효과가 있지만, processing 시간이 크게 늘어남 (x360 times)

method     | latency 
omages     | ~14ms
garmagenet | ~5.1s