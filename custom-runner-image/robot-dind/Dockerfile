FROM ghcr.io/actions/actions-runner:latest

ENV DEBIAN_FRONTEND=noninteractive

## --------- modify below commands to customize

RUN sudo apt update && sudo apt install -y wget && sudo rm -rf /var/lib/apt/lists/*

RUN wget --version

RUN sudo apt-get update

RUN wget -O /tmp/chrome.deb https://mirror.cs.uchicago.edu/google-chrome/pool/main/g/google-chrome-stable/google-chrome-stable_120.0.6099.71-1_amd64.deb

RUN sudo apt install -y --allow-downgrades /tmp/chrome.deb

RUN rm /tmp/chrome.deb

RUN wget https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/119.0.6045.105/linux64/chromedriver-linux64.zip

RUN unzip chromedriver-linux64.zip

RUN sudo cp chromedriver-linux64/chromedriver /usr/local/bin/

## -----------

USER runner




