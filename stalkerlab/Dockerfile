FROM --platform=linux/amd64 python:3.11-slim

# Install Chrome and Chrome WebDriver dependencies
RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    unzip \
    curl \
    fonts-liberation \
    libasound2 \
    libatk-bridge2.0-0 \
    libatk1.0-0 \
    libatspi2.0-0 \
    libcups2 \
    libdbus-1-3 \
    libdrm2 \
    libgbm1 \
    libgtk-3-0 \
    libnspr4 \
    libnss3 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    xdg-utils \
    && mkdir -p /etc/apt/sources.list.d/ \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update \
    && apt-get install -y google-chrome-stable \
    && apt-mark hold google-chrome-stable \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Get Chrome version and install matching ChromeDriver
RUN chrome_version=$(google-chrome --version | awk '{ print $3 }' | cut -d'.' -f1) \
    && wget -q -O /tmp/chromedriver.zip https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/$(curl -s https://googlechromelabs.github.io/chrome-for-testing/LATEST_RELEASE_$chrome_version)/linux64/chromedriver-linux64.zip \
    && unzip /tmp/chromedriver.zip -d /tmp/ \
    && mv /tmp/chromedriver-linux64/chromedriver /usr/local/bin/ \
    && rm -rf /tmp/chromedriver* \
    && chmod +x /usr/local/bin/chromedriver

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY . .

# Modify admin_bot.py to use localhost instead of 127.0.0.1
RUN sed -i 's/127.0.0.1/localhost/g' admin_bot.py

# Create a startup script
RUN echo '#!/bin/bash\n\
python app.py & \n\
sleep 5\n\
export PYTHONUNBUFFERED=1\n\
python admin_bot.py & \n\
wait' > /app/start.sh && chmod +x /app/start.sh

# Expose port
EXPOSE 5000

# Run the startup script
CMD ["/app/start.sh"] 