# 🚀 Getting Started with FM4M (Pyenv / WSL 2 Edition)

Welcome to the setup guide for IBM’s multi-modal foundation model for materials, FM4M. This guide adapts the original Conda instructions to use **pyenv** and **pyenv-virtualenv** on an Ubuntu WSL 2 environment.

> **Note:** This tutorial assumes you already have `pyenv` and `pyenv-virtualenv` installed and configured in your shell.

---

## 🛠️ Environment Setup

### 1. Install the Required Python Version
The project officially requires Python 3.9.7 or later. We will use Python 3.9.8 for this setup to match the repository's badge requirements.
```bash
pyenv install 3.9.8
```

### 2. Create the Virtual Environment
Create an isolated virtual environment named `fm4m`.
```bash
# Syntax: pyenv virtualenv <python_version> <environment_name>
pyenv virtualenv 3.9.8 fm4m
```

### 3. Activate the Environment
Navigate to the root directory of your cloned FM4M repository and link the virtual environment so it activates automatically:
```bash
pyenv local fm4m
```
*(Alternatively, you can activate it manually by running `pyenv activate fm4m`).*

### 4. Install Required Packages
With your `fm4m` environment active, upgrade pip and install the standard project dependencies listed in the `requirements.txt` file:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

⚠️ Troubleshooting: Dependency Conflicts
If you encounter a dependency resolution error regarding networkx and scikit-image (e.g., scikit-image 0.26.0 requires networkx>=3.0, but you have networkx 2.8.8), this happens because the graph-based models in FM4M strictly require networkx 2.8.8, while newer versions of scikit-image demand 3.0+.

To fix this, simply downgrade scikit-image to a compatible version by running:

Bash
pip install "scikit-image<0.23"

### 5. Install Torch-Scatter
The FM4M project requires `torch-scatter`, which needs to be compiled specifically for your PyTorch and CUDA versions. 

> **⚠️ Important:** You must replace `${CUDA}` with the specific version that matches your WSL 2 PyTorch installation (e.g., `cpu`, `cu118`, `cu124`, `cu126`, `cu128`, or `cu129`).

```bash
# Replace ${CUDA} before running!
pip install torch-scatter -f [https://data.pyg.org/whl/torch-$(python](https://data.pyg.org/whl/torch-$(python) -c "import torch; print(torch.__version__.split('+')[0])")+${CUDA}.html
```

---

## 💻 Usages

Once your `pyenv` environment is fully set up and active, you have three main ways to utilize FM4M:

### 3-1. Individual model access
One way to utilize FM4M is by accessing each uni-modal model individually. Within each model’s folder (e.g., SMI-TED), you’ll find comprehensive documentation and example notebooks to guide effective usage. This approach allows users to explore and apply each model’s specific functions in detail.

### 3-2. FM4M-Kit (a wrapper toolkit)
A more streamlined approach is to use FM4M-Kit, a wrapper toolkit that enables users to work with all models within a unified framework. The example notebook provides step-by-step instructions for feature extraction with each model, as well as for multi-modal integration.

**Retrieving feature representations:**
To extract representations from a specific model (e.g., `selfies-ted`), call the following api:
```python
# Replace model parameter with any other model name supported by FM4M-Kit
feature_selfies_train = fm4m.get_representation(model="selfies-ted", data=xtrain) #
```

**Downstream modeling and evaluation by a uni-modal model:**
To evaluate the performance of a uni-modal model (e.g., `mhg-ged`), the following example demonstrates how to perform a regression task:
```python
# Replace model and task parameters depending on your requirement
score = fm4m.single_modal(model="MHG-GED", x_train=xtrain, y_train=ytrain, x_test=xtest, y_test=ytest, downstream_model="DefaultClassifier") #
```

**Downstream modeling and evaluation by a multi-modal model:**
For multi-modal model evaluation (e.g., `smi-ted`, `mhg-ged`, and `selfies-ted`), simply list the models as follows:
```python
# Replace model_list and task parameters depending on your requirement
score = fm4m.multi_modal(model_list=["SELFIES-TED","MHG-GED","SMI-TED"], x_train=xtrain, y_train=ytrain, x_test=xtest, y_test=ytest, downstream_model="DefaultClassifier") #
```

### 3-3. Web UI (WSL 2 Local Setup)



The easiest approach is to use FM4M-Kit through a web UI available on Hugging Face Space. This intuitive interface allows you to access FM4M-Kit functions, including data selection, model building, training for downstream tasks, and basic result visualization.

**🛠️ Running the UI Locally inside WSL 2:**
If you are cloning the Space or running the UI code locally via Gradio/Streamlit inside your WSL 2 terminal, you might face issues accessing the `localhost` URL from your Windows browser. Follow these steps to fix it:

1. **Verify Environment:** Ensure your terminal shows `(fm4m)` indicating the pyenv environment is active.
2. **Expose the Host:** By default, local UIs bind to `127.0.0.1`. In WSL 2, you need to bind it to `0.0.0.0` to access it from Windows. 
   * If running a script, append the host argument (e.g., `python app.py --server.address 0.0.0.0` or edit the Gradio `launch(server_name="0.0.0.0")` parameters).
3. **Access from Windows:** Open your Windows web browser and navigate to `http://localhost:<PORT>` (usually `7860` for Gradio or `8501` for Streamlit).
