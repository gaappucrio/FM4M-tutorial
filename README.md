Para acessar o tutorial original da IBM: https://github.com/IBM/materials

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

### Clone the original repository
```bash
git clone https://github.com/IBM/materials.git
```

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

First, verify your CUDA version by running:
```bash
python -c "import torch; print(torch.version.cuda)"
(If your output is 12.1, you will use cu121. Map your output accordingly: 11.8 ➡️ cu118, None ➡️ cpu, etc.)
```

Then, install torch-scatter by exporting your PyTorch version to a variable and passing your mapped CUDA version (e.g., cu121) in the URL. This two-step method prevents parsing errors in terminals like zsh:

```bash
# Extract PyTorch version
TORCH_VER=$(python -c "import torch; print(torch.__version__.split('+')[0])")

# Install torch-scatter (Replace cu121 with your specific CUDA version)
pip install torch-scatter -f "[https://data.pyg.org/whl/torch-$](https://data.pyg.org/whl/torch-$){TORCH_VER}+cu121.html"
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

### 3-3. Web UI (Local WSL 2 Setup)

The easiest approach is to use FM4M-Kit through a web UI. While the official instance is available on Hugging Face Space, you can also run the interface locally. If you downloaded the `app.py` file to run the GUI on your WSL 2 environment, follow these specific steps to resolve common dependency and network access issues:

**Step 1: Install UI-Specific Dependencies**
The default `requirements.txt` focuses on core modeling packages. To run the web interface locally, you must manually install Gradio and RDKit (which resolves the `rdkit.Contrib.SA_Score` module error):
```bash
pip install gradio rdkit
```

**Step 2: Expose the Local Server to Windows**
By default, Gradio apps run on `127.0.0.1` inside Linux, which your Windows browser might not be able to reach. You need to bind the server to `0.0.0.0` to expose it to your host machine.

Open the `app.py` file in your code editor and modify the final launch command at the bottom of the script:
```python
# Change this:
demo.launch() 

# To this:
demo.launch(server_name="0.0.0.0")
```

**Step 3: Run the Application**
Since the application is built with Gradio, do not use Streamlit commands. Run the script directly with Python:
```bash
python app.py
```

**Step 4: Access the UI**
Once the terminal indicates the server is running, open your Windows web browser and navigate to `http://localhost:7860` (or the specific port displayed in your terminal output).
