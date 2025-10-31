# 🛰️ sm-smap-nisar-preproc

A modular Jupyter-based workflow for **harmonizing multi-sensor remote-sensing datasets** — including **NISAR GCOV**, **SMAP**, **NDVI**, and **soil clay fraction** — into a common **UTM 200 m grid** on NASA’s **MAAP** platform.

---

## 🌍 Overview

This repository demonstrates end-to-end preprocessing of multi-source Earth Observation data for soil-moisture algorithm development:

| Step | Notebook | Purpose |
|------|-----------|----------|
| 00 | `00_env_check.ipynb` | Verify environment, S3 access, and reprojection tools |
| 01 | `01_discover_links.ipynb` | Discover S3/STAC links for SMAP, NISAR, NDVI, and clay |
| 02 | `02_bbox_to_grid.ipynb` | Define target grid (UTM 200 m) from bounding box |
| 03 | `03_preprocess_clay.ipynb` | Clip and reproject soil clay fraction GeoTIFF |
| 04 | `04_preprocess_ndvi.ipynb` | Read NDVI binary (EASE‑Grid 2) and reproject |
| 05 | `05_preprocess_nisar_hh.ipynb` | Read NISAR GCOV HH, apply RTC, compute σ⁰ → dB |
| 06 (planned) | `06_preprocess_smap.ipynb` | Read SMAP L3 soil‑moisture and reproject to grid |

Each notebook can be run independently in the **MAAP JupyterLab** environment.

---

## ⚙️ Environment setup

```bash
cd /projects/sm-smap-nisar-preproc
mamba env create -f environment.yml
conda activate sm-smap-nisar-preproc
```

To verify kernel:
```bash
jupyter kernelspec list
```

---

## 🗂️ Repository structure

```
sm-smap-nisar-preproc/
├── notebooks/
│   ├── 00_env_check.ipynb
│   ├── 01_discover_links.ipynb
│   ├── 02_bbox_to_grid.ipynb
│   ├── 03_preprocess_clay.ipynb
│   ├── 04_preprocess_ndvi.ipynb
│   └── 05_preprocess_nisar_hh.ipynb
├── src/
│   └── ...
├── config/
│   └── project.yaml
├── docs/
│   └── figures/
├── data/
│   ├── inputs/
│   ├── outputs/
│   │   └── aligned/
│   └── temp/
├── environment.yml
├── .gitignore
└── README.md
```

---

## 🔐 Access and credentials

- **MAAP STAC Catalog:** <https://stac.maap-project.org/>
- **ASF S3 Credentials:** temporary tokens via `MAAP().aws.earthdata_s3_credentials()`
- **NSIDC/SMAP Data:** via [`earthaccess`](https://github.com/nsidc/earthaccess)

> ⚠️ Credentials are short‑lived — re‑run login cell before reading protected S3 data.

---

## 🧩 Data alignment design

All sources are reprojected to the same **UTM 200 m** grid defined by `02_bbox_to_grid.ipynb`.

| Dataset | Native grid | File type | Reprojected to |
|----------|--------------|-----------|----------------|
| NISAR GCOV | 20–80 m | HDF5 | UTM 200 m |
| SMAP L3 | 9 km EASE‑Grid 2 | HDF5 | UTM 200 m |
| NDVI (climatology) | 1 km EASE‑Grid 2 | binary | UTM 200 m |
| Soil Clay Fraction | 250 m Goode Homolosine | GeoTIFF | UTM 200 m |

Reprojection uses `rasterio.warp.reproject()` (bilinear for continuous, nearest for categorical).

---

## 🧠 MATLAB → Python tips

| MATLAB | Python Equivalent |
|---------|-------------------|
| `imread` / `geotiffread` | `rasterio.open(path).read(1)` |
| `maprefcells` / `refmat` | `rasterio.transform.from_origin` |
| `imwarp` | `rasterio.warp.reproject` |
| `struct` | `dict` / `xarray.Dataset` |
| `parfor` | `ThreadPoolExecutor` or `dask` |

---

## 🧭 Example outputs

```
data/outputs/aligned/
├── clay_fraction_utm200m.tif
├── ndvi_window_utm200m.tif
├── nisar_hh_sigma0_db.tif
└── smap_sm_utm200m.tif
```

Each output has a companion JSON metadata file (source link, timestamp, notes).

---

## 🧰 Key dependencies

- `pystac-client`
- `maap-py`
- `rasterio`
- `s3fs`
- `h5py`
- `xarray`
- `numpy`, `pandas`, `matplotlib`
- `pyproj`, `shapely`
- `earthaccess`

---

## 🧾 Citation

> Xu, Xiaolan., *MAAP multi‑sensor soil‑moisture preprocessing toolkit*, 2025, GitHub Repository: <https://github.com/xlxu220/sm-smap-nisar-preproc>

---

## 🧱 Next steps

- [ ] Implement SMAP L3 preprocessing (06)
- [ ] Add global mosaic + temporal stacker
- [ ] Publish tutorials under `/docs/tutorials/`
- [ ] Automate STAC discovery for all sensors
