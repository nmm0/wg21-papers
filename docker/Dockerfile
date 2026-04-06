FROM ubuntu:26.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-latex-recommended \
  texlive-latex-extra \
  git
RUN apt-get update && apt-get install -y \
  make \
  curl \
  python3.14-venv

WORKDIR /opt/wg21
RUN git clone https://github.com/mpark/wg21.git
COPY ./entrypoint.sh /opt/entrypoint.sh
RUN chmod +x /opt/entrypoint.sh
WORKDIR /opt/build
RUN make -f /opt/wg21/wg21/Makefile \
  /opt/wg21/wg21/deps/python \
  /opt/wg21/wg21/deps/pandoc/2.18 \
  /opt/wg21/wg21/data/defaults.yaml \
  /opt/wg21/wg21/data/csl.json \
  /opt/wg21/wg21/data/annex-f

ENTRYPOINT ["/opt/entrypoint.sh"]
