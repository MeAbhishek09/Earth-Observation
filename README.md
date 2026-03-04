# Earth-Observation

This project implements an Earth Observation (EO) machine learning pipeline to analyze land-use patterns in the Delhi-NCR region using Sentinel-2 satellite imagery and ESA WorldCover 2021 land-cover data.

The goal of the project is to build a spatially aligned dataset by combining satellite image patches with land-cover raster labels and train a deep learning model for land-use classification.

---

# Project Objective

The Ministry of Environment commissioned an AI-based audit of the Delhi Airshed to identify land-use patterns and potential pollution sources.

This project performs the following tasks:

• Spatial filtering of satellite images inside the Delhi-NCR boundary  
• Creation of spatial grids for geographic reasoning  
• Automatic label generation using ESA WorldCover land-cover data  
• Construction of a machine learning dataset  
• Training and evaluation of a CNN-based land-use classifier  

---

# Datasets Used

## 1. Delhi-NCR Boundary
Vector shapefile defining the spatial extent of the NCR region.

Format: GeoJSON / Shapefile  
Coordinate Reference System: EPSG:4326

This boundary is used to filter satellite images that fall within the NCR region.

---

## 2. Sentinel-2 RGB Image Patches

Satellite image patches used as input for the classification model.

Specifications:

Image size: 128 × 128 pixels  
Resolution: 10 meters per pixel  
Format: PNG  

Each filename contains the center coordinate of the image.

Example:

28.6139_77.2090.png

These images represent approximately:

128 pixels × 10 meters = 1.28 km spatial footprint

---

## 3. ESA WorldCover 2021 Land-Cover Raster

The ESA WorldCover dataset provides global land-cover classification.

Specifications:

Resolution: 10 meters per pixel  
Format: GeoTIFF (.tif)  

Each pixel represents a land-use class.

Example classes:

10 → Tree cover  
20 → Shrubland  
30 → Grassland  
40 → Cropland  
50 → Built-up  
60 → Bare land  
80 → Water  
90 → Wetland  

This raster is used as the **ground truth source for labeling satellite images**.

---

# Pipeline Overview

The project pipeline consists of three main stages:

Spatial Filtering → Label Construction → Model Training

---

# Step 1: Spatial Data Processing

## Loading NCR Boundary

The Delhi-NCR shapefile is loaded using GeoPandas.

Original CRS: EPSG:4326 (Latitude–Longitude)

To perform distance-based operations such as grid generation, the shapefile is reprojected to:

EPSG:32644 (UTM Zone 44N)

This projection allows coordinates to be measured in meters.

---

## Creating a 60 × 60 km Grid

A uniform spatial grid is generated over the NCR region.

Each grid cell has size:

60,000 meters × 60,000 meters

The grid is created using the bounding box of the NCR region and helps perform structured spatial analysis.

---

## Filtering Satellite Images Within NCR

Each Sentinel-2 image filename contains latitude and longitude coordinates.

Steps performed:

1. Parse coordinates from image filenames  
2. Convert coordinates into spatial points  
3. Reproject points to EPSG:32644  
4. Perform spatial join with NCR boundary  

Only images whose centers fall inside the NCR region are retained for further analysis.

---

# Step 2: Land-Cover Label Construction

## Extracting Land-Cover Patches

For each filtered RGB image, a corresponding land-cover patch is extracted from the ESA WorldCover raster.

Steps:

1. Convert image center (longitude, latitude) to raster pixel coordinates  
2. Create a centered window of size 128 × 128 pixels  
3. Extract land-cover values from the raster  

Since both datasets have the same resolution (10 m), this ensures spatial alignment.

Spatial footprint:

128 pixels × 10 meters = 1.28 km

Thus, each RGB image and its land-cover patch represent the same geographic region.

---

## Assigning Dominant Land-Cover Class

Each extracted land-cover patch contains multiple pixel values representing different land types.

The label for the image is determined using the **dominant class (mode)**.

Example:

8000 pixels → Built-up  
3000 pixels → Cropland  
512 pixels → Tree  

Dominant class = Built-up

This dominant class becomes the label assigned to the RGB image.

---

## ESA Class Simplification

ESA land-cover codes are mapped to simplified categories used for classification.

Class mapping used in the model:

0 → Built-up  
1 → Cropland  
2 → Grassland  
3 → Shrubland  
4 → Water  
5 → Wetland  
6 → Tree  

This mapping is stored in:

label_map.json

---

# Step 3: Dataset Preparation

After label assignment, the dataset consists of:

X → RGB image patches  
y → encoded land-use labels  

The dataset is split using stratified sampling.

Training set: 60%  
Testing set: 40%

Stratification ensures that class distribution remains balanced across both sets.

Saved dataset files include:

## Processed Dataset Files

| File | Description |
|-----|-------------|
| X_images.npy | Full image dataset (too large to commit) |
| y_labels.npy | Encoded labels |
| X_train.npy | Training images (too large to commit) |
| X_test.npy | Testing images (too large to commit) |
| y_train.npy | Training labels |
| y_test.npy | Testing labels |
| label_map.json | Mapping between class indices and labels |

Saving these files allows reproducibility and reuse of the processed dataset.

---

# Model Training

A Convolutional Neural Network (CNN) is trained to classify land-use categories from RGB image patches.

Input shape:

128 × 128 × 3

Training configuration typically includes:

Optimizer: Adam  
Loss function: Categorical Crossentropy  
Evaluation metrics: Accuracy and F1-score

---

# Model Evaluation

Model performance is evaluated using several metrics.

## Accuracy

Accuracy measures the proportion of correct predictions.

Accuracy = Correct Predictions / Total Predictions

---

## F1 Score

The F1 score balances precision and recall.

F1 = 2 × (Precision × Recall) / (Precision + Recall)

This metric is useful when dealing with imbalanced datasets.

---

## Confusion Matrix

The confusion matrix provides insight into classification performance for each class.

It helps identify:

• Correct predictions  
• Misclassified classes  
• Patterns of confusion between similar land types  

Typical observations:

Built-up areas are often classified accurately due to strong urban patterns.  
Cropland and grassland may sometimes overlap because of similar vegetation characteristics.  
Water bodies are usually detected with high precision due to distinct spectral properties.

---

# Project Structure

project/

Data/  
    rgb/  
    delhi_ncr_region.geojson  
    worldcover_bbox_delhi_ncr_2021.tif  

Dataset/  
    filtered_images.csv  
    processed/  

Visualizations/  

Earth_Observation.pdf  

README.md  

---

# Technologies Used

Python libraries used in this project:

GeoPandas – vector spatial data processing  
Rasterio – raster data handling  
NumPy – numerical computations  
Pandas – dataset management  
OpenCV – image processing  
Scikit-learn – data splitting and evaluation  
Matplotlib – visualization  
TensorFlow / PyTorch – deep learning model training  

---

