FROM python:3.8.9
LABEL maintainer="ychanc2104@gmail.com"
ENV PYTHONUNBUFFERED=1
RUN mkdir /docker_api
WORKDIR /docker_api
COPY . /docker_api



RUN pip install --no-cache-dir -r requirements.txt