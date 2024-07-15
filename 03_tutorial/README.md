# Tutorial for Fundamental Concepts
> https://airflow.apache.org/docs/apache-airflow/stable/tutorial/fundamentals.html


[Understanding the Airflow DAG Context: How Your Tasks Find Their DAG Automatically](https://medium.com/@minkj1992/understanding-the-airflow-dag-context-how-your-tasks-find-their-dag-automatically-80b865dcd1d5)


```py
import textwrap
from datetime import datetime, timedelta

from airflow.models.dag import DAG
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="tutorial",
    description="A simple tutorial DAG",
    schedule=timedelta(days=1),
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=["tutorial"],
    # These args will get passed on to each operator
    # Can be override by task init
    default_args={
        "depends_on_past": False,
        "email": ["airflow@example.com"],
        "email_on_failure": False,
        "email_on_retry": False,
        "retries": 1,
        "retry_delay": timedelta(minutes=5),
        # 'queue': 'bash_queue',
        # 'pool': 'backfill',
        # 'priority_weight': 10,
        # 'end_date': datetime(2016, 1, 1),
        # 'wait_for_downstream': False,
        # 'sla': timedelta(hours=2),
        # 'execution_timeout': timedelta(seconds=300),
        # 'on_failure_callback': some_function, # or list of functions
        # 'on_success_callback': some_other_function, # or list of functions
        # 'on_retry_callback': another_function, # or list of functions
        # 'sla_miss_callback': yet_another_function, # or list of functions
        # 'on_skipped_callback': another_function, #or list of functions
        # 'trigger_rule': 'all_success'
    },
) as dag:
    t1 = BashOperator(
        task_id="print_date",
        bash_command="date",
    )

    t2 = BashOperator(
        task_id="sleep",
        depends_on_past=False,
        bash_command="sleep 5",
        retries=3,
    )

    t1.doc_md = textwrap.dedent(
        """\
    #### Task Documentation
    You can document your task using the attributes `doc_md` (markdown),
    `doc` (plain text), `doc_rst`, `doc_json`, `doc_yaml` which gets
    rendered in the UI's Task Instance Details page.
    ![img](https://imgs.xkcd.com/comics/fixing_problems.png)
    **Image Credit:** Randall Munroe, [XKCD](https://xkcd.com/license.html)
    """
    )

    dag.doc_md = """
    This is a documentation placed anywhere
    """

    templated_command = textwrap.dedent(
        """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
    {% endfor %}
    """
    )

    t3 = BashOperator(
        task_id="templated",
        depends_on_past=False,
        bash_command=templated_command,
    )

    t1 >> [t2, t3]

```

run above script `pyhton [FILE_NAME].py`. 


```sh
> cd ~/airflow
> ls -al
total 1132
drwxr-xr-x    7 minwook     224  7 14 16:43 .
drwxrwxr-x+ 100 minwook    3200  7 15 14:26 ..
-rw-------    1 minwook   83071  7 14 16:40 airflow.cfg
-rw-r--r--    1 minwook 1060864  7 14 16:43 airflow.db
drwxr-xr-x    4 minwook     128  7 14 16:40 logs
-rw-------    1 minwook      16  7 14 16:40 standalone_admin_password.txt
-rw-r--r--    1 minwook    4762  7 14 16:40 webserver_config.py
```

Default airflow conf loc is `~/airflow/airflow.cfg`. Let's check `dags_folder`

```sh
cat ~/airflow/airflow.cfg | grep -i dags_folder
```



