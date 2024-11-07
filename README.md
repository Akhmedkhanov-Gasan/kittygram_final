# Kittygram — A Social Network for Sharing Photos of Beloved Pets

### Project Description
Users can register, upload photos with descriptions, and view pets of other users.

### Installation
<i>Note: All examples are provided for Linux</i><br>
1. Clone the repository to your computer:
    ```
    git clone git@github.com:Akhmedkhanov-Gasan/kittygram_final.git
    ```
2. Create a .env file and fill it with your data. All required variables are listed in the .env.example file in the project's root directory.

### Building Docker Images

1. Replace YOUR_USERNAME with your DockerHub username:

    ```
    cd frontend
    docker build -t YOUR_USERNAME/kittygram_frontend .
    cd ../backend
    docker build -t YOUR_USERNAME/kittygram_backend .
    cd ../nginx
    docker build -t YOUR_USERNAME/kittygram_gateway . 
    ```

2. Upload the images to DockerHub:

    ```
    docker push YOUR_USERNAME/kittygram_frontend
    docker push YOUR_USERNAME/kittygram_backend
    docker push YOUR_USERNAME/kittygram_gateway
    ```

### Server Deployment

1. Connect to the remote server:

    ```
    ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS 
    ```

2. Create a kittygram directory on the server:

    ```
    mkdir kittygram
    ```

3. Install Docker Compose on the server:

    ```
    sudo apt update
    sudo apt install curl
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    sudo apt install docker-compose
    ```

4. Copy the docker-compose.production.yml and .env files to the kittygram/ directory on the server:

    ```
    scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml
    ```
    
    Where:
		PATH_TO_SSH_KEY - path to your SSH key file
		SSH_KEY_NAME - SSH key file name
		YOUR_USERNAME - your server username
		SERVER_IP_ADDRESS - your server's IP address

5. Run Docker Compose in detached mode:

    ```
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Run migrations, collect static files for the backend, and copy them to /backend_static/static/:

    ```
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
	sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
	sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. Open the Nginx configuration file with nano:

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Modify the location settings in the server section:

    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Test the Nginx configuration:

    ```
    sudo nginx -t
    ```

    If successful, you should see:

    ```
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Restart Nginx:

    ```
    sudo service nginx reload
    ```

### CI/CD Setup

1. The workflow file is already written and located at:

    ```
    kittygram/.github/workflows/main.yml
    ```

2. To adapt it to your server, add secrets to GitHub Actions:

    ```
    DOCKER_USERNAME                # DockerHub username
    DOCKER_PASSWORD                # DockerHub password
    HOST                           # Server IP address
    USER                           # Server username
    SSH_KEY                        # Content of the private SSH key (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # SSH key passphrase

    TELEGRAM_TO                    # Your Telegram account ID (use @userinfobot, command /start)
    TELEGRAM_TOKEN                 # Bot token (get it from @BotFather, command /token, bot name)
    ```

### Technologies
Python 3.11.1,
Django 3.2.3,
djangorestframework==3.12.4, 
PostgreSQL 13.10