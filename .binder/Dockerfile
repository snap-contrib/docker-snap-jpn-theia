FROM terradue/l1-binder:3.1

MAINTAINER Terradue S.r.l

USER ${NB_USER}

COPY --chown=${NB_USER}:${NB_GID} . ${HOME}

RUN /opt/anaconda/bin/conda env create --file ${HOME}/.binder/environment.yml && /opt/anaconda/bin/conda clean -a -y

RUN /opt/anaconda/envs/env_s3/bin/python -m ipykernel install --name kernel_s3

ENV PREFIX /opt/anaconda/envs/env_s3

RUN test -f ${HOME}/.binder/postBuild && chmod +x ${HOME}/.binder/postBuild && ${HOME}/.binder/postBuild || exit 0

WORKDIR ${HOME}
