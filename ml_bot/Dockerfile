FROM pytorch/pytorch:latest
ADD requirements.txt /app/requirements.txt
# set working directory to /app/
WORKDIR /app/
# set up pip and install packages
RUN apt-get update &&\
pip install --upgrade pip &&\
pip install -r requirements.txt &&\
apt-get install -y vim
# Run initial command
CMD bash
