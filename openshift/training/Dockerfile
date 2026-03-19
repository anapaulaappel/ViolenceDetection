# ── Training Job — OpenShift-compatible Dockerfile ───────────────────────────
# Heavier image: includes dev tools, MediaPipe, scikit-learn

FROM nvcr.io/nvidia/cuda:12.1.1-cudnn8-devel-ubi9

LABEL maintainer="violence-detection-team" \
      version="1.0.0" \
      description="Violence Detection Training Job"

RUN dnf install -y python3.11 python3.11-pip python3.11-devel \
        gcc gcc-c++ make git wget \
        libGL libGLU glib2 \
        libXext libSM libXrender \
    && dnf clean all && rm -rf /var/cache/dnf

RUN useradd -u 1001 -r -g 0 -m -s /bin/bash appuser \
    && chown -R 1001:0 /home/appuser && chmod -R g+rwX /home/appuser

WORKDIR /app

COPY training/requirements.txt .
RUN pip3.11 install --no-cache-dir --upgrade pip \
    && pip3.11 install --no-cache-dir -r requirements.txt

COPY training/ ./training/
COPY shared/   ./shared/

RUN mkdir -p /app/training/data /app/training/models \
    && chown -R 1001:0 /app && chmod -R g+rwX /app

USER 1001

ENV PYTHONPATH=/app \
    PYTHONUNBUFFERED=1

# Default command: run full training
CMD ["python3.11", "training/scripts/train.py", \
     "--dataset-yaml", "training/data/processed/dataset.yaml"]
