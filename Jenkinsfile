pipeline {
  agent any
  parameters {
    choice(name: 'DEPLOY_ENV', choices: ['sandbox', 'dev', 'stage', 'prod'], description: "Choose an environment to deploy")
    choice(name: 'DEPLOY_TYPE', choices: ['Deploy', 'ReDeploy', 'Clean'], description: "Choose the deploy type")
    string(name: 'CANARY', defaultValue:"none", description: "MANDATORY FIELD - 'start', 'finish', 'none' - Specify whether to start or finish a canary deploy, or 'none' deploy")
    string(name: 'MYHOSTTYPES', defaultValue:"", description: "master,slave - In redeployment you can define which host type you like to redeploy. If not defined it will redeploy all host types")
  }

  stages {
    stage('Init Environment') {
      environment {
        VAULT_PASSWORD_BUILDENV = credentials("VAULT_PASSWORD_${params.DEPLOY_ENV.toUpperCase()}")
        VAULT_PASSWORD_ALL = credentials('VAULT_PASSWORD_ALL')
      }
      steps {
        sh 'env'
        sh 'pipenv install --python /usr/bin/python3'
      }
    }
    stage('Execute Deploy Playbook') {
      environment {
        VAULT_PASSWORD_BUILDENV = credentials("VAULT_PASSWORD_${params.DEPLOY_ENV.toUpperCase()}")
        VAULT_PASSWORD_ALL = credentials('VAULT_PASSWORD_ALL')
        DEPLOY_ENV="${params.DEPLOY_ENV}"
      }  
      when {
        expression { params.DEPLOY_TYPE == 'Deploy' }
      }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "VTP_${params.DEPLOY_ENV.toUpperCase()}_SSH_KEY", keyFileVariable: 'keyfile', usernameVariable: 'sshuser')]) {
          sh 'env'
          sh 'echo "$DEPLOY_ENV: len $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/sum)  "'
          sh 'echo "all: len $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/sum)"'          
          sh 'pipenv run ansible-playbook  -u ${sshuser} --private-key=${keyfile}  -e buildenv=$DEPLOY_ENV -e clusterid=aws_eu_west_1  -e skip_package_upgrade=true --vault-id=all@.vaultpass-client.py --vault-id=$DEPLOY_ENV@.vaultpass-client.py cluster.yml -e clean=false'
        }
      }
    }
    stage('Execute ReDeploy Playbook with myhosttypes') {
      environment {
        VAULT_PASSWORD_BUILDENV = credentials("VAULT_PASSWORD_${params.DEPLOY_ENV.toUpperCase()}")
        VAULT_PASSWORD_ALL = credentials('VAULT_PASSWORD_ALL')
        DEPLOY_ENV="${params.DEPLOY_ENV}"
        CANARY="-e canary=${params.CANARY}"
        MYHOSTTYPES="-e myhosttypes=${params.MYHOSTTYPES}"
      }      
      when {
        expression { params.DEPLOY_TYPE == 'ReDeploy' && params.MYHOSTTYPES != ""}
      }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "VTP_${params.DEPLOY_ENV.toUpperCase()}_SSH_KEY", keyFileVariable: 'keyfile', usernameVariable: 'sshuser')]) {
          sh 'env'
          sh 'echo "$DEPLOY_ENV: len $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/sum)  "'
          sh 'echo "all: len $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/sum)"'          
          sh 'pipenv run ansible-playbook  -u ${sshuser} --private-key=${keyfile}  -e buildenv=$DEPLOY_ENV -e clusterid=aws_eu_west_1  -e skip_package_upgrade=true --vault-id=all@.vaultpass-client.py --vault-id=$DEPLOY_ENV@.vaultpass-client.py redeploy.yml $CANARY $MYHOSTTYPES'
        }
      }
    }
    stage('Execute ReDeploy Playbook without myhosttypes') {
      environment {
        VAULT_PASSWORD_BUILDENV = credentials("VAULT_PASSWORD_${params.DEPLOY_ENV.toUpperCase()}")
        VAULT_PASSWORD_ALL = credentials('VAULT_PASSWORD_ALL')
        DEPLOY_ENV="${params.DEPLOY_ENV}"
        CANARY="-e canary=${params.CANARY}"
      }      
      when {
        expression { params.DEPLOY_TYPE == 'ReDeploy' && params.MYHOSTTYPES == ""}
      }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "VTP_${params.DEPLOY_ENV.toUpperCase()}_SSH_KEY", keyFileVariable: 'keyfile', usernameVariable: 'sshuser')]) {
          sh 'env'
          sh 'echo "$DEPLOY_ENV: len $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/sum)  "'
          sh 'echo "all: len $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/sum)"'          
          sh 'pipenv run ansible-playbook  -u ${sshuser} --private-key=${keyfile}  -e buildenv=$DEPLOY_ENV -e clusterid=aws_eu_west_1  -e skip_package_upgrade=true --vault-id=all@.vaultpass-client.py --vault-id=$DEPLOY_ENV@.vaultpass-client.py redeploy.yml $CANARY'
        }
      }
    }    
    stage('Execute Clean Playbook') {
      environment {
        VAULT_PASSWORD_BUILDENV = credentials("VAULT_PASSWORD_${params.DEPLOY_ENV.toUpperCase()}")
        VAULT_PASSWORD_ALL = credentials('VAULT_PASSWORD_ALL')
        DEPLOY_ENV="${params.DEPLOY_ENV}"   
      }      
      when {
        expression { params.DEPLOY_TYPE == 'Clean' }
      }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "VTP_${params.DEPLOY_ENV.toUpperCase()}_SSH_KEY", keyFileVariable: 'keyfile', usernameVariable: 'sshuser')]) {
          sh 'env'
          sh 'echo "$DEPLOY_ENV: len $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_BUILDENV | /usr/bin/sum)  "'
          sh 'echo "all: len $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/wc -c) sum $(echo -n $VAULT_PASSWORD_ALL | /usr/bin/sum)"'          
          sh 'pipenv run ansible-playbook  -u ${sshuser} --private-key=${keyfile}  -e buildenv=$DEPLOY_ENV -e clusterid=aws_eu_west_1  -e skip_package_upgrade=true --vault-id=all@.vaultpass-client.py --vault-id=$DEPLOY_ENV@.vaultpass-client.py cluster.yml --tags=clusterbuild_clean -e clean=true'
        }
      }            
    }
  }
}
