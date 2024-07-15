
## Installation
> https://airflow.apache.org/docs/apache-airflow/stable/start.html

```sh
python3.12 -m venv myenv

source myenv/bin/activate
pip install --upgrade pip
```

```sh
export AIRFLOW_HOME=~/airflow
```

```sh
AIRFLOW_VERSION=2.9.2

# Extract the version of Python you have installed. If you're currently using a Python version that is not supported by Airflow, you may want to set this manually.
# See above for supported versions.
PYTHON_VERSION="$(python -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"
# 3.12

CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
# https://raw.githubusercontent.com/apache/airflow/constraints-2.9.2/constraints-3.12.txt

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

```sh
airflow standalone
# localhost:8080
```
