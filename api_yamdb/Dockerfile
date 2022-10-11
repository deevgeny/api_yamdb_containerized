FROM python:3.7-slim

# Prepare working directory
RUN mkdir /app

WORKDIR /app

# Install requirements and copy web app code
COPY requirements.txt .

RUN pip install -r requirements.txt --no-cache-dir --upgrade pip

COPY . .

# Web app entry point
CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000" ]
