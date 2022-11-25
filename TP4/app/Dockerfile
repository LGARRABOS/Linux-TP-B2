FROM python:3

WORKDIR /usr/src/app

RUN pip install --upgrade pip
RUN pip install scapy netifaces netaddr

COPY ./nmap_project .

CMD [ "python", "./scan.py" ]
