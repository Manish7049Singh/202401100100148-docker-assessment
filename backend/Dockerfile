FROM python:3.12-slim-trixie@sha256:6b25cbe6577518889d553cdd5a5da97d259348d8f6e45b527dd731caaba27a07

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

COPY requirements.txt ./

RUN python -m pip install --no-cache-dir --upgrade \
        pip==26.2.2 setuptools==84.2.1 wheel==0.48.0 \
    && python -m pip install --no-cache-dir -r requirements.txt \
    && python -m pip uninstall --yes pip setuptools wheel \
    && useradd --create-home --shell /usr/sbin/nologin appuser

COPY --chown=appuser:appuser app.py ./

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
