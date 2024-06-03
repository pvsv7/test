# test
#!/bin/bash

# Define the URL and HOST

URL="[https://your-url-here.com](https://your-url-here.com/)"
HOST="[your-url-here.com](http://your-url-here.com/)"
PORT=443

# Function to check for SSL issues with curl

check_ssl_with_curl() {
echo "Checking SSL with curl for $URL..."
curl_response=$(curl --silent --head --location "$URL" 2>&1)

```
if echo "$curl_response" | grep -q "SSL certificate problem"; then
    echo "SSL certificate problem detected with $URL"
else
    echo "No SSL certificate problem detected with $URL"
fi

```

}

# Function to retrieve and examine SSL certificate with openssl

check_ssl_with_openssl() {
echo "Checking SSL certificate details with openssl for $HOST..."
cert_output=$(echo | openssl s_client -showcerts -servername $HOST -connect $HOST:$PORT 2>/dev/null)

```
if [ -z "$cert_output" ]; then
    echo "Failed to retrieve the SSL certificate for $HOST"
    exit 1
fi

echo "$cert_output" | openssl x509 -inform pem -noout -text > cert.txt

# Extract the expiration date from the certificate
expiration_date=$(openssl x509 -in cert.txt -noout -enddate | cut -d= -f2)

if [ -z "$expiration_date" ]; then
    echo "Could not parse the certificate expiration date."
    exit 1
fi

# Convert the expiration date to seconds since epoch
expiration_seconds=$(date -d "$expiration_date" +%s)
current_seconds=$(date +%s)

# Calculate the number of days until the certificate expires
days_until_expiration=$(( (expiration_seconds - current_seconds) / 86400 ))

# Check if the certificate is expired or about to expire
if [ $days_until_expiration -le 0 ]; then
    echo "The SSL certificate for $HOST has expired."
else
    echo "The SSL certificate for $HOST is valid for $days_until_expiration more days."
fi

```

}

# Run the functions

check_ssl_with_curl
check_ssl_with_openssl
