FROM jupyterhub/jupyterhub:3.1.1

RUN pip install psycopg2-binary

WORKDIR /jupyter

ENTRYPOINT ["jupyterhub", "--log-level=DEBUG"]
