/*
 * Copyright 2020 Crown Copyright
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
//node-affinity
//nodes 1..3 are reserved for Jenkins slave pods.
//node 0 is used for the Jenkins master
@Library('shared-library')_

podTemplate(yaml: '''
apiVersion: v1
kind: Pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: palisade-node-name
            operator: In
            values: 
            - node1
            - node2
            - node3
  containers:
  - name: jnlp
    image: jenkins/jnlp-slave
    imagePullPolicy: Always
    args: 
    - $(JENKINS_SECRET)
    - $(JENKINS_NAME)
    resources:
      requests:
        ephemeral-storage: "4Gi"
      limits:
        ephemeral-storage: "8Gi"
  
  - name: docker-cmds
    image: 779921734503.dkr.ecr.eu-west-1.amazonaws.com/jnlp-did:200608
    imagePullPolicy: IfNotPresent
    command:
    - sleep
    args:
    - 99d
    env:
      - name: DOCKER_HOST
        value: tcp://localhost:2375
''') {

    node(POD_LABEL) {
        def GIT_BRANCH_NAME

        stage('Bootstrap') {
            bootstrap(GIT_BRANCH_NAME)
        }

        stage('Prerequisites') {
            prerequisites(repo: 'Palisade-common', branch: GIT_BRANCH_NAME)
            prerequisites(repo: 'Palisade-clients', branch: GIT_BRANCH_NAME)
        }
        stage('Install, Unit Tests, Checkstyle') {
            dir('Palisade-readers') {
                install(repo: 'Palisade-readers', branch: GIT_BRANCH_NAME)
            }
        }

        stage('Maven deploy') {
            dir('Palisade-readers') {
                container('docker-cmds') {
                    configFileProvider([configFile(fileId: "${env.CONFIG_FILE}", variable: 'MAVEN_SETTINGS')]) {
                        if (("${env.BRANCH_NAME}" == "develop") ||
                                ("${env.BRANCH_NAME}" == "master")) {
                            sh 'mvn -s $MAVEN_SETTINGS deploy -P quick'
                        } else {
                            sh "echo - no deploy"
                        }
                    }
                }
            }
        }
    }
}
