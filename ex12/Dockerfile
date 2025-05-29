FROM python:3.9 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --prefix=/dependencies -r requirements.txt

FROM python:3.9-slim
RUN useradd -m user
WORKDIR /
COPY --from=builder /dependencies /usr/local
COPY app.py .
EXPOSE 5000
USER user
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]