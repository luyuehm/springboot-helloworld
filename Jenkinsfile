// jenkins slave 执行流水线任务
timeout(time: 600, unit: 'SECONDS') {
    try{
        def label = "jnlp-agent"  
        podTemplate(label: label,cloud: 'kubernetes' ){
            node (label) { 
                agent any
                parameters {
                    gitParameter branch: '', branchFilter: '.*', defaultValue: '', description: '', name: 'Tag', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'DESCENDING_SMART', tagFilter: '*', type: 'PT_TAG'
                }
                stage('Git阶段'){
                    echo "Git 阶段${params.Tag}."
                    git branch: "master" ,changelog: true , url: "https://github.com/luyuehm/springboot-helloworld.git"
                }
                stage('Maven阶段'){
                    container('maven') {
                        //这里引用上面设置的全局的 settings.xml 文件，根据其ID将其引入并创建该文件
                        configFileProvider([configFile(fileId: "c94f922c-5a2d-4fab-a6fe-1eb98298392c", targetLocation: "settings.xml")]){
                            //sh "mvn clean install -Dmaven.test.skip=true --settings settings.xml"
                            }    
                    }
                }
                stage('Docker阶段'){
                    container('docker') {
                        // 读取pom参数
                        echo "读取 pom.xml 参数"
                        pom = readMavenPom file: './pom.xml'
                        // 设置镜像仓库地址
                        hub = "hb.sparke.cn"
                        // 设置仓库项目名
                        project_name = "k8s"
                        echo "编译 Docker 镜像"
                        docker.withRegistry("http://${hub}", "011bcec2-26fc-4e4b-801d-fe53b9d2e5df") {
                            echo "构建镜像"
                            // 设置推送到aliyun仓库的mydlq项目下，并用pom里面设置的项目名与版本号打标签
                            def customImage = docker.build("${hub}/${project_name}/${pom.artifactId}:${pom.version}")
                            echo "推送镜像"
                            //customImage.push()
                            echo "删除镜像"
                            //sh "docker rmi ${hub}/${project_name}/${pom.artifactId}:${pom.version}" 
                        }
                    }
                }
                stage('Deploy阶段') {
                    echo "Deploy Stage"
                    //sh "kubectl apply -f deploy/kubernetes.yaml"
                }
            }
        }
    }catch(Exception e) {
        currentBuild.result = "FAILURE"
    }finally {
        // 获取执行状态
        def currResult = currentBuild.result ?: 'SUCCESS' 
        // 判断执行任务状态，根据不同状态发送邮件
        stage('email'){
            if (currResult == 'SUCCESS') {
                echo "发送成功邮件"
                //emailext(subject: '任务执行成功',to: 'luyuehm@hotmail.com',body: '''任务已经成功构建完成...''')
            }else {
                echo "发送失败邮件"
               //emailext(subject: '任务执行失败',to: 'luyuehm@hotmail.com',body: '''任务执行失败构建失败...''')
            }
        }
    }
}
