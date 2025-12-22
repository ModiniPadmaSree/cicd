Documentation to setup ci-cd pipeline using AWS and JENKINS  
Step-1: Launch an EC2 instance for Jenkins  
  
1. Log in to the AWS Management Console.
2. Navigate to EC2 → Launch Instance.
3. Choose Ubuntu Server 22.04 LTS.
4. Select instance type: t2.micro (free tier).
5. Configure Security Group:
6. Allow SSH (22) from your IP
7. Allow HTTP (80) from anywhere
8. Allow Custom TCP (8080) from anywhere (Jenkins)
9. Create a new key pair and download or choose the exisiting key pair.  
  
Step-2: Install jenkins on EC2 instance  
1. Connect to EC2 by: ssh -i <key> ubuntu@<EC2_PUBLIC_IP>
2. Install Java sudo apt update  
   sudo apt install openjdk-17-jdk -y
3. Install jenkins by: curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
   /usr/share/keyrings/jenkins-keyring.asc > /dev/null  
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \  
   https://pkg.jenkins.io/debian binary/ | sudo tee \  
   /etc/apt/sources.list.d/jenkins.list > /dev/null  
  
   sudo apt update  
   sudo apt install jenkins -y  
4. Start jenkins by: sudo systemctl start jenkins  
sudo systemctl enable jenkins  
5. To access jenkins: http://<EC2_PUBLIC_IP>:8080  
  
Step-3: Initial jenkins  
1. Unlock Jenkins:  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
2. Install Suggested Plugins
3. Create Admin User
  
Step 4: Create a Jenkins Pipeline Job  
1. Open Jenkins Dashboard
2. Click New Item
3. Enter name of the pipeline 
4. Select Pipeline from options (freestyle, pipeline, etc)
5. Mostly pipeline is used and choose any one option from pipeline script or pipeline script from SCM where groovy script is written to build, test, deploy.
6. Under Source Code Management, select Git
7. Add repository URL
8. Add GitHub credentials (if private repo)
9. Select the branch of repo.  
  
Step-5: Defining Ci/Cd pipeline Stages for sample python file  
1. Build: The Build stage is the first step in the Continuous Integration pipeline.It ensure that the application can be successfully prepared for execution in a clean and controlled environment.  
   i.Checks if the required runtime (Python) is installed  
  ii.Validates the source code  
 iii.Prepares the application for testing  
   Configuration:  
  i.Jenkins agent is assigned  
 ii.Build commands are executed using shell scripts  
 stage('Build') {  
    steps {  
        echo 'Starting build process...'
        sh 'python3 --version'
    }  
}  
2. Test: The Test stage verifies the correctness and quality of the application by running automated tests.
   i.This ensures that new code changes do not break existing functionality.  
  ii.Executes unit test cases  
 iii.Validates application logic  
  iv.Reports test failures immediately  
   Configuration:  
   i.Jenkins runs test scripts using Python’s built-in unittest framework  
  ii.Test results determine whether the pipeline continues  
stage('Test') {  
    steps {
        echo 'Running automated tests...'
        sh 'python3 -m unittest discover'
    }
}  
3. Deploy: The Deploy stage automatically delivers the tested application to an AWS EC2 instance, making it available for use.  
   i.Jenkins connects to the deployment EC2 instance via SSH.  
  ii.Pulls the latest code from the repository  
 iii.Runs the application on the server  
  Configuration:  
  i.Secure Shell (SSH) is used for remote deployment  
 ii.Deployment occurs only after successful build and test stages  
stage('Deploy') {  
    steps {
        echo 'Deploying application to EC2...'
        sh '''
        ssh -o StrictHostKeyChecking=no ubuntu@<DEPLOY_EC2_IP> "
        cd /home/ubuntu/app &&
        git pull origin main &&
        python3 app.py
        "
        '''
    }
}  
  Step-6: Configure Build triggers  
1. Go to Job Configuration 
2. Enable GitHub hook trigger for GITScm polling
3. In GitHub repository: Go to Settings → Webhooks
4. Add webhook  
Payload URL: http://<JENKINS_EC2_IP>:8080/github-webhook/  
Content type: application/json  
Event: Push  
  
Step-7: Test the pipeline  
i.Make a code change  
ii.Commit and push to GitHub  
iii.Jenkins automatically triggers the pipeline  
iv.Monitor stages:Build, Test, Deploy  
