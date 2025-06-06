FROM centos:7

WORKDIR /code

RUN yum install epel-release python36  fontconfig wqy-microhei-fonts -y

RUN python3.6 -m ensurepip --default-pip

RUN pip3.6 install pip --upgrade

RUN rpm -vhU https://nmap.org/dist/nmap-7.93-1.x86_64.rpm

RUN yum install python36-devel gcc-c++ -y

RUN yum install nginx unzip wget -y

COPY app/ app/

COPY test/ test/

RUN ln -s /code/app/tools/phantomjs  /usr/bin/phantomjs

COPY docker/frontend/ frontend/

COPY docker/nginx.conf  /etc/nginx/nginx.conf

## 复制生成ssl证书文件
COPY docker/worker/gen_crt.sh /usr/bin/gen_crt.sh
## 复制 wait-for-it
COPY docker/worker/wait-for-it.sh /usr/bin/wait-for-it.sh

## 复制 ncrack依赖
COPY docker/ncrack /usr/local/bin/ncrack
COPY docker/ncrack-services /usr/local/share/ncrack/ncrack-services
## 复制npoc 并安装
COPY docker/ARL-NPoC/ /opt/ARL-NPoC/
WORKDIR /opt/ARL-NPoC/
ENV LANG en_US.UTF-8
RUN pip3.6 install -e .

WORKDIR /code

RUN curl https://ssl-config.mozilla.org/ffdhe2048.txt -o /etc/ssl/certs/dhparam.pem

COPY requirements.txt requirements.txt

#RUN pip3.6 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

COPY docker/GeoLite2/ /data/GeoLite2/

RUN pip3.6 install -r requirements.txt


## 安装 NPoC 的依赖
WORKDIR /opt/ARL-NPoC/
RUN pip3.6 install -r requirements.txt


# 定义nuclei版本
ARG NUCLEI_VERSION=3.4.2

# 安装nuclei
RUN wget "https://github.com/projectdiscovery/nuclei/releases/download/v${NUCLEI_VERSION}/nuclei_${NUCLEI_VERSION}_linux_amd64.zip" \
    && unzip "nuclei_${NUCLEI_VERSION}_linux_amd64.zip" nuclei \
    && mv nuclei /usr/bin/ \
    && rm -f "nuclei_${NUCLEI_VERSION}_linux_amd64.zip"

# 运行nuclei
RUN nuclei -ut

## 安装 WIH
RUN wget https://github.com/Aabyss-Team/arl_files/raw/master/wih/wih_linux_amd64 -O /usr/bin/wih && chmod +x /usr/bin/wih
RUN wih --version

## 切换工作目录
WORKDIR /code

## 自测一下
RUN cp app/config.yaml.example app/config.yaml
RUN env FLASK_APP=app.main:arl_app python3.6 -m flask routes
RUN python3 -m app.celerytask
