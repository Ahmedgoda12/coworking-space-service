# Use official slim Python 3.10 image matching our local environment
FROM python:3.10-slim

# Set working directory inside the container
WORKDIR /app

# Install system dependencies required for psycopg2
RUN apt update -y && apt install -y build-essential libpq-dev

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r requirements.txt

# Copy application source code
COPY . .

# Expose Flask port
EXPOSE 5153

# Run the application
CMD ["python", "app.py"]
