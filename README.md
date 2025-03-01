# Lung & Colon Cancer Classification with Explainable AI

This repository presents a deep learning model that utilizes a customized EfficientNetB3 architecture to classify lung and colon cancer from histopathological images. The model has been fine-tuned for high performance in medical image analysis and is equipped with Explainable AI techniques such as LIME and SHAP, which provide visual insights into the model's decision-making process.

## Features

- **Model Architecture:** Customized EfficientNetB3 tailored for medical image classification.
- **Explainability:** Integrates LIME (Local Interpretable Model-agnostic Explanations) and SHAP (SHapley Additive exPlanations) to highlight the image features influencing the model's predictions.
- **Dataset:** Trained on a comprehensive dataset of 25,000 histopathological images across five classes, covering both benign and cancerous lung and colon tissues.
- **Performance Metrics:**
  - Accuracy: **99.66%**
  - Precision: **99.78%**
  - Recall: **98.58%**
  - F1-score: **99.18%**

## Technologies Used

- **EfficientNetB3:** A scalable deep learning model known for its balance between accuracy and computational efficiency.
- **Explainable AI:** Utilization of LIME and SHAP for model interpretability.
- **Python Libraries:** TensorFlow, Keras, NumPy, and Matplotlib for model development and evaluation.

---

Feel free to explore the repository for more details on implementation and usage.
