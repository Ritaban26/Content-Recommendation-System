# Content-Recommendation-System

A color-based image recommendation system that uses unsupervised machine learning (k-means clustering) and vector similarity search (FAISS) to recommend images based on color similarity. The system extracts dominant colors from images and provides real-time search capabilities with 70%+ color matching accuracy.

## Table of Contents

- [Problem Definition](#problem-definition)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Installation](#installation)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Performance & Limitations](#performance--limitations)
- [Future Improvements](#future-improvements)
- [Ethical Considerations](#ethical-considerations)

## Problem Definition

### Problem Statement

In the rapidly expanding digital design and visual art space, professionals face information overload across platforms like Pinterest, Google Images, and Shutterstock. Designers and artists struggle to efficiently locate imagery that aligns with specific visual themes, making inspiration workflows time-consuming and inefficient.

### Real-world Relevance

This project addresses a practical workflow inefficiency in photography and web design. When sourcing images from existing libraries to match specific website themes (e.g., a green color palette), traditional search methods prove inadequate. This system enables filtering and recommending content based on specific color metrics.

## Features

- **Color-Based Search**: Find images similar to a specific color (HEX, RGB, or LAB)
- **Reverse Image Search**: Find images similar to an uploaded image
- **Fast Similarity Search**: FAISS-powered index for millisecond query times (5-50ms)
- **Dominant Color Extraction**: Extract top 5 dominant colors from any image
- **Perceptual Color Matching**: Uses LAB color space for human-like color perception
- **Interactive Widget**: Color picker interface for easy searching
- **Multi-Format Support**: Supports JPG, PNG, BMP, GIF, WEBP

## System Architecture

### Pipeline Overview

1. **Data Ingestion**: Load images from local directory
2. **Feature Extraction**: Extract 5 dominant colors from each image using k-means clustering
3. **Color Space Conversion**: Convert RGB to LAB for perceptual color similarity
4. **Indexing**: Build FAISS (Facebook AI Similarity Search) index for fast retrieval
5. **Query Processing**: Accept user color input and return similar images

### Key Components

#### 1. ImageDataset

- Manages image collection from directories
- Validates and loads images in various formats
- Resizes images for consistent processing
- Stores metadata (file paths, sizes, formats)

#### 2. ColorFeatureExtractor

- Extracts top 5 dominant colors using k-means clustering
- Converts RGB to LAB color space for perceptual accuracy
- Creates 15-dimensional feature vectors (5 colors × 3 LAB values)
- Handles color palette extraction with RGB, LAB, and HEX representations

#### 3. ImageIndex (FAISS)

- Builds searchable index from feature vectors
- Uses L2 distance for similarity measurement
- Supports exact nearest neighbor search
- Provides fast query capabilities

#### 4. ColorQueryEngine

- Parses color input (HEX, RGB, tuple, list)
- Creates query vectors from single colors or images
- Calculates similarity scores (0-1 range)
- Filters results by 70% similarity threshold

#### 5. ColorRecommender

- Main API orchestrating all modules
- Builds recommendation system from image directories
- Provides color-based and image-based search
- Returns ranked results with similarity scores

### Design Justifications

- **LAB Color Space**: Provides color perception closest to human vision, ensuring more intuitive matching
- **FAISS**: Enables fast, scalable similarity search even with large image collections
- **K-means Clustering**: Efficiently identifies dominant colors without supervision
- **Unsupervised Learning**: No labeled data required, works with any image collection

## Installation

### Requirements

- Python 3.8+
- Google Colab (recommended) or Jupyter Notebook

### Dependencies

```bash
pip install numpy>=1.24.0 \
            scikit-image>=0.21.0 \
            Pillow>=10.0.0 \
            faiss-cpu>=1.7.4 \
            scikit-learn>=1.3.0 \
            colorthief>=0.2.1 \
            ipywidgets>=8.0.0 \
            matplotlib>=3.7.0
```

## Usage

### 1. Data Preparation

**Option A: Upload ZIP file (Recommended)**

```python
# Upload a ZIP file containing your images
from google.colab import files
import zipfile

uploaded = files.upload()
for filename in uploaded.keys():
    if filename.endswith('.zip'):
        with zipfile.ZipFile(filename, 'r') as zip_ref:
            zip_ref.extractall('images')

IMAGE_DIR = 'images'
```

**Option B: Mount Google Drive**

```python
from google.colab import drive
drive.mount('/content/drive')
IMAGE_DIR = '/content/drive/MyDrive/your_images_folder'
```

### 2. Build the Recommendation System

```python
recommender = ColorRecommender(n_colors=5, quality=10)

stats = recommender.build_from_directory(
    image_dir=IMAGE_DIR,
    recursive=True,
    max_size=(800, 800)
)
```

### 3. Search by Color

**Using HEX color code:**

```python
results = recommender.recommend_by_color('#FF5733', k=10)
```

**Using RGB tuple:**

```python
results = recommender.recommend_by_color((255, 87, 51), k=10)
```

**Popular colors to try:**

- Vibrant Red: `#FF5733`
- Ocean Blue: `#3498DB`
- Forest Green: `#2ECC71`
- Sunset Orange: `#FF8C42`
- Royal Purple: `#9B59B6`
- Hot Pink: `#E91E63`

### 4. Reverse Image Search

```python
similar = recommender.recommend_by_image(
    'images/your_image.jpg',
    k=10,
    exclude_self=True
)
```

### 5. Extract Dominant Colors

```python
colors = recommender.get_image_colors('images/your_image.jpg')
print("Dominant Colors:")
for i, (rgb, hex_code) in enumerate(zip(colors['rgb_colors'], colors['hex_colors'])):
    print(f"Color {i+1}: {hex_code} - RGB{rgb}")
```

### 6. Interactive Search Widget

```python
# Use the built-in color picker widget in the notebook
# Pick a color, adjust number of results, and click "Search Images"
```

## How It Works

### Feature Extraction Process

1. **K-means Clustering**: Pixels are clustered into 5 dominant colors
2. **Color Ranking**: Clusters sorted by pixel count (most dominant first)
3. **RGB to LAB Conversion**: Each RGB color converted to LAB space
4. **Feature Vector**: 15D vector created (5 colors × 3 LAB dimensions)

### Similarity Calculation

- **Distance Metric**: L2 (Euclidean) distance in LAB space
- **Similarity Score**: `max(0, 1 - distance/50000)`
- **Threshold**: Results filtered at 70% similarity minimum
- **Ranking**: Results sorted by similarity score (highest first)

### Query Processing

1. User provides color (HEX/RGB) or image
2. Query vector created (15D LAB feature)
3. FAISS performs k-nearest neighbor search
4. Results filtered by 70% threshold
5. Top-k results returned with similarity scores

## Performance & Limitations

### Performance

- **Build Time**: ~1-2 seconds per image
- **Query Time**: 5-50ms depending on index size
- **Similarity Threshold**: 70% minimum match
- **Accuracy**: High for well-separated colors

### Limitations

1. **Dataset Dependency**: Results quality depends on dataset size and diversity
2. **Color Separation**: Similar colors may be hard to distinguish with small datasets
3. **Empty Results**: Some colors may return no results if not represented in dataset
4. **Local Only**: Currently designed for local directories (not cloud-optimized)
5. **Image Variety**: Lack of variety makes color separation challenging

## Future Improvements

### Planned Enhancements

1. **Complex Filtering**: Multi-color queries, color combinations, and palettes
2. **Scalability**: Optimization for larger datasets (100K+ images)
3. **Cloud Integration**: Support for cloud storage (S3, Google Cloud Storage)
4. **Advanced Features**:
   - Color weight/importance specification
   - Style and texture similarity
   - Multi-modal search (color + text)
   - Batch processing capabilities

### Domain-Specific Applications

The core system can be adapted for various use cases:

- **Fashion Industry**: Clothing recommendation based on color preferences
- **Interior Design**: Furniture matching for color schemes
- **E-commerce**: Product discovery by color
- **Digital Asset Management**: Organize and search image libraries
- **Photography**: Find complementary images for portfolios

## Ethical Considerations

### Bias & Fairness

- System outputs are entirely dependent on user-provided datasets
- Results will reflect biases present in the training data
- Insufficient dataset size or diversity will impact recommendation quality
- No demographic or cultural bias introduced by the algorithm itself

### Data Privacy

- All processing is local (no data sent to external servers)
- Users have full control over their image collections
- No data retention or sharing mechanisms

### Responsible Use

- Input validation for file types and formats
- Error handling for corrupted or invalid images
- Transparent similarity scoring
- No content moderation (user responsibility)

### Dataset Limitations

- Currently designed for personal/local directories
- Requires sufficient data volume for meaningful results
- User responsible for image rights and usage permissions

## Technical Specifications

- **Feature Dimension**: 15 (5 colors × 3 LAB channels)
- **Clustering Algorithm**: K-means (n_clusters=5, n_init=10, max_iter=500)
- **Index Type**: FAISS IndexFlatL2 (exact search)
- **Color Space**: LAB (perceptual uniformity)
- **Supported Formats**: JPG, JPEG, PNG, BMP, GIF, WEBP
- **Default Image Size**: 800×800 max (maintains aspect ratio)
- **Minimum Similarity**: 70% (configurable)

## Contributing

This is a personal project created for educational purposes and workflow optimization. Suggestions and improvements are welcome!

## License

This project is open-source and available for educational and personal use.

## Acknowledgments

- FAISS: Facebook AI Similarity Search
- Scikit-learn: K-means clustering implementation
- ColorThief: Color extraction utilities
- Scikit-image: Color space conversions

---

**Built with**: Python, FAISS, scikit-learn, Pillow, Jupyter Notebook

**Author**: Ritaban Biswas

**Status**: Active Development
