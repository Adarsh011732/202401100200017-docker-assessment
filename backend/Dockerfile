FROM python:3.12-alpine

# Set build argument for app versioning
ARG APP_VERSION=dev
ENV APP_VERSION=${APP_VERSION}

# Create a non-root user and group
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

# Switch to non-root user
USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
