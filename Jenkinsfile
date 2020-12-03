import groovy.json.JsonSlurperClassic

node {
    withEnv([
        "REGISTRY=ricardodlc/minimal-nodejs-image"
    ]) {
        def app
        def scmVars
        def MAJOR_VERSION
        def MINOR_VERSION
        def PATCH_VERSION

        stage('Clone repository') {
            // Let's make sure we have the repository cloned to our workspace
            scmVars = checkout scm
        }

        stage('Generate version variables for tagging') {
            MAJOR_VERSION = '12'
            MINOR_VERSION = '12.19'
            PATCH_VERSION = '12.19.0'
        }

        stage('Build image from Dockerfile') {
            timeout(time: 6, unit: 'HOURS') {
                sh "docker build -t ${REGISTRY} . --squash"
            }
        }

        stage('List Docker images after build') {
            sh 'docker images'
        }

        stage('Point to recent created image') {
            // This points the builded image to app
            app = docker.image("${REGISTRY}")
        }

        stage('Test image') {
            /* Ideally, we would run a test framework against our image.
            * For this example, we're using a Volkswagen-type approach ;-) */

            app.inside {
                sh 'node -v'
            }
        }

        stage('Push image to Docker Hub') {
            /* Finally, we'll push the image with 3 or 4 tags:
            * First, the 'latest' tag.
            * Second, but not always, the incremental major version.
            * Third, the incremental minor version.
            * Finally, the incremental patch version.
            * Pushing multiple tags is cheap, as all the layers are reused. */

            docker.withRegistry('', 'docker-hub-credentials') {
                app.push("latest")

                // Not push the 0 major version
                if (Integer.parseInt(MAJOR_VERSION) > 0) {
                    app.push(MAJOR_VERSION)
                }

                app.push(MINOR_VERSION)
                app.push(PATCH_VERSION)
            }
        }

        stage('Clean up') {
            /* Delete all tagged images:
            * docker images | grep ${REGISTRY} | tr -s ' ' | cut -d ' ' -f 2 | xargs -I {} docker rmi ${REGISTRY}:{}
            * Note: Using docker image prune -f -a instead in order to fully clean up (remove FROM images i.e.)
            * */
            sh '''
            docker image prune -f -a
            '''
        }
    }
}
