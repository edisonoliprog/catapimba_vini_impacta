# catapimba
1. digite `terraform apply --auto-approve`;
2. Acessar o Elastic IP da instância que saiu no output `jenkins_aws_elastic_ip` do Terraform (ex: `http://44.206.118.193`). OBS: Este passo pode demorar para carregar a tela de configuração do Jenkins;
3. Recuperar a senha de administrador do Jenkins acessando o serviço AWS Systems Manager > Repositório de parâmetros > JenkinsinitialAdminPassword. OBS: Este passo pode demorar para criar o parâmetro;
4. Configurar o Jenkins utilizando a senha de administrador e criar um job do tipo pipeline com o nome `Catapimba Infrastructure`, marcar a opção para habilitar a opção `GitHub hook trigger for GITScm polling` e utilizar o script:

```
pipeline {
agent any
  stages {
    stage('Clone') {
      steps {
        git url: 'https://github.com/edisonoliprog/catapimba_vini_impacta', branch: 'main'
      }
    }

    stage('TF Init&Plan') {
      steps {
        script {
          sh 'terraform init'
          sh 'terraform plan -out=myplan.out'
        }
      }
    }

    stage('Approval') {
      steps {
        script {
          def userInput = input(id: 'confirm', message: 'Deseja alterar a Infraestrutura?', description: 'Acao ', name: 'Confirm')
        }
      }
    }

    stage('TF Apply') {
      steps {
          sh 'terraform apply myplan.out'
      }
    }
  }
}
```
