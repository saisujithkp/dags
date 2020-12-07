
*** Trying to get logs (last 100 lines) from worker pod funlogslogtesttask-3fffe1e13e574b428b42c851a0f31ae0 ***

[2020-12-07 20:49:28,624] {__init__.py:50} INFO - Using executor LocalExecutor
[2020-12-07 20:49:28,625] {dagbag.py:417} INFO - Filling up the DagBag from /opt/airflow/dags/dags/show_some_logs.py
Running <TaskInstance: fun_logs.log_test_task 2019-07-26T04:45:00+00:00 [queued]> on host funlogslogtesttask-3fffe1e13e574b428b42c851a0f31ae0

### Task code
import logging
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'benten',
    'depends_on_past': False,
    'start_date': datetime(2019, 7, 25),
    'email_on_failure': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=1),
    'catchup' : False
    }

def logging_is_fun():
    logging.debug("hellow world")
    logging.info("hello world")
    logging.critical("hello world")
    return None

with DAG('fun_logs', schedule_interval='45 * * * *', default_args=default_args) as dag:
    log_task = PythonOperator(python_callable=logging_is_fun, task_id='log_test_task')
    
### configuration set via helm chart
    config:
      AIRFLOW__KUBERNETES__GIT_REPO: "ssh://git@kpmi-bitbucket.appl.kp.org:7999/devops/airflow-dags.git"
      AIRFLOW__KUBERNETES__GIT_BRANCH: "master"
      AIRFLOW__KUBERNETES__GIT_DAGS_FOLDER_MOUNT_POINT: "/opt/airflow/dags"
      AIRFLOW__KUBERNETES__GIT_SSH_KEY_SECRET_NAME: "secret-airflow"
      AIRFLOW__KUBERNETES__GIT_SSH_KEY: "id_rsa"
      AIRFLOW__KUBERNETES__DELETE_WORKER_PODS: "False"
      AIRFLOW__KUBERNETES__RUN_AS_USER: "50000"
      AIRFLOW__CORE__LOAD_EXAMPLES: "False"
      AIRFLOW__SCHEDULER__DAG_DIR_LIST_INTERVAL: "60"
      AIRFLOW__KUBERNETES__WORKER_SERVICE_ACCOUNT_NAME: "airflow"
      AIRFLOW__KUBERNETES__WORKER_CONTAINER_IMAGE_PULL_POLICY: "IfNotPresent"
      AIRFLOW__KUBERNETES__WORKER_PODS_CREATION_BATCH_SIZE: "10"
      AIRFLOW__KUBERNETES__LOGS_VOLUME_CLAIM: ""
      AIRFLOW__KUBERNETES__DAGS_VOLUME_SUBPATH: repo/
      AIRFLOW__WEBSERVER__LOG_FETCH_TIMEOUT_SEC: "20"
    
