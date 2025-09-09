pipeline {
agent {
label 'slave' // Ensure the slave node has this label
}
stages {
stage('Verify SSH Access') {
steps {
echo 'Testing SSH access to the slave node...'
sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.17.0.3 echo "SSH
connection successful"'
}
}
stage('Setup Environment') {
steps {
echo 'Setting up the environment on the slave node...'
sh '''
ssh root@172.17.0.1 <<EOF
sudo apt update -y
sudo apt install docker.io -y
sudo usermod -aG docker $USER
EOF
'''
}
}
stage('Deploy Containerized Python App') {
steps {
echo 'Deploying containerized Python application...'
sh '''
ssh root@172.17.0.1 <<EOF
mkdir -p /data/python-app
cat <<PYTHON_APP > ~/python-app/app.py
from flask import Flask
app = Flask(__name__)
@app.route('/')
def home():
return "Hello from Python app!"
if __name__ == '__main__':
app.run(host='0.0.0.0', port=5000)
PYTHON_APP
cat <<DOCKERFILE > ~/python-app/Dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
WORKDIR /app
CMD ["python", "app.py"]
DOCKERFILE
cd ~/python-app
docker build -t python-app .
docker run -d -p 5000:5000 python-app
EOF
'''
echo 'Python application deployed and running on the slave node at port 5000.'
}
}
}
post {
always {
echo 'Pipeline execution completed.'
}
}
}
