# ========= base ===========
FROM python:3.11-slim AS base

# Add curl for healthcheck
RUN apt-get update
RUN apt-get install -y --no-install-recommends curl
RUN rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

COPY requirements.txt ./requirements.txt

RUN pip install --no-cache-dir -r requirements.txt

# ========== dev ===========
# Ex: docker run -it -p 8080:80 --mount type=bind,src=.,dst=/usr/local/app leTagDeMonImage
FROM base AS dev

RUN pip install watchdog
ENV FLASK_ENV=development
CMD ["python", "app.py"]

# ========== dist ===========
FROM base AS dist

COPY . .

EXPOSE 80

CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
