# Monitoring the Shrinkage of Lake Daihai (岱海) with K-means Unsupervised Classification

**GEOL0069 — Artificial Intelligence for Earth Observation — Final Project**

This project uses free Sentinel-2 satellite imagery and an unsupervised K-means classifier to measure how much surface-water area **Lake Daihai**, a shrinking endorheic lake on the Inner Mongolian Plateau in northern China, lost between **2018 and 2024** — and validates that measurement against independently published figures.


<img width="685" height="380" alt="524163_1_En_1_Fig7_HTML" src="https://github.com/user-attachments/assets/e4bde702-556c-47c5-ac53-7ca2e90b71c0" />


---

## 1. The problem

Lakes across arid and semi-arid northern China have been contracting for decades. Lake Daihai (岱海), in Liangcheng County, Ulanqab, Inner Mongolia, is among the most strongly affected: published remote-sensing reconstructions track it falling from roughly **115 km² in 1989 to about 52 km² by 2018**, with the steepest decline in the years just before 2018. The decline is driven by a warming, highly evaporative climate combined with intensive human water use — expansion of irrigated agriculture and groundwater abstraction in the catchment.

A shrinking closed-basin lake is an ideal target for satellite monitoring. The change is large and unambiguous, and **water is the most spectrally distinct surface in the optical domain** — it absorbs strongly in the near-infrared, so water indices (NDWI, MNDWI) separate water from land almost bimodally. That separability is exactly the condition under which *unsupervised* classification works well, letting us avoid hand-labelling any training data.

**Research question:** *How much surface-water area did Lake Daihai lose between 2018 and 2024, as measured by unsupervised classification of Sentinel-2 imagery, and is that estimate consistent with the published record of the lake's decline?*


<img width="1200" height="683" alt="12665_2022_10526_Fig1_HTML" src="https://github.com/user-attachments/assets/8319380b-6eec-4138-bd3b-b45e46897b0d" />


---

## 2. Method


| Stage | What happens |
|-------|--------------|
| **1. Acquire** | Fetch Sentinel-2 SR imagery from Google Earth Engine for two matched summer windows (2018, 2024). |
| **2. Preprocess** | Mask cloud/shadow/cirrus via the Scene Classification Layer; build a median composite per window; compute NDWI and MNDWI. |
| **3. Cluster** | Fit K-means on standardised (NDWI, MNDWI) features sampled from the 2018 composite; choose *k* by silhouette score. |
| **4. Classify** | Apply the **frozen** 2018 model to the full pixel grid of each date; the cluster with the highest mean NDWI is labelled *water*. |
| **5. Change-detect** | Difference the two water masks; convert water-pixel counts to km² (each pixel = 20 × 20 m). |
| **6. Validate** | Compare the 2018 area estimate to the published ~52 km². |

**Why K-means / unsupervised:** no ground-truth labels are needed, the method is fast and runs on a free CPU, and the strong spectral separability of water makes a small-*k* clustering reliable. Fitting once on 2018 and reusing the model on 2024 prevents model drift from contaminating the change signal.


 <img width="320" height="320" alt="Geographical-location-of-Daihai-Lake-and-Daihai-watershed-the-map-was-prepared-in-ArcGIS_Q320" src="https://github.com/user-attachments/assets/7288a0c2-e19e-4a7c-b285-a80d512d01c4" />



<img width="1400" height="933" alt="1*igBfOi1IFWA_H3aNZG0bzQ" src="https://github.com/user-attachments/assets/bb2c8c50-42ec-42c6-8446-8132afa69edb" />



---

## 3. Results

*Populate from your notebook run.*

| Date | Surface water area |
|------|--------------------|
| 2018 | `__` km² |
| 2024 | `__` km² |
| **Change** | `__` km² (`__`%), ≈ `__`% per year |

**Validation.** The 2018 K-means estimate of `__` km² compares with the published ~52 km² (Sun et al., 2022), a difference of `__`%. Agreement at this level indicates the pipeline is measuring the real lake rather than a clustering artefact.

The change map shows loss concentrated as a band around the former shoreline — the spatial signature of a contracting closed-basin lake, not random noise.
<img width="1589" height="473" alt="Lake Daihai surface-water change(K-means, Sentinel-2)" src="https://github.com/user-attachments/assets/ddd3743e-aed5-4926-bba3-eec887f57519" />
<img width="690" height="590" alt="Lake Daihai pixels in water-index feature space(2018) K-means clusters" src="https://github.com/user-attachments/assets/07f872f5-1bd0-4df6-8d27-183128ac6a25" />
<img width="489" height="490" alt="Lake Daihai area, 2018 vs 2024" src="https://github.com/user-attachments/assets/ea94edc1-c431-4b75-b674-b4134283b6c5" />

---

## 4. Repository structure

```
Daihai_Lake_KMeans/
├── README.md                     This file
├── Daihai_KMeans_Analysis.ipynb  Main analysis notebook (run on Colab)
├── requirements.txt              Python dependencies (for local runs)
├── figures/                      Generated figures
├── data/                         Small exported arrays (optional)
└── LICENSE
```

---

## 5. How to reproduce

The notebook is built for **Google Colab** (free, CPU-only). All heavy spatial computation runs server-side in Earth Engine; only small arrays return to the notebook, so no GPU is needed.

1. Open `Daihai_KMeans_Analysis.ipynb` in Google Colab.
2. Create a free [Google Earth Engine](https://earthengine.google.com) account and a [Google Cloud](https://console.cloud.google.com) project; register the project with Earth Engine (non-commercial use).
3. In the authentication cell, replace `'your-project-id'` with your own project ID.
4. Run the cells top to bottom. The authentication step opens a browser prompt the first time.

To run locally instead: `pip install -r requirements.txt`, then run the notebook in Jupyter (you will still authenticate to Earth Engine).

---

## 6. Environmental cost

**Compute added by this project is small.** The analysis runs on a free Colab CPU session plus Earth Engine servers — no GPU, no training loop. A 1–2 hour session at ~50–100 W is on the order of **0.05–0.2 kWh**; at a typical grid carbon intensity this is roughly **~0.02–0.1 kg CO₂e** for the whole analysis. (The `codecarbon` package can measure this directly — see the final notebook cell.)

**Upstream cost is used, not added.** Launching and operating the Sentinel-2 satellites and the Copernicus/Earth Engine data centres happens regardless of this project; we amortise a negligible slice of an already-collected, openly shared archive.

**The method choice is also the low-carbon choice.** A deep-learning alternative (training a CNN/ViT) would consume one to two orders of magnitude more energy in GPU time, before hyperparameter search. K-means answers the research question at a fraction of the footprint — the lowest-risk and lowest-carbon options coincide.

---

## 7. Limitations

- **Mixed shoreline pixels** at 20 m blend water and wet sediment, adding ~one pixel-width of uncertainty around the perimeter.
- **Two dates** cannot separate a steady trend from inter-annual variability; a multi-date series would constrain the trajectory.
- **Single-season snapshots** — mitigated by matching season and compositing over each summer window.
- **No in-situ ground truth** — validation is against another remote-sensing study, so shared methodological biases cannot be fully excluded.

---

## 8. References

- McFeeters, S. K. (1996). The use of the Normalized Difference Water Index (NDWI) in the delineation of open water features. *Int. J. Remote Sensing*, 17(7), 1425–1432.
- Xu, H. (2006). Modification of normalised difference water index (NDWI) to enhance open water features in remotely sensed imagery. *Int. J. Remote Sensing*, 27(14), 3025–3033.
- Sun, M. et al. (2022). Impacts of climate and human activities on Daihai Lake in a typical semi-arid watershed, Northern China. *PLOS ONE*.
- Variations in lake water storage over Inner Mongolia during recent three decades based on multi-mission satellites (2022). *Journal of Hydrology*.
- Gorelick, N. et al. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of Environment*, 202, 18–27.

*Methodological structure adapted from the GEOL0069 in-class Borneo deforestation K-means notebook, re-pointed from vegetation indices to water indices.*


---

## Contact

Author: Yuning Liu
Department of Earth Sciences, University College London
peko.liu.22@ucl.ac.uk
Project link: https://github.com/lynokep/AI_Final_Project_Daihai
