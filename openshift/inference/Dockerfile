# ── Inference Service — OpenShift-compatible Dockerfile ─────────────────────
# Base: CUDA 12.1 + Python 3.11 (UBI9-based for OpenShift)
# Runs as non-root user (UID 1001) to comply with OpenShift SCC

FROM nvcr.io/nvidia/cuda:12.1.1-cudnn8-runtime-ubi9

LABEL maintainer="violence-detection-team" \
      version="1.0.0" \
      description="Violence Detection Inference Service"

# ── System deps ───────────────────────────────────────────────────────────────
RUN dnf install -y python3.11 python3.11-pip python3.11-devel \
        gcc gcc-c++ make \
        libGL libGLU \
        libgthread-2.0 \
        glib2 \
        # OpenCV headless deps
        libXext libSM libXrender \
    && dnf clean all \
    && rm -rf /var/cache/dnf

# ── Non-root user (OpenShift requires arbitrary UID support) ──────────────────
RUN useradd -u 1001 -r -g 0 -m -s /bin/bash appuser \
    && chown -R 1001:0 /home/appuser \
    && chmod -R g+rwX   /home/appuser

WORKDIR /app

# ── Python deps ───────────────────────────────────────────────────────────────
COPY inference/requirements.txt .
RUN pip3.11 install --no-cache-dir --upgrade pip \
    && pip3.11 install --no-cache-dir -r requirements.txt

# ── Application code ──────────────────────────────────────────────────────────
COPY inference/ ./inference/
COPY shared/    ./shared/

# ── Model weights (mounted via PVC in OpenShift) ──────────────────────────────
# Default path: /app/models/best.pt  — override with YOLO_WEIGHTS env var
RUN mkdir -p /app/models && chown -R 1001:0 /app/models && chmod -R g+rwX /app/models

# ── Permissions for OpenShift arbitrary UID ───────────────────────────────────
RUN chown -R 1001:0 /app && chmod -R g+rwX /app

USER 1001

ENV PYTHONPATH=/app \
    PYTHONUNBUFFERED=1 \
    PORT=8080 \
    YOLO_WEIGHTS=/app/models/best.pt \
    INFERENCE_DEVICE=cuda \
    CONF_THRESHOLD=0.35 \
    IOU_THRESHOLD=0.45 \
    TARGET_FPS=10 \
    IMG_SIZE=640 \
    TEMPORAL_WINDOW=16

EXPOSE 8080

# Liveness / readiness via FastAPI /health and /ready
HEALTHCHECK --interval=15s --timeout=5s --start-period=30s \
    CMD python3.11 -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')"

CMD ["python3.11", "-m", "uvicorn", "inference.api.main:app", \
     "--host", "0.0.0.0", "--port", "8080", "--workers", "1"]
