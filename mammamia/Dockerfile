#Use an official Python runtime as a parent image
FROM python:3.11-slim-bullseye

# Set the working directory in the container to /app
WORKDIR /app

RUN apt-get update && \
    apt-get install -y tesseract-ocr libtesseract-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install git
RUN apt-get update && apt-get install -y git

# Clone the repository
RUN git clone --branch main https://github.com/Johnnymnem/MammamiaetuaUHD .

ADD https://raw.githubusercontent.com/Johnnymnem/stremio-ocihosted_JM/refs/heads/main/mammamia/config.json config.json
ADD https://raw.githubusercontent.com/Johnnymnem/stremio-ocihosted_JM/refs/heads/main/mammamia/update_domains.py update_domains.py
# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install --no-cache-dir pytesseract

# Expose the port, for now default is 8080 cause it's the only one really allowed by HuggingFace
EXPOSE 8080

# Run run.py when the container launches
#CMD ["python", "run.py"]
CMD python update_domains.py && python run.py
