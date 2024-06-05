FROM python:3.9-alpine3.20

# Set default values for environment variables
ARG DB_USERNAME=${DB_USERNAME:-myuser}
ENV DB_USERNAME=${DB_USERNAME}

ARG DB_PASSWORD=${DB_PASSWORD:-mypassword}
ENV DB_PASSWORD=${DB_PASSWORD}

ARG DB_HOST=${DB_HOST:-127.0.0.1}
ENV DB_HOST=${DB_HOST}

ARG DB_PORT=${DB_PORT:-5433}
ENV DB_PORT=${DB_PORT}

ARG DB_NAME=${DB_NAME:-mydatabase}
ENV DB_NAME=${DB_NAME}

## Step 1:
# Create a working directory
WORKDIR /app

## Step 2:
# Copy source code to working directory
COPY ./ requirements.txt /app/

## Step 3:
# Install build dependencies and runtime dependencies
RUN apk update && apk add --no-cache build-base libpq && \
    pip install --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r requirements.txt

## Step 4:
# Expose port 5153
EXPOSE 5153

## Step 5:
# Run app.py at container launch
CMD ["python", "app.py"]
