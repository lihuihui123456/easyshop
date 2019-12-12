@Library('Autozi') _


// 初始化配置
dockerImage.nameSpace = 'hub.huihui.net/passcar'
dockerImage.releaseNameSpace = 'hub.huihui.com/test'
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
        booleanParam(name: 'Build_test', defaultValue: false, description: '构建test项目')
  
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

       
        stage('构建test项目') {
            when {
                expression {
                    return params.Build_test
                }
            }
            steps {
                script {
                    dockerImage.repoName = 'b2bex'
                }
                autoziAutoMvn artifacts: 'easyshop/target/*.war', sh: 'cd easyshop && mvn clean package -Dmaven.test.skip=true -P "Autozi Public"'
                dockerStash targetDir: 'easyshop/target', target: '*.war', dockerDir: 'docker/test', docker: '**'
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
