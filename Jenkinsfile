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

podTemplate(readYaml file: 'shared-library/com/example/templates/readers.yaml' {

    node(POD_LABEL) {
        def GIT_BRANCH_NAME

        stage('Bootstrap') {
            GIT_BRANCH_NAME = bootstrap(GIT_BRANCH_NAME)
        }

        stage('Prerequisites') {
            prerequisites(repo: 'Palisade-common', branch: GIT_BRANCH_NAME)
            prerequisites(repo: 'Palisade-clients', branch: GIT_BRANCH_NAME)
        }

        stage('Install, Unit Tests, Checkstyle') {
            install(repo: 'Palisade-readers', branch: GIT_BRANCH_NAME)
        }

        stage('SonarQube Analysis') {
            sqAnalysis('Palisade-readers')
        }

        stage("SonarQube Quality Gate") {
            sqQualityGate()
        }

        stage('Maven deploy') {
            deploy(repo: 'Palisade-readers', branch: GIT_BRANCH_NAME)
        }
    }
}
