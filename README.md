## Deploying a Flask App on EC2 Using Gunicorn and Nginx

This README provides a step-by-step guide to hosting a Flask web application on an AWS EC2 instance. We’ll use Gunicorn as the WSGI server and Nginx as a reverse proxy to serve our app efficiently in a production environment.

---

### Step 1: Install Python Virtual Environment

Update the package list and install `python3-venv` to manage isolated Python environments:

```bash
sudo apt-get update
sudo apt-get install python3-venv
```

---

### Step 2: Set Up the Virtual Environment

Create a project directory and set up a virtual environment:

```bash
mkdir project
cd project
python3 -m venv venv
source venv/bin/activate
```

This keeps project dependencies isolated from the system Python installation.

---

### Step 3: Install Flask

Install Flask within your virtual environment, allowing you to develop web applications using Python.
```bash
pip install flask
```

---

### Step 4: Clone Your Flask App from GitHub

Clone your Flask app repository:

```bash
git clone <your-repo-link>
cd <repo-folder>
```

Run the app to ensure it's working locally:

```bash
python app.py
```

---

### Step 5: Install and Run Gunicorn

Install Gunicorn:

```bash
pip install gunicorn
```

Run the app using Gunicorn:Gunicorn, or Green Unicorn, is a WSGI server for running Flask applications. Installing it is a crucial step in deploying a production-ready Flask app.

```bash
gunicorn -b 0.0.0.0:8000 app:app
```

You run Gunicorn, binding it to address 0.0.0.0:8000, and specifying the entry point to your Flask application (app:app).


---

### Step 6: Create a systemd Service for Gunicorn

Create a new systemd service file:

```bash
sudo nano /etc/systemd/system/project.service
```

Add the following configuration:

```ini
[Unit]
Description=Gunicorn instance for Flask Project
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/project
ExecStart=/home/ubuntu/project/venv/bin/gunicorn -b localhost:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start the Gunicorn service:

```bash
sudo systemctl daemon-reload
sudo systemctl start project
sudo systemctl enable project
```

---

### Step 7: Install and Configure Nginx

Install Nginx: Nginx is a web server that will act as a reverse proxy for your Flask application, forwarding requests to Gunicorn.

```bash
sudo apt-get install nginx
```

Start and enable Nginx:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Edit the Nginx default site configuration:

```bash
sudo nano /etc/nginx/sites-available/default
```

Update the configuration as follows: You configure Nginx by editing its default configuration file, specifying the upstream server (Gunicorn) and the location for proxying requests.

```nginx
upstream flask_project {
    server 127.0.0.1:5000;
}

location / {
    proxy_pass http://flask_project;
}
```

Restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

---

### Final Step: Test Your Deployment

Visit your EC2 instance's public IP address in a browser. If everything is set up correctly, you should see your Flask app running via Nginx and Gunicorn.

---

# Create Lambda Layers
To create layers for your Lambda handler, you need to package the required dependencies and libraries into a zip file and then upload it as a layer in the AWS Lambda console or through the AWS CLI. Here are the steps to create layers for your Lambda handler:

1. **Create a Directory for the Layer:**
   Create a directory to organize the files that will be included in your layer.

   ```bash
   mkdir lambda_layer
   cd lambda_layer
   ```

2. **Install Dependencies:**
   Install the required dependencies (e.g., `pandas`, `boto3`, etc.) into the directory. You can use a virtual environment to avoid conflicts with your system's libraries.

   ```bash
   pip install pandas -t .
   pip install boto3 -t .
   ```

   The `-t .` flag tells `pip` to install the packages in the current directory.

3. **Remove Unnecessary Files:**
   Remove any unnecessary files, such as test files or documentation, to keep the layer size smaller.

4. **Create a Zip File:**
   Create a zip file containing the contents of the directory. THe command below will compress your file to size below the maximum size limit in AWS

   ```bash
   powershell "Compress-Archive -Path '.\*' -DestinationPath '.\lambda_layer.zip' -CompressionLevel Optimal -Force"
   ```

5. **Upload as a Layer:**
   Open the AWS Lambda Console, navigate to the Layers section, and click "Create Layer." Provide a name, description, and upload the zip file.


6. **Attach the Layer to Your Lambda Function:**
   Open your Lambda function in the AWS Lambda Console. In the "Designer" section, click "Layers" and then "Add a layer." Select the layer you created.

7. **Test Your Lambda Function:**
   After attaching the layer, test your Lambda function to ensure that it can access the libraries from the layer.

By following these steps, you can create and attach layers to your Lambda function, allowing you to manage dependencies separately and keep your Lambda deployment package smaller.


# Tutorial Guide:

Check out the tutorial for the project: [Watch on YouTube](https://www.youtube.com/watch?v=uqXX4fsUA64&list=PLRRQG4QYcyEXf2sq2XtpfvYmXMgyfWj2g)



