FROM python:3.9.7-alpine3.14
RUN pip install flask
WORKDIR /app
COPY day-01/app.py .
ENTRYPOINT ["python", "app.py"]
