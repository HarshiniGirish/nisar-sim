FROM continuumio/miniconda3 as base

COPY environment.yml ./
RUN conda update conda && \
    conda env update -f environment.yml -n base && \
    conda clean -af


FROM base as dependencies

ENV PATH="/opt/venv/bin:$PATH"
WORKDIR /opt/stactools-nisar-sim
COPY --from=base /opt/conda /opt/conda
COPY pyproject.toml setup.cfg ./
COPY src/stactools/nisar_sim/__init__.py src/stactools/nisar_sim/
RUN apt-get -y -q update \
    && apt-get -y -q install build-essential \
    && rm -rf /var/lib/apt/lists/
RUN python -m venv /opt/venv
RUN pip install . h5py --no-binary rasterio \
    && ls /opt/venv \
    && rm -r /opt/venv/lib/python3.10/site-packages/stactools/nisar_sim


FROM dependencies as builder

ENV PATH="/opt/venv/bin:$PATH"
WORKDIR /opt/stactools-nisar-sim
COPY --from=base /opt/conda /opt/conda
COPY --from=dependencies /opt/venv /opt/venv
COPY pyproject.toml setup.cfg ./
COPY src ./src
RUN pip install -U pip \
    && pip install . h5py --no-binary rasterio
WORKDIR /
RUN rm -rf /opt/stactools-nisar-sim
CMD [ "stac", "nisarsim" ]


FROM dependencies as dev-dependencies

ENV PATH="/opt/venv/bin:$PATH"
WORKDIR /opt/stactools-nisar-sim
COPY --from=dependencies /opt/venv /opt/venv
COPY requirements-dev.txt .
RUN pip install -r requirements-dev.txt \
    && git init


FROM dev-dependencies as dev-builder

ENV PATH="/opt/venv/bin:$PATH"
WORKDIR /opt/stactools-nisar-sim
COPY --from=base /opt/conda /opt/conda
COPY --from=dependencies /opt/venv /opt/venv
COPY . .
RUN pip install -e .
CMD [ "stac", "nisarsim" ]
