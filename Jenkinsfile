properties([parameters([string(defaultValue: '0.0.0', description: '服务版本号', name: 'serverversion'),string(defaultValue: 'false', description: '是否需要推送镜像', name: 'needpushimage')])])

node('go111-npm'){
    ws('/home/jenkins/src/') {
        stage('Git CLone') {
           checkout scm
           script {
              version="${params.serverversion}"
              build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
              project_name = 'wayne'
              sh "echo 用户指定version：${version}"
              sh "echo 是否需要推送镜像：${params.needpushimage}"
              if("${version}"=='0.0.0' || "${version}"=='null' ){
                 version="${build_tag}"
              }
           }
        }

        stage('Build Docker Image') {
            sh "docker build ./ -t repository.parkingwang.com/frm/${project_name}:${version}"
        }

        stage('Commit Docker Image') {
            if("${params.needpushimage}"=='true'){
                withCredentials([usernamePassword(credentialsId: 'Harbor', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} repository.parkingwang.com"
                    sh "docker push repository.parkingwang.com/frm/${project_name}:${version}"
                }
            }
        }

        stage('Save image info') {
            TimeZone.getTimeZone('PRC')
            def date = new Date();
            deployed_time = date.getTime()
            def git_comments = sh(returnStdout: true, script: "git log -n 1 --pretty=format:\"%cd  %h  %s\" --date=short --since=\"last day\"")
            def remove_blank_comments = java.net.URLEncoder.encode(git_comments, "UTF-8")
            def git_commit_date = sh(returnStdout: true, script: "git log -n 1 --pretty=format:\"%cd\" --date=short --since=\"last day\"")
            httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "http://k8s-porter.frm:8443/v1/save-deployed?commit_date=${git_commit_date}&branch_name=${env.BRANCH_NAME}&resource_type=Deployment&commit_id=${build_tag}&project_name=${project_name}&commit_msg=${remove_blank_comments}&ns=frm&deployed_time=${deployed_time}&version=${version}"
        }
     }
}
