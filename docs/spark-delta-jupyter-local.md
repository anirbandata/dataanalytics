# Setting Up Spark, Delta Lake, and Jupyter Locally with Miniconda

This guide walks you through setting up Apache Spark, Delta Lake, and Jupyter Lab in a local development environment using Miniconda and SDKMAN for Java management.

## Prerequisites

### 1. Install OpenJDK Java using SDKMAN

SDKMAN is a tool for managing parallel versions of multiple JDKs. It simplifies installation and switching between different Java versions.

#### Install SDKMAN

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Verify the installation:

```bash
sdk version
```

#### Install OpenJDK 17 (Recommended for Spark)

```bash
sdk install java 17.0.9-tem
```

Set it as the default:

```bash
sdk default java 17.0.9-tem
```

Verify the Java installation:

```bash
java -version
```

You should see output similar to:
```
openjdk version "17.0.9" 2023-09-19
OpenJDK Runtime Environment (build 17.0.9+9)
OpenJDK 64-Bit Server VM (build 17.0.9+9, mixed mode, sharing)
```

---

## 2. Install Miniconda

Miniconda is a minimal installer for Conda, which simplifies environment management.

#### Download and Install Miniconda

For macOS:

```bash
# Download the installer
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh

# Run the installer
bash Miniconda3-latest-MacOSX-x86_64.sh
```

Follow the installation prompts and accept the default settings. After installation, restart your terminal or run:

```bash
source ~/miniconda3/bin/activate
```

Verify Conda installation:

```bash
conda --version
```

---

## 3. Create Conda Environment using YAML File

### Create the Environment YAML File

Create a file named `spark-delta-jupyter.yml` in your project root directory with the following configuration:

```yaml
name: spark-delta_jupyter
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - ipykernel
  - nb_conda
  - jupyterlab
  - jupyterlab_code_formatter
  - isort
  - black
  - pyspark=3.5.0
  - pip
  - pip:
    - delta-spark==3.2.0
```

### Create the Conda Environment

Navigate to your project directory and create the environment:

```bash
conda env create -f spark-delta-jupyter.yml
```

This command will:
- Create a new Conda environment named `spark-delta_jupyter`
- Install all specified dependencies including Python 3.12, PySpark 3.5.0, and Delta Lake
- Set up Jupyter Lab with code formatting tools

### Activate the Environment

```bash
conda activate spark-delta_jupyter
```

You should see the environment name in your terminal prompt:

```bash
(spark-delta_jupyter) your-username@hostname ~ %
```

---

## 4. Verify the Installation

### Verify Java

```bash
java -version
```

### Verify Python and Key Packages

```bash
python --version
pyspark --version
python -c "import delta; print(delta.__version__)"
```

### Verify Jupyter Lab Installation

```bash
jupyter lab --version
```

---

## 5. Start Jupyter Lab

With the conda environment activated, launch Jupyter Lab:

```bash
jupyter lab
```

This will open Jupyter Lab in your default web browser at `http://localhost:8888`.

### Create a Test Notebook

1. In Jupyter Lab, create a new Python notebook
2. Test Spark and Delta Lake with the following code:

```python
from pyspark.sql import SparkSession

# Create a Spark session with Delta Lake support
spark = SparkSession.builder \
    .appName("DeltaLakeTest") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Create a simple DataFrame
data = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
columns = ["Name", "Age"]
df = spark.createDataFrame(data, columns)

# Display the DataFrame
df.show()

# Write to Delta Lake format
df.write.format("delta").mode("overwrite").save("/tmp/delta_test")

# Read from Delta Lake format
delta_df = spark.read.format("delta").load("/tmp/delta_test")
delta_df.show()

print("Spark and Delta Lake are working correctly!")
```

---

## 6. Environment Management

### Update the Environment

If you need to add more packages in the future, you can update the `environment.yml` file and apply the changes:

```bash
conda env update -f environment.yml --prune
```

The `--prune` flag removes packages that are no longer in the YAML file.

### List All Conda Environments

```bash
conda env list
```

### Remove the Environment (if needed)

```bash
conda env remove -n spark-delta_jupyter
```

### Deactivate the Environment

```bash
conda deactivate
```

---

## Troubleshooting

### Issue: Java version not found after SDKMAN installation

**Solution:** Ensure SDKMAN is properly initialized by running:

```bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Add this line to your shell configuration file (`~/.zshrc` or `~/.bash_profile`) to make it persistent.

### Issue: Conda command not found

**Solution:** Reinitialize Conda by running:

```bash
conda init zsh
```

Or for bash:

```bash
conda init bash
```

Then restart your terminal.

### Issue: Spark not finding Java

**Solution:** Ensure the `JAVA_HOME` environment variable is set correctly:

```bash
export JAVA_HOME=$HOME/.sdkman/candidates/java/current
```

Add this to your shell configuration file for persistence.

### Issue: Delta Lake import error

**Solution:** Ensure Delta Lake is installed in the correct environment:

```bash
conda activate spark-delta_jupyter
pip install delta-spark==3.2.0
```

---

## Summary

You now have a fully functional local development environment with:

- ✅ OpenJDK Java managed by SDKMAN
- ✅ Miniconda for environment management
- ✅ Apache Spark 3.5.0
- ✅ Delta Lake 3.2.0
- ✅ Jupyter Lab with code formatting tools
- ✅ Python 3.12

This setup is ideal for developing and testing Spark and Delta Lake applications locally before deploying to production environments.
