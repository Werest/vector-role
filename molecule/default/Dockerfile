FROM ubuntu:latest

RUN apt update -y && \
    apt install -y --no-install-recommends \
    # Основные системные утилиты
    apt-utils \
    systemctl \
    aptitude \
    bash \
    rsync \
    sudo \
    # Инструменты разработки
    gcc \
    git \
    libssl-dev \
    # Python-окружение
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-virtualenv \
    # Python-библиотеки
    python3-apt \
    python3-click \
    python3-toml \
    # Управление конфигурацией
    ansible \
    && apt autoremove -y \
    && apt clean -y \
    && rm -rf /var/lib/apt/lists/*