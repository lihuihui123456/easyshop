@Library('Autozi') _


// 初始化配置
dockerImage.nameSpace = 'hub.autozi.net/passcar'
dockerImage.releaseNameSpace = 'hub.autozi.com/passcar'
dockerImage.release = params.Release
dockerImage.repoName = 'none'
dockerImage.lastTag = dockerTagGenerate.FormBranch(env.BRANCH_NAME)
dockerImage.versionTag = dockerTagGenerate.DateTimeTag()
// 构建的Pipeline
pipeline {
    // 任意节点
    agent any
    // 构建空间优化
    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: '5', artifactNumToKeepStr: '20', daysToKeepStr: '5', numToKeepStr: '20'))
    }
    // 构建参数化
    parameters {
        booleanParam(name: 'Release', defaultValue: false, description: '释放镜像到IDC中心')
        booleanParam(name: 'Autozi_Common', defaultValue: false, description: '构建Autozi_Common项目')
	    booleanParam(name: 'Build_Settle_Dubbo', defaultValue: false, description: '构建Settle_Dubbo项目')
        booleanParam(name: 'Build_B2BEX', defaultValue: false, description: '构建B2BEX项目')
        booleanParam(name: 'Build_B2CEX', defaultValue: false, description: '构建B2CEX项目')
        booleanParam(name: 'Build_Mobile', defaultValue: false, description: '构建Mobile项目')
		booleanParam(name: 'Build_M2VB2CEX', defaultValue: false, description: '构建M2VB2CEX项目')
        booleanParam(name: 'Build_M2VMobile', defaultValue: false, description: '构建M2VMobile项目')
        booleanParam(name: 'Build_Msite', defaultValue: false, description: '构建Msite项目')
        booleanParam(name: 'Build_Task', defaultValue: false, description: '构建Task项目')
        booleanParam(name: 'Build_Style', defaultValue: false, description: '构建Style项目')
        booleanParam(name: 'Build_Web_Dubbo', defaultValue: false, description: '构建Web_Dubbo项目')
	    booleanParam(name: 'Build_Erp_Dubbo', defaultValue: false, description: '构建Erp_Dubbo项目')
        booleanParam(name: 'Build_Goods_Dubbo', defaultValue: false, description: '构建Goods_Dubbo项目')
        booleanParam(name: 'Build_Order_Dubbo', defaultValue: false, description: '构建Order_Dubbo项目')
    }
    // 工具支持
    tools {
        maven 'Maven3'
        jdk 'Jdk8'
    }
    // 构建步骤
    stages {
        stage('初始化构建环境') {
            steps {
                // 初始化脚本
                autoziInit()
                //打印提示信息
                echo "完成构建环境初始化，开始构建 $env.BRANCH_NAME 分支,Docker Image Name Space:$dockerImage.nameSpace, DcokerImage标志:$dockerImage.lastTag,DockerTag:$dockerImage.versionTag"
            }
        }
                stage('构建Autozi_Common项目代码') {
                    steps {
                        script {
                            if (!params.Autozi_Common) {
                                echo '跳过步骤'
                                return
                            }
                        }
                        dir('autozi-common-parent') {
                            autoziAutoMvn artifacts: '*/target/*.jar,*/target/*.pom', sh: 'mvn clean install -Dmaven.test.skip=true -P "Autozi Public"'
                        }
                    }
                }

        stage('构建Settle_Dubbo项目') {
            when {
                expression {
                    return params.Build_Settle_Dubbo
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'settle-dubbo'
                }
                autoziAutoMvn artifacts: 'autozi-settle-parent/*/target/*.war', sh: 'cd autozi-settle-parent && mvn clean install -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'autozi-settle-parent/autozi-settle-dubbo-service/target', target: '*.war', dockerDir: 'docker/settle-dubbo', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建B2BEX项目') {
            when {
                expression {
                    return params.Build_B2BEX
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'b2bex'
                }
                autoziAutoMvn artifacts: 'passcar-web-b2bex/target/*.war', sh: 'cd passcar-web-b2bex-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-b2bex/target', target: '*.war', dockerDir: 'docker/b2bex', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建B2CEX项目') {
            when {
                expression {
                    return params.Build_B2CEX
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'b2cex'
                }
                autoziAutoMvn artifacts: 'passcar-web-b2cex/target/*.war', sh: 'cd passcar-web-b2cex-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-b2cex/target', target: '*.war', dockerDir: 'docker/b2cex', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Mobile项目') {
            when {
                expression {
                    return params.Build_Mobile
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'mobile'
                }
                autoziAutoMvn artifacts: 'passcar-web-mobile/target/*.war', sh: 'cd passcar-web-mobile-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-mobile/target', target: '*.war', dockerDir: 'docker/mobile', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
		stage('构建M2VB2CEX项目') {
            when {
                expression {
                    return params.Build_M2VB2CEX
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'm2vb2cex'
                }
                autoziAutoMvn artifacts: 'passcar-web-M2Vb2cex/target/*.war', sh: 'cd passcar-web-M2Vb2cex-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-M2Vb2cex/target', target: '*.war', dockerDir: 'docker/m2vb2cex', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建M2VMobile项目') {
            when {
                expression {
                    return params.Build_M2VMobile
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'm2vmobile'
                }
                autoziAutoMvn artifacts: 'passcar-web-M2Vmobile/target/*.war', sh: 'cd passcar-web-M2Vmobile-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-M2Vmobile/target', target: '*.war', dockerDir: 'docker/m2vmobile', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Msite项目') {
            when {
                expression {
                    return params.Build_Msite
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'msite'
                }             
				autoziAutoMvn artifacts: 'passcar-web-msite/target/*.war', sh: 'cd passcar-web-msite-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-msite/target', target: '*.war', dockerDir: 'docker/msite', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Task项目') {
            when {
                expression {
                    return params.Build_Task
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'task'
                }
                autoziAutoMvn artifacts: 'passcar-web-task/target/*.war', sh: 'cd passcar-web-task-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-task/target', target: '*.war', dockerDir: 'docker/task', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Style项目') {
            when {
                expression {
                    return params.Build_Style
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'style'
                }
                autoziAutoMvn artifacts: 'passcar-web-style/target/*.war', sh: 'cd passcar-web-style &&mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-style/target', target: '*.war', dockerDir: 'docker/style', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Web_Dubbo项目') {
            when {
                expression {
                    return params.Build_Web_Dubbo
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'web-dubbo'
                }
                autoziAutoMvn artifacts: 'passcar-web-dubbo-service/target/*.war', sh: 'cd passcar-web-dubbo-service-parent && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-dubbo-service/target', target: '*.war', dockerDir: 'docker/web-dubbo', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
	stage('构建Erp_Dubbo项目') {
            when {
                expression {
                    return params.Build_Erp_Dubbo
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'erp-dubbo'
                }
                autoziAutoMvn artifacts: 'passcar-web-erp-dubbo-service/target/*.war', sh: 'cd passcar-web-erp-dubbo-service-parent && mvn clean install -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-erp-dubbo-service/target', target: '*.war', dockerDir: 'docker/erp-dubbo', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Goods_Dubbo项目') {
            when {
                expression {
                    return params.Build_Goods_Dubbo
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'goods-dubbo'
                }
                autoziAutoMvn artifacts: 'passcar-web-goods-dubbo-service/target/*.war', sh: 'cd passcar-web-goods-dubbo-service-parent && mvn clean install -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-goods-dubbo-service/target', target: '*.war', dockerDir: 'docker/goods-dubbo', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
        stage('构建Order_Dubbo项目') {
            when {
                expression {
                    return params.Build_Order_Dubbo
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'order-dubbo'
                }
                autoziAutoMvn artifacts: 'passcar-web-order-dubbo-service/target/*.war', sh: 'cd passcar-web-order-dubbo-service-parent && mvn clean install -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'passcar-web-order-dubbo-service/target', target: '*.war', dockerDir: 'docker/order-dubbo', docker: '**'
                dir('./docker-image') {
                    //清空工作区
                    deleteDir()
                    dockerUnStash()
                    sh '''
                    cd target
                    jar -vxf *.war
                    rm -rfv *.war
                    ls -lR
                    cd ..
                    '''
                    autoziDockerJob img: dockerImage, build: [
                            imageName: dockerImage.fullVersionImageName()
                    ]
                }
                script {
                    //修改构建说明
                    currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}${dockerImage.fullLastImageName()}\r\n${dockerImage.fullVersionImageName()}"
                    if (params.Release) {
                        //修改构建说明
                        currentBuild.description = "${currentBuild.description == null ? '' : currentBuild.description + '\r\n'}Release:\r\n${dockerImage.fullReleaseLastImageName()}\r\n${dockerImage.fullReleaseVersionImageName()}"
                    }
                }
            }
        }
    }
}
