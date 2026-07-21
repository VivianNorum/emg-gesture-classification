# EMG Gesture Classification

This project is about reading EMG signals from muscles and figuring out what hand gesture someone is making. It's a step toward how myoelectric prosthetic hands work. Right now I'm loading real EMG data, matching it up with gesture labels, looking at what the signals actually look like, cleaning the data, and training a very basic classifier.

## What this does

Loads EMG signal windows, matches them with gesture labels, plots what a few gestures look like, checks the data for problems, and trains a simple classifier to guess the gesture from the signal.

---

## Dataset

Using DS1 from a 2025 MDPI paper called *"Surface EMG Datasets for Hand Gesture Recognition Under Constant and Three-Level Force Conditions"* (found it on Kaggle).

- 20 healthy people, dominant arm
- 3 EMG channels (anterior forearm, posterior forearm, anterior radius)
- 5 gestures: fist, thumb bend, rest, finger extension, middle/ring bend
- 1000 Hz sampling rate, constant moderate force
- Already split into 200 ms windows (200 timesteps), 80% overlap
- 174,000 labeled windows total, split evenly across gestures (34,800 each)

I picked DS1 over the other version of the dataset (DS2, which uses different force levels) and over bigger datasets like NinaPro because it's simpler. Fewer channels, fewer gestures, constant force. Good place to start.

**Download:** [DS1 on Kaggle](https://kaggle.com/datasets/d27a113cf8221f4344e5b6834dedc26fb6fbc16c1a9ad28d56cc5d5d77860c40)

The dataset itself is about 753MB, so it's not in this repo. Download it yourself and put the files in `data/raw/`. That folder is gitignored since it's too big to commit.

---

## Project Structure

```
emg-gesture-classification/
├── data/
│   ├── raw/                 # DS1 files go here (download separately, gitignored)
│   └── src/
│       └── visualization/   # saved plots go here
├── notebooks/
│   └── main_pipeline.ipynb
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Installation

Needs Python 3.12+.

```bash
git clone https://github.com/VivianNorum/emg-gesture-classification.git
cd emg-gesture-classification

python3 -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

On Linux, using a virtual environment like this avoids the "externally managed environment" error pip gives you otherwise.

---

## Usage

1. Download DS1 from Kaggle and put it in `data/raw/`
2. Open `notebooks/main_pipeline.ipynb` in Jupyter
3. Pick the `venv` interpreter as the kernel
4. Run all the cells top to bottom

Takes under a minute, including training the classifier.

---

## Pipeline Overview

Six phases in the notebook:

| Phase | What it does | Output |
|---|---|---|
| 1 | Load EMG signal windows from `EMG_S.mat` | `matriz_ventanas`, shape (174000, 3, 200) |
| 2 | Load and check gesture labels from `Labels.mat` | `labels`, shape (174000, 5), balanced and lined up with the data |
| 3 | Plot one example window, all 3 channels | line plot |
| 4 | Plot one example window per gesture, side by side | 5 plots |
| 5 | Check for missing values, dead/flat windows, and the signal range | no problems found |
| 6 | Get the mean absolute value per channel, split into train/test, train a `LogisticRegression` model | `features`, shape (174000, 3), accuracy around 63% |

---

## Key Findings

The data came out clean. No NaN values, no flat or dead windows anywhere. The values are already normalized (around -5 to 4, std about 0.16), not raw millivolt readings like EMG is usually measured in. The labels are perfectly balanced, 34,800 windows per gesture. One thing worth noting: the data is stored in blocks by gesture, not shuffled, so splitting it into train/test needs to shuffle or stratify first, otherwise you could end up testing on a totally different gesture than you trained on. The current split uses `shuffle=True`, so this is handled.

A simple classifier (logistic regression using the mean absolute value of each channel, just 3 numbers per window) gets about 63% accuracy on a 5-class problem where guessing randomly would get 20%. Not amazing, but a reasonable starting point for such a small feature set and a basic model.

---

## Future Work & Notes

- Try more features per channel (RMS, waveform length, zero-crossing rate), not just mean absolute value
- Stratify the train/test split on purpose (`stratify=gesture_number`) instead of just shuffling
- Look at a confusion matrix or per-class accuracy, not just the overall number, to see which gestures get mixed up
- Try other models (SVM, random forest) and compare to logistic regression
- Double check the gesture-to-column mapping against DS1's actual documentation
- Eventually try real-time classification for actual prosthetic control
- Raw data files aren't in this repo, see the note above and `.gitignore`
- This is a beginner project, so the code is written to be easy to follow, not to be fast or fancy
