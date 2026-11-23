Running on Server:8501

Clone the application --> cd to the folder -->
Create python venv
(python3 -m venv venv1)  
--> activate virtual env (source venv1/bin/activate), venv1 should be added before the hostname (also run which python to confirm)
--> install dependencies inside the venv (pip install -r requirements.txt)

-->

Setup ssh/token authentication with jenkins in github --> global setting --> Devloper setting --> Personal access token

## Create a service for the applicaton to run in background and starts when server gets up

/etc/systemd/system/innews.service

####

[Unit]
Description=InNews Streamlit app
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory= /opt/project2/InNews
Environment='PYTHONUNBUFFERED=1'
ExecStart=/opt/project2/InNews/venv/bin/streamlit run App.py --server.port 8501 --server.address 0.0.0.0 --server.headless true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=mulit-user.target

######

sudo mkdir -p /opt/myapp && sudo chown ubuntu /opt/myapp
sudo systemctl enable myapp

## Allow the deploy user to start it without password prompt

echo 'ubuntu ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp' | sudo tee /etc/sudoers.d/myapp

### Write jenkins file
