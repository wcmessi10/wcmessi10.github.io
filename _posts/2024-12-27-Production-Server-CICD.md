---
layout: post
title: 운영 서버의 CICD는 어떻게 진행되야할까?
tags: [CI/CD, Jenkins]
comments: true
mathjax: true
author: 이희두
---

# **운영 서버**

운영 서버(prod)는 개발용 서버와 다르게 운영 방식이 다르다. 최대한 장애가 발생하면 안되고, 데이터가 유실이 되면 절대 안된다. 그래서 상당히 운영 측면으로 조심해야 하는 부분이 있다. 그러면 DevOps의 배포쪽 CICD는 어떻게 진행되어야 할까?

# **현재 구축된 dev, stage 단계의 CICD**

**사전 설정 :**

1. **인스턴스 내부의 git 글로벌 설정으로 사용자 이메일을 추가해야 한다. 주의점 : 한 번은 인스턴스 내부에서 git fetch나 명령어 통해 계정을 인증해야 한다.**
2. **jenkins에서 시스템 설정에 들어가 Publish over SSH의 SSH Server를 추가한다.**
    1. **Publish over SSH : SSH 접속이 가능하게 하는 플러그인**

```groovy
pipeline {
    agent any

    environment {
        DEPLOY_PATH = "/path/to/deploy/repo"
        LOG_PATH = "/path/to/log"
        JAVA_HOME = "/path/to/java-21"
        PATH = "${JAVA_HOME}/bin:$PATH"
        GIT_USER = "<git-username>"
        GIT_PASSWORD = "<git-password-or-token>"
        SERVICE_NAME = "<service-name>"
    }

    stages {
        stage('Prepare Directories') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        mkdir -p ${DEPLOY_PATH} ${LOG_PATH};
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Pull Latest Code') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        cd ${DEPLOY_PATH} &&
                                        if [ ! -d .git ]; then
                                            git clone -b <branch-name> http://<git-username>:<git-credential>@<git-host>/<org-or-group>/<repo>.git .
                                        else
                                            git pull
                                        fi
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Check Java Version') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        java --version > ${LOG_PATH}/java_version.txt;
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Build Application') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        cd ${DEPLOY_PATH} &&
                                        chmod +x ./gradlew &&
                                        ./gradlew clean build || exit 1;
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Register Service') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        if [ -f /etc/systemd/system/${SERVICE_NAME}.service ]; then
                                            echo "Service file already exists: /etc/systemd/system/${SERVICE_NAME}.service"
                                            echo "Existing service file content:"
                                            cat /etc/systemd/system/${SERVICE_NAME}.service
                                        else
                                            echo '[Unit]
                                            Description=<service-description>
                                            After=network.target

                                            [Service]
                                            User=<run-user>
                                            WorkingDirectory=${DEPLOY_PATH}
                                            ExecStart=/bin/bash -c "exec /usr/bin/java -jar ${DEPLOY_PATH}/build/libs/<app-artifact>.jar > ${LOG_PATH}/${SERVICE_NAME}_\$(date +%%Y%%m%%d_%%H%%M%%S).log 2>&1"
                                            SuccessExitStatus=143
                                            Restart=always
                                            RestartSec=10

                                            [Install]
                                            WantedBy=multi-user.target' | sudo tee /etc/systemd/system/${SERVICE_NAME}.service > /dev/null

                                            sudo systemctl daemon-reload
                                            sudo systemctl enable ${SERVICE_NAME}
                                            echo 'Service registered and enabled.'
                                        fi
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Stop Existing Application') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        echo 'Stopping service...'
                                        sudo systemctl stop ${SERVICE_NAME}
                                        echo 'Service stopped.'
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Start Application') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '<ssh-server-config-name>',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        echo 'Starting service...'
                                        sudo systemctl start ${SERVICE_NAME}
                                        echo 'Service started.'
                                    """
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }
    }
}
````

### **각 stage 별 기능**

Prepare Directories : clone할 위치와 로그가 담길 위치를 생성

Pull Latest Code : 해당 위치로 이동해 clone하거나 pull하여 최신화 한다.

Check Java Version : 자바 버전 체크

Build Application : 소스 빌드

Register Service : 해당 빌드 파일 서비스 등록

Stop Existing Application : 기존 배포 소스 중단

Start Application : 재시작

# **승인 절차**

| Level              | 배포 전략                       |
| ------------------ | --------------------------- |
| 운영 단계              | 승인 단계 포함                    |
| 개발 단계 (dev, stage) | Webhook으로 실시간 이벤트에 반응하여 자동화 |

# **승인 절차란?**

특정 인물의 승인이 이뤄져야 배포가 가능한 절차

### **승인 절차 예시**

jenkins 빌드 -> 바로 설정되어있는 특정 승인자에게 메일/메시지 전송 -> 일정 기간 내에 승인 or 미승인 -> (다음 절차)

### **승인 절차의 특징**

승인 속도: 승인 체계가 배포 속도를 과도하게 저해하지 않도록 설계.

적절한 책임 분배: 승인의 책임자가 과중한 업무 부담을 느끼지 않도록 분산.

모니터링 및 롤백: 승인 후에도 모니터링 및 롤백 체계를 마련.

# **차후 배포 전략**

# **MSA 무중단 배포 전략**

비즈니스 마비를 막기 위해 서비스 기능이 멈추지 않고, 실제 사용자들이 정상적으로 서비스를 사용할 수 있는 기능을 제공하는 배포 방식

* **롤링 배포**

  점진적으로 재배포 하는 형식

  * 인스턴스 관리를 늘리지 않아도 된다.
  * 손쉽게 롤백이 가능
  * 중간에 트래픽이 한곳으로 몰리게 된다.
  * 기존 버전, 새 버전 호환성 문제, 데이터 충돌 문제가 생길 수 있다.

* **블루 그린 배포**

  한꺼번에 모든 서비스를 전환하는 방식

  * 구 버전 기록으로 롤백이 바로 가능하다
  * 운영 환경이 충돌날 일이 없다
  * 인스턴스 활용이 두 배로 필요

* **카나리 배포**

  일부를 신 버전으로 바꿔 분산시켜 오류여부 확인

  * 리스크 최소화
  * 기존 버전과 새 버전 비교 가능
  * 기존 버전, 새 버전 호환성 문제, 데이터 충돌 문제가 생길 수 있다.

