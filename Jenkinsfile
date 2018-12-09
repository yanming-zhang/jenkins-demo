node('rocketzhang-jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm: [$class: 'GitSCM', branches: [[name: '*/master']]]
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }

    stage('Test') {
      echo "2.Test Stage"
    }

    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t registry.cn-shanghai.aliyuncs.com/zym-svc/jenkins-demo:${build_tag} ."
    }

    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'Ali-dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} registry.cn-shanghai.aliyuncs.com"
            sh "docker push registry.cn-shanghai.aliyuncs.com/zym-svc/jenkins-demo:${build_tag}"
            sh "docker rmi registry.cn-shanghai.aliyuncs.com/zym-svc/jenkins-demo:${build_tag}"
        }
    }

    stage('Deploy') {
        echo "5.Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }

        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-test.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-test.yaml"
        sh "kubectl apply -f k8s-test.yaml --record"
    }
}

