# How to deploy the application on EC2 instance

## Prerequisites

- Create a Ubuntu EC2 instance on AWS.
- Update the security group to allow incoming traffic on port 80, 443, 22 and other ports as required.
- Connect to the instance using SSH.
- Update the instance.

```bash
sudo apt-get update -y
```

## Python application setup

- Install python3 and pip3.

```bash
sudo apt install python3 python3-pip -y
```

## Node.js application setup

- Install Node.js and npm.

```bash
sudo apt-get install nodejs npm -y
```

## Docker and Docker Compose setup

- Install and configure Docker.

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

- Install Docker Compose.

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Check the installation by running the following commands.

```bash
docker --version
docker-compose --version
```

## Deploy the application

- Clone the repository you want to deploy.

```bash
git clone <repository-url>
cd <repository-name>
```

- If you are using Docker Compose, run the following command.

```bash
docker-compose up -d
```

- Start the new screen session. Replace `<screen-name>` with the name of the screen you want to create.

```bash
screen -S <screen-name>
```

- If you want to copy the environment file from the local machine to the EC2 instance, use the following command.

```bash
rsync -avz -e "ssh -i <key-file>" <local-path> ubuntu@<public-ip>:<remote-path>
```

- Run the following command to start the application.

  - ### Python application

    - Create a virtual environment and install the dependencies.

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt
    ```

    - Run application specific commands like environment variable setup, etc.

    - Start the application.

    ```bash
    python app.py
    ```

  - ### Node.js application

    - Install the dependencies.

    ```bash
    npm install
    ```

    - Run application specific commands like environment variable setup, etc.

    - Start the application.

    ```bash
    npm start
    ```

- Detach the screen session by pressing `Ctrl + A` followed by `D`.

- To reattach the screen session, run the following command.

```bash
screen -ls
screen -x <screen-id>
```

## Setup Apache as a reverse proxy

- Create a `A` record in the DNS settings of the domain pointing to the public IP of the EC2 instance.

- Install Apache server.

```bash
sudo apt-get install apache2 -y
```

- Install certbot and python3-certbot-apache.

```bash
sudo apt install certbot python3-certbot-apache -y
```

- Configure Apache to use the SSL certificate.

```bash
sudo certbot --apache -d <domain-name>
```

- Update the Apache configuration file.

```bash
cd /etc/apache2/sites-available/
sudo nano 000-default-le-ssl.conf
```

- Add the following lines just before the `ServerName` directive. Replace `<port>` with the port on which the application is running.

```apache
ProxyPass / http://127.0.0.1:<port>
ProxyPassReverse / http://127.0.0.1:<port>
```

- Enable the configuration and restart Apache.

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo systemctl restart apache2
```

- Access the application using the domain name.
