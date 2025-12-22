The file explains the setup of CI/CD pipeline using GCP and CircleCI, stages, configuring triggers  
Step 1: Create a Google Compute Engine Instance  
1. Log in to the GCP Console

2. Navigate to Compute Engine → VM Instances

3. Create a new VM with:

4. OS: Ubuntu 20.04 LTS

5. Machine type: e2-micro (or any)

6. Firewall: Allow SSH (port 22)

7. Note the: External IP address  
   SSH username  
Step-2: Prepare the VM  
1. SSH into VM - ssh <username>@<GCE_VM_EXTERNAL_IP>
2. Install required packages, for python application - sudo apt update  
sudo apt install python3 python3-pip -y
Step-3: Connect the Circle CI to Repository  
1. Log in to CircleCI
2. Connect GitHub account
3. Select the repository
4. Enable the project in CircleCI 
Step-4: Configure CI/CD variables  
1. Project settings->Environment variables, add:
2. External IP of GCP VM
3. SSH username
4. Private SSH key for VM  
Step-5: Adding application files  
1. Add Python script 
2. Add python file for testing
3. Requirements is optional  
Step-6: Create Circle CI configuration file  
Create .circleci/config.yml in the repository  
Defining Pipeline stages  
CircleCI pipelines are divided into jobs, executed as part of a workflow.  
1. Build stage-Set up Python environment  
Install dependencies  
Validate application setup  
version: 2.1  

jobs:  
  build:  
    docker:
      - image: cimg/python:3.10  
    steps:
      - checkout  
      - run:  
          name: Install dependencies
          command: |
            pip install --upgrade pip
            pip install -r requirements.txt

2. Test stage - Run automated unit tests  
Ensure application correctness  
  test:  
    docker:
      - image: cimg/python:3.10  
    steps:
      - checkout  
      - run:  
          name: Run tests
          command: |
            pip install pytest
            pytest test_app.py
3. Deploy stage - Copy application files to GCE VM  
Execute the application on the VM  
  deploy:  
    docker:  
      - image: cimg/python:3.10  
    steps:
      - checkout  
      - run:  
          name: Deploy to GCE VM  
          command: |  
            mkdir -p ~/.ssh  
            echo "$GCP_SSH_KEY" > ~/.ssh/id_rsa  
            chmod 600 ~/.ssh/id_rsa  
            ssh-keyscan -H $GCP_VM_IP >> ~/.ssh/known_hosts  
            scp -r . $GCP_VM_USER@$GCP_VM_IP:/home/$GCP_VM_USER/app  
            ssh $GCP_VM_USER@$GCP_VM_IP "python3 /home/$GCP_VM_USER/app/app.py"  
Configuring Build triggers  
1. Automatic Trigger on Code Changes  
Once the GitHub repository is connected to CircleCI, the pipeline starts automatically when:  
Code is pushed to any branch  
A pull request is created or updated  
Commits are merged into the main branch  
2. Branch-Based Trigger Control  
Branch filters are used to control which jobs run on specific branches  
filters:  
  branches:  
    only: main  
Workflow  
workflows:  
  ci_pipeline:  
    jobs:
      - build  
      - test:  
          requires:
            - build  
      - deploy:  
          requires:
            - test  
          filters:
            branches:
              only: main

