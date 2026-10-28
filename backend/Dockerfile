FROM python:3.14.2-slim-bookworm
ARG APP_VERSION=dev
ENV APP_VERSION=${APP_VERSION}
WORKDIR /app
COPY requirements.txt .
RUN python -m pip install --no-cache-dir --upgrade pip setuptools wheel \
	&& python -m pip install --no-cache-dir -r requirements.txt \
	&& groupadd --gid 10001 appuser \
	&& useradd --uid 10001 --gid appuser --create-home appuser
COPY --chown=10001:10001 app.py .
USER 10001:10001
EXPOSE 5000
CMD ["python", "app.py"]