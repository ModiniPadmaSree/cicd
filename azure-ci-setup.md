Documentation to setup ci-cd pipeline using Azure and GitLab CI  
Step-1: Create an Azure Virtual Machine  
1. Log in to the Azure Portal
2. Navigate to Virtual Machines → Create
3. Choose:  
   - Image: Ubuntu 22.04 LTS  
   - Authentication type: SSH public key  
   - Username: azureuser
4. Open inbound port 22 (SSH)
5. Create the VM and note down:  
   - Public IP address  
   - Username  
Step 2: Prepare the Azure VM  
Connect to the VM using SSH:  
```bash  
ssh azureuser@<AZURE_VM_PUBLIC_IP>  
Step-3: Install software required for application  
sudo apt update  
sudo apt install python3 python3-pip -y  
Step 4: Create GitLab Repository
1. Log in to GitLab
2. Create a new project repository
3. Clone the repository locally
4. Create a new branch  
Step-5: Adding python files  
1. Add a python script 
2. Add a python file for testing python application
3. Add requirements file is optional  
Step-6: Step 5: Configure GitLab Runner  
GitLab CI uses GitLab Runners to execute jobs.  
Options:
GitLab Shared Runner  
Self-hosted Runner on VM or server  
For basic CI: Use GitLab shared runners with Docker executor  
Step-7: Configure Gitlab CI/CD variables  
1. Navigate to Settings->CI/CD->Variables
2. Add Public IP of Azure VM
3. Add VM SSH Username
4. Add private SSH key of VM  

Step-8: Create GitLab CI Pipeline File  
1. Create .gitlab-ci.yml in the repository root.
2. This file defines:
Pipeline stages  
Job execution order  
Scripts to run  
Trigger conditions  

Defining Pipeline stages  
1. Build  
Validate Python environment  
Install application dependencies  
Ensure code is syntactically correct
build_job:  
  stage: build  
  image: python:3.10  
  script:  
    - echo "Build stage started"  
    - python --version  
    - pip install --upgrade pip  
    - pip install -r requirements.txt  

2. Test  
Run automated tests  
Verify correctness of application logic  
test_job:  
  stage: test  
  image: python:3.10  
  script:  
    - echo "Executing test stage"  
    - pip install pytest  
    - pytest test_app.py  

3. Deploy  
Transfer application files to Azure VM  
Execute application on the VM  
Secure Copy (SCP) for file transfer  
deploy_job:  
  stage: deploy  
  image: python:3.10  
  before_script:  
    - apt-get update -y  
    - apt-get install openssh-client -y  
    - mkdir -p ~/.ssh  
    - echo "$AZURE_SSH_KEY" > ~/.ssh/id_rsa  
    - chmod 600 ~/.ssh/id_rsa  
    - ssh-keyscan -H $AZURE_VM_IP >> ~/.ssh/known_hosts  
  script:  
    - scp -r . $AZURE_VM_USER@$AZURE_VM_IP:/home/$AZURE_VM_USER/app  
    - ssh $AZURE_VM_USER@$AZURE_VM_IP "python3 /home/$AZURE_VM_USER/app/app.py"  
  only:  
    - main  

Configuring build triggers  
Build triggers define when the CI pipeline starts automatically.  
1. Push trigger  
The pipeline starts automatically when code is pushed to the repository.  
build_job:  
  stage: build  
  script:  
    - echo "Build triggered on code push"  
2. Branch-based trigger  
Controls which branches can run specific jobs  
deploy_job:  
  stage: deploy  
  only:  
    - main
3. Merge Request trigger  
Pipeline runs when a merge request is created or updated.  
test_job:  
  stage: test  
  only:  
    - merge_requests

