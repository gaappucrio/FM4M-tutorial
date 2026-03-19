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

### 4. Clone the original repository
```bash
git clone [https://github.com/IBM/materials.git](https://github.com/IBM/materials.git)
cd materials
```

### 5. Install Required Packages & Fix Dependencies
With your `fm4m` environment active, upgrade pip and install the dependencies. 

*Note: The Web UI requires modern Hugging Face libraries that strictly depend on PyTorch >= 2.4.0. We will pin PyTorch to `2.4.0` immediately to avoid compiling errors later.*

```bash
pip install --upgrade pip

# 1. Pin PyTorch to satisfy Web UI requirements
pip install "torch==2.4.0" torchvision torchaudio

# 2. Install the standard requirements
pip install -r requirements.txt

# 3. Install Web UI specific packages and update xgboost to prevent Python 3.12+ errors
pip install gradio rdkit ipykernel transformers selfies
pip install --upgrade xgboost
```

> **⚠️ Troubleshooting: Dependency Conflicts**
> If you encounter a dependency resolution error regarding `networkx` and `scikit-image` (e.g., scikit-image 0.26.0 requires networkx>=3.0, but you have networkx 2.8.8), this happens because the graph-based models in FM4M strictly require networkx 2.8.8, while newer versions of scikit-image demand 3.0+.
> 
> To fix this, simply downgrade `scikit-image` to a compatible version by running:
> ```bash
> pip install "scikit-image<0.23"
> ```

### 6. Install Torch-Scatter
The FM4M project requires `torch-scatter`, which needs to be compiled specifically for your PyTorch and CUDA versions. Because we pinned PyTorch to `2.4.0`, we must use the exact wheel for that version.

*(The example below uses `cu121` for CUDA 12.1. Replace it with your specific CUDA version like `cu118` or `cpu` if necessary).*

> **⚠️ Note:** Copy the command below exactly as plain text. Avoid copying rich-text link formatting to prevent terminal parsing errors.

```bash
pip install --force-reinstall torch-scatter -f [https://data.pyg.org/whl/torch-2.4.0+cu121.html](https://data.pyg.org/whl/torch-2.4.0+cu121.html)
```

---

## 💻 Usages

Once your `pyenv` environment is fully set up and active, you have three main ways to utilize FM4M:

### 3-1. Individual model access & Hugging Face Migration
One way to utilize FM4M is by accessing each uni-modal model individually. Within each model’s folder (e.g., SMI-TED), you’ll find comprehensive documentation and example notebooks to guide effective usage. 

> **🚨 CRITICAL UPDATE: Outdated GitHub Examples & Hugging Face Migration**
> Using the cloned repository wont just work for this method, so there will be tutorials for each model in this link:
https://drive.google.com/drive/u/1/folders/1dR3DH-lETj8nkPWC20rGXdOe-ZGVHqIw


### 3-2. FM4M-Kit (a wrapper toolkit)
A more streamlined approach is to use FM4M-Kit, a wrapper toolkit that enables users to work with all models within a unified framework. 

**Retrieving feature representations:**
```python
# Replace model parameter with any other model name supported by FM4M-Kit
feature_selfies_train = fm4m.get_representation(model="selfies-ted", data=xtrain)
```

**Downstream modeling and evaluation by a uni-modal model:**
```python
# Replace model and task parameters depending on your requirement
score = fm4m.single_modal(model="MHG-GED", x_train=xtrain, y_train=ytrain, x_test=xtest, y_test=ytest, downstream_model="DefaultClassifier")
```

**Downstream modeling and evaluation by a multi-modal model:**
```python
# Replace model_list and task parameters depending on your requirement
score = fm4m.multi_modal(model_list=["SELFIES-TED","MHG-GED","SMI-TED"], x_train=xtrain, y_train=ytrain, x_test=xtest, y_test=ytest, downstream_model="DefaultClassifier")
```

### 3-3. Web UI (Local WSL 2 Setup)

The easiest approach is to use FM4M-Kit through a web UI. While the official instance is available on Hugging Face Space, you can also run the interface locally. If you downloaded the `app.py` file to run the GUI on your WSL 2 environment, follow these specific steps to resolve network access issues:

**Step 1: Expose the Local Server to Windows**
By default, Gradio apps run on `127.0.0.1` inside Linux, which your Windows browser might not be able to reach. You need to bind the server to `0.0.0.0` to expose it to your host machine.

Open the `app.py` file in your code editor and modify the final launch command at the bottom of the script:

```python
# Change this:
demo.launch() 

# To this:
demo.launch(server_name="0.0.0.0")
```

**Step 2: Run the Application**
Since the application is built with Gradio, do not use Streamlit commands. Run the script directly with Python:

```bash
python app.py
```

**Step 3: Access the UI**
Once the terminal indicates the server is running, open your Windows web browser and navigate to `http://localhost:7860` (or the specific port displayed in your terminal output).
