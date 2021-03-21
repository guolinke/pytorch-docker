FROM mmdog/pytorch:pytorch-1.7-ib2

RUN cd $(mktemp -d) && \
    git clone -q https://github.com/guolinke/fused_ops.git && \
    cd fused_ops && \
    pip install ./