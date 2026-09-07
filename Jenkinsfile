pipeline {
  agent any
  triggers { githubPush() }
  environment {
    IMAGE = 'chandanavar/flask-ec2-demo:latest'
    ANSIBLE_DIR = '/home/ubuntu/ansible'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh '/usr/local/bin/flask-repo-sync pull /opt/flask-ec2-demo-repository'
      }
    }
    stage('Verify dependencies and Flask') {
      steps {
        sh 'python3 -m venv .ci-venv && .ci-venv/bin/pip install -r requirements.txt && .ci-venv/bin/python -c "from app import app; c=app.test_client(); assert c.get(\"/\").status_code == 200; assert c.get(\"/health\").get_json() == {\"status\": \"ok\"}"'
      }
    }
    stage('Build and test image') {
      steps {
        sh 'docker build -t $IMAGE .'
        sh 'docker run -d --rm --name flask-ci-test -p 5001:5000 $IMAGE && sleep 3 && curl --fail http://127.0.0.1:5001/ && curl --fail http://127.0.0.1:5001/health && docker stop flask-ci-test'
      }
    }
    stage('Push image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_TOKEN')]) {
          sh 'echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin && docker push $IMAGE'
        }
      }
    }
    stage('Deploy with Ansible') {
      steps {
        sh 'cd $ANSIBLE_DIR && ansible-playbook -i inventory.ini playbook.yml'
      }
    }
    stage('Verify managed nodes') {
      steps {
        sh 'cd $ANSIBLE_DIR && ansible managed -i inventory.ini -b -m uri -a "url=http://127.0.0.1:5000/health status_code=200"'
      }
    }
  }
  post {
    success { sh '/usr/local/bin/flask-repo-sync pull /opt/flask-ec2-demo-repository' }
    failure { sh '/usr/local/bin/flask-repo-sync pull /opt/flask-ec2-demo-repository || true' }
  }
}
