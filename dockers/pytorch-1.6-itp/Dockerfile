FROM mmdog/pytorch:pytorch-1.6

RUN conda install -c r -y conda pip=20.1.1

# ==================================================================
# config & cleanup
# ------------------------------------------------------------------

RUN ldconfig && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/*
