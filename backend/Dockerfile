FROM python:3.8-slim-bullseye

RUN apt-get update && apt-get upgrade -y 

RUN apt install -y cmake build-essential

WORKDIR /app

COPY backend/requirements.txt .

# ------------------------------
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \ 
    git \ 
    libgtk2.0-dev \ 
    pkg-config \ 
    libavcodec-dev \ 
    libavformat-dev \ 
    libswscale-dev \ 
    libjpeg-dev \ 
    libpng-dev \ 
    libtiff-dev \ 
    libdc1394-22-dev \ 
    && apt-get clean \ 
    && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/davisking/dlib.git

RUN cd dlib && mkdir build && cd build && cmake .. && cmake --build . && cd .. && python3 setup.py install

RUN pip install --no-cache-dir -r requirements.txt

COPY backend/ .

EXPOSE 5000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000"]