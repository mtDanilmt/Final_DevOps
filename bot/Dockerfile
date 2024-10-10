FROM python:3.10-slim

WORKDIR /app
RUN apt-get update && apt-get install -y \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*


RUN mkdir /var/run/sshd

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . /app
EXPOSE 22

CMD ["/bin/bash", "-c", "service ssh start && python main.py"]
