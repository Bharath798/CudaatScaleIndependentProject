# CUDA at Scale Independent Project

This repository contains my final project for **“CUDA at Scale Independent Project”** course.  

## 📖 Overview  

The project is inspired by the work of **Chaudhury et al.**:  
[Fast and Provably Accurate Bilateral Filtering](https://paperswithcode.com/paper/fast-and-provably-accurate-bilateral/review/).  
The original MATLAB implementation can be found [here](https://www.mathworks.com/matlabcentral/fileexchange/56158-fast-and-accurate-bilateral-filtering).  

I implemented **four different versions** of the Gaussian-Polynomial Approximate Bilateral Filter (GPA-BF):  

1. **NPPI-based implementation**  
2. **Custom CUDA kernel implementation**  
3. **OpenCV (CPU + OpenCL via TAPI)**  
4. **OpenCV CUDA implementation**  

The OpenCV versions serve as a benchmark since OpenCV is widely used and industry-validated. Each implementation supports **1–4 channels** and multiple data types:  
- `unsigned char`  
- `unsigned short`  
- `short`  
- `float32`  

These are the only data types supported for convolution in NPPI, OpenCV, and CUDA.  

---

## ⚙️ Implementation Notes  

- **NPPI**  
  - The most challenging implementation due to limited documentation and inconsistencies in function APIs.  
  - Variance calculation for Gaussian kernels (below 17×17) required reverse-engineering.  
  - Despite hurdles, NPPI exploration was highly insightful.  

- **CUDA**  
  - More straightforward than NPPI.  
  - Implemented a **separable filtering class** that allocates buffers once, avoiding repeated allocations and improving performance.  
  - Future work could explore using **texture memory** to reduce global memory transfers.  

- **OpenCV (CPU, OpenCL, CUDA)**  
  - Useful for result comparison and performance benchmarking.  
  - OpenCL support is integrated via OpenCV’s **Transparent API (TAPI)**, offering CPU, OpenCL, and CUDA backends.  

---

## 📂 Project Structure  

```
cudaGPA/              # Custom CUDA implementation → libcuda_gpabf.so + cuda_gpabf.hpp
neif/                 # NPPI implementation → libnppi_extra_image_filtering.so + nppi_extra_image_filtering.hpp
samples_gpa_lib/      # Unified wrapper library with OpenCV + CUDA + NPPI backends
samples/              # Demo executables (demo1, demo2)
bin/                  # Generated binaries, headers, and libraries
```

### Generated Artifacts  
- **Libraries:**  
  - `libcuda_gpabf.so`, `libnppi_extra_image_filtering.so`, `libgpa.so`  
- **Headers:**  
  - `cuda_gpabf.hpp`, `nppi_extra_image_filtering.hpp`, `gpa.hpp`  
- **Executables:**  
  - `demo1` → Single-image test & comparison  
  - `demo2` → Batch processing on multiple images  

---

## ▶️ Demo Usage  

### `demo1` — Single Image Test
Applies GPA-BF on one image and compares all five implementations (OpenCV CPU, OpenCV-OpenCL, OpenCV-CUDA, NPPI, custom CUDA). Reports **Mean Square Error (MSE)** and execution time.

**Arguments:**
```
-h | -help | -usage | -?        Show help
-path | -folder | -ifd          Input folder (default: "")
-filename | -f | -if            Input filename (required unless sample data available)
-output_path | -ofd             Output folder (default: "")
-flag                           Filter kernel (default: gaussian)
-sigma_range | -sr              Range kernel std. dev. (default: 40.0)
-sigma_spatial | -sp            Spatial kernel std. dev. (default: 4.0)
-box_width | -bw                Box filter width (default: 40)
-epsilon | -eps                 Accuracy threshold (default: 1e-3)
```

**Notes:**  
- If OpenCV is compiled in `samples_gpa_lib/third_party`, input filename is optional (uses OpenCV sample images).  
- Providing `-output_path` disables display mode.  

---

### `demo2` — Batch Image Test
Applies GPA-BF on multiple images using all implementations.

**Arguments:**
```
-h | -help | -usage | -?        Show help
-path | -folder | -ifd          Input folder (default: "")
-output_path | -ofd             Output folder (default: "")
-N                              Number of images to process (default: 10)
-flag                           Filter kernel (default: gaussian)
-sigma_range | -sr              Range kernel std. dev. (default: 40.0)
-sigma_spatial | -sp            Spatial kernel std. dev. (default: 4.0)
-box_width | -bw                Box filter width (default: 40)
-epsilon | -eps                 Accuracy threshold (default: 1e-3)
```

**Notes:**  
- If OpenCV is compiled in `samples_gpa_lib/third_party`, input folder is optional (random sample images used).  
- Providing `-output_path` disables display mode.  

---

## 🚀 Future Improvements  
- Use **texture memory** in CUDA to reduce global memory transfers.  
- Extend NPPI exploration to more kernels.  
- Explore mixed-precision optimizations.  

---

## Performance Comparison

| Method             | Avg Computation Time (ms) | Typical Error Range (approximate)               | Relative Speed Compared to OpenCV CPU |
|--------------------|---------------------------|------------------------------------------------|-------------------------------------|
| OpenCV CPU         | ~37.5                     | None (Reference)                                | 1x (baseline)                       |
| OpenCV OpenCL      | ~37.8                     | [0, 0, 0, 0]                                   | ~1x                                |
| OpenCV CUDA        | ~18.5                     | Very low (order of e-5 to 0)                    | ~2x faster                        |
| NPPI               | ~191                      | High variability, errors up to tens or thousands| ~5x slower                       |
| Custom CUDA Kernel  | ~15.4                     | Moderate errors (approx 0.4 to 2 range)         | ~2.5x faster                      |


# CUDA at Scale Independent Project
