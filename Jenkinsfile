import groovy.json.JsonSlurperClassic

node {
    withEnv([
        "REGISTRY=ricardodlc/minimal-nodejs-image"
    ]) {
        def app
        def scmVars
        def LATEST_VERSION
        def LATEST_VERSION_NUMBERS
        def LATEST_VERSION_MAJOR_NUMBER
        def LATEST_VERSION_MINOR_NUMBER
        def LATEST_VERSION_PATCH_NUMBER
        def LATEST_VERSION_MAJOR
        def LATEST_VERSION_MINOR
        def LATEST_VERSION_PATCH
        def PREVIOUS_VERSION
        def PREVIOUS_VERSION_NUMBERS
        def PREVIOUS_VERSION_MAJOR_NUMBER
        def PREVIOUS_VERSION_MINOR_NUMBER
        def PREVIOUS_VERSION_PATCH_NUMBER
        def PREVIOUS_VERSION_MAJOR
        def PREVIOUS_VERSION_MINOR
        def PREVIOUS_VERSION_PATCH

        stage('Clone repository') {
            // Let's make sure we have the repository cloned to our workspace
            scmVars = checkout scm
        }

        stage('Generate version variables for tagging') {
            def TAG_PREFIX = ~/^Node/

            // Build tags for the tag tried to build (latest one)
            LATEST_VERSION = sh(returnStdout: true, script: 'git describe --abbrev=0 --tags').trim() - TAG_PREFIX
            LATEST_VERSION_NUMBERS = LATEST_VERSION.split('\\.')
            LATEST_VERSION_MAJOR = LATEST_VERSION_NUMBERS[0]
            LATEST_VERSION_MINOR = LATEST_VERSION_MAJOR + '.' + LATEST_VERSION_NUMBERS[1]
            LATEST_VERSION_PATCH = LATEST_VERSION
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
                if (Integer.parseInt(LATEST_VERSION_MAJOR) > 0) {
                    app.push(LATEST_VERSION_MAJOR)
                }

                app.push(LATEST_VERSION_MINOR)
                app.push(LATEST_VERSION_PATCH)
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
