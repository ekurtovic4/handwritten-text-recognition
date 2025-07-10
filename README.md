# A Handwritten Text Recognition Model for the Bosnian Language: Mixed Print and Cursive Scripts

## Introduction & Project Overview

## Workflow & Pipeline Structure
**1. Dataset preprocessing**
- The initial dataset consists of 156 different texts, each with an image of the full handwritten text (scanned at 300 DPI) and an XML file containing word labels (in **PascalVOC** format).
- Labels and images of individual words and lines of text are extracted, resulting in three datasets (_labels_w_, _labels_l_, and _labels_l_m_). All images are in JPG format, and all labels are stored in TXT files.
- The data is split into training (70%), validation (20%), and test (10%) partitions.
- **HDF5** files are then created from these datasets, following the previous partitioning.

**2. Model Architecture**
<br><br>
The HTR model in this project is implemented in Python using TensorFlow, following the approach from the [original paper](https://ieeexplore.ieee.org/abstract/document/9266005) and its [GitHub repository](https://github.com/arthurflor23/handwritten-text-recognition/tree/master), with minor adaptations.

The model consists of three main components:

- **Convolutional Block**
  - A series of mini-blocks combining standard and gated convolutions
  - All convolutional layers use He uniform initialization and PReLU activation
  - Batch normalization is applied after each layer
  - Dropout (0.2) is used in the last three gated convolutions for regularization

- **Recurrent Block**  
  - Two bidirectional GRU (BGRU) layers with an intermediate dense layer
  - Dropout (0.5) is applied within the GRU cells

- **Output Layer**  
  - A final dense layer maps to the character set size plus one CTC blank token
  - The model uses Connectionist Temporal Classification (CTC) for transcription

The model architecture is shown in the image below:
<br><br>
<img src="https://github.com/user-attachments/assets/d0f51239-c079-4cc4-80ec-089798c6b9b2" width="300" />

## Sample Data & Formats
Three individual datasets are used, containing images and labels of:
- **words** (_labels_w_)
  
![EM_mix_002_2](https://github.com/user-attachments/assets/dcbf3189-e4e9-496b-9a72-a215f94f5a6d)

`EM_mix_002_2.jpg|2|2092|2928|462|36|654|140|ZNALI`

- **lines of text**, extracted using two different methods (_labels_l_ and _labels_l_m_)
  
![EM_mix_001_1](https://github.com/user-attachments/assets/d5551d52-665a-40d3-a14a-abd3e7c08ba4)

`EM_mix_001_1.jpg|1|2136|3012|120|196|811|391|ŠTA ĆEŠ ONDA ?`

and

`EM_mix_001_1.jpg|1|2136|3012|120|196|811|391|ŠTA ĆEŠ ONDA?`

The datasets are converted into **HDF5** format, following the instructions given in section _1.4 Kreiranje HDF5 formata dataset-a_ of the [notebook](HTR_bos3%20notebook.ipynb). These HDF5 files are used as inputs to the **HTR-flor** model.

Currently supported datasets for HDF5 file creation are the three mentioned above. In order to do the same for other datasets, additional methods would have to be added to the [reader.py](src/data/reader.py) file. For further understanding, it is recommended to analyze the existing methods and the provided [documentation](documentation/HTR_bos3%20dokumentacija.pdf).

## Model Training Process
- Training details:
  - **Connectionist Temporal Classification (CTC)** loss is used
  - Testing includes an additional **CTC decoding** step to convert predictions into readable text

- Optimization:
  - Models use the **AdamW** optimizer
  - Gradient normalization is applied via a custom `NormalizedOptimizer` wrapper class

- Learning rate scheduling and early stopping:
  - The learning rate is reduced by a factor of 0.1 if validation loss does not improve for 15 consecutive epochs
  - Early stopping is triggered if no improvement is seen after 20 epochs

- Three models were trained using identical hyperparameters, with the only difference being the `batch size`

- Each model was trained and evaluated on a distinct dataset: `labels_w`, `labels_l` and `labels_l_m`.

## Evaluation Metrics & Results

To evaluate the model’s performance, three standard metrics for HTR were used:

- **Character Error Rate (CER)** – Measures character-level mistakes using Levenshtein distance, normalized by the number of characters
- **Word Error Rate (WER)** – Measures word-level errors, using Levenshtein distance between word sequences
- **Sequence Error Rate (SER)** – Calculates the percentage of completely incorrect sequences

### Model trained on `labels_w` dataset

- The first and best-performing model was trained on individual words from the `labels_w` dataset
- Due to the larger dataset size, a batch size of 64 was used to speed up training
- Early stopping was triggered after 116 epochs
- Training loss reached a minimum of 0.385, while validation loss was 1.229
- Although validation loss was higher, no signs of overfitting were observed
- After training, the model was tested and achieved the following metrics on the test set:
  - CER: 0.0657
  - WER: 0.1964
  - SER: 0.1964  
### Training and validation loss plot

<img src="https://github.com/user-attachments/assets/45425980-3258-46df-9a7f-f2cc648263dd" width="400" />

### Example predictions

<img src="https://github.com/user-attachments/assets/87d61b41-e75b-4cf3-a3dd-b9f59465de82" width="300" />
<br>
<img src="https://github.com/user-attachments/assets/6a4c7657-4278-422c-9aff-2947663fb2fd" width="300" />
<br>
<img src="https://github.com/user-attachments/assets/4fecf955-6dae-45fc-8719-ea7f86e8f107" width="300" />

---

### Model trained on `labels_l` dataset

- The second model was trained and tested on `labels_l`
- This dataset is significantly smaller so a reduced batch size of 16 was used
- Early stopping triggered after 35 epochs
- Final training and validation losses were 5.391 and 14.907, respectively
- The main reason for poor performance was low data quality: concatenation produced images with inconsistent text segments and mismatched labels
- The test set metrics were:
  - CER: 0.1164
  - WER: 0.2796
  - SER: 0.7688  
- Despite a relatively low character error (11.64%), nearly 77% of sequences contained at least one error, suggesting frequent minor and systematic mistakes
- Attempts to relax early stopping criteria did not yield better results and risked overfitting
### Training and validation loss plot
<img src="https://github.com/user-attachments/assets/20a4e01e-6e28-4dcb-811b-3f9921c07730" width="400"/>

### Example predictions
<img src="https://github.com/user-attachments/assets/0d19ebbc-55c4-47b1-8580-c8aa6480bd1c" width="900" />
<br>
<img src="https://github.com/user-attachments/assets/f2219b8e-1950-4152-ac3f-7d3392e986ae" width="900" />
<br>
<img src="https://github.com/user-attachments/assets/8b9bb27f-7ed6-4ad7-b136-f8b26b9574db" width="900"/>

---

### Model trained on `labels_l_m` dataset

- The third model was trained and tested on `labels_l_m`
- The same model architecture was used
- Training loss reached 4.645 and validation loss 9.037
- Early stopping activated after 160 epochs
- Test metrics improved compared to the previous model:
  - CER: 0.0812
  - WER: 0.2674
  - SER: 0.7174  
- Although character-level errors decreased, the rate of incorrect sequences remained high, indicating limited improvement
### Training and validation loss plot
<img src="https://github.com/user-attachments/assets/ea65da73-1735-402a-8501-6681627bc7a3" width="400" />

### Example predictions
<img src="https://github.com/user-attachments/assets/90ad9fd7-60a3-4b8e-a937-e26eb6395b1e" width="900" />
<br>
<img src="https://github.com/user-attachments/assets/b563ec55-41e8-42f0-9906-14fcee938475" width="900" />
<br>
<img src="https://github.com/user-attachments/assets/bc82e7c0-29b3-48e8-9fab-c5d990c1f5c3" width="900" />

## Model Comparison & Analysis

The graph below compares CER, WER, and SER for all three models:
<br><br>
<img src="https://github.com/user-attachments/assets/34c3d3c7-1128-4d94-8d24-a39ca7ef5c98" width="500" />

The model trained on individual words (`labels_w`) performed best, achieving a CER of 6.57% and WER of 19.64%. Although character error rates are similar across datasets, models trained on concatenated lines show higher word- and sequence-level errors.

This indicates that small character errors accumulate, affecting overall sequence accuracy. Compared to related CNN-LSTM-CTC models, this model is competitive, but there is room for improvement through better data and hyperparameter tuning.


## Referenced repositories
The following repositories were used within the project:
- [handwritten-text-recognition](https://github.com/arthurflor23/handwritten-text-recognition)
- [masters-theses-htr](https://github.com/abegovac2/masters-theses-HTR)
