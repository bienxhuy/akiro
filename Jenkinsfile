pipeline {
    agent any

    triggers {
        cron('0 0 * * *')
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Define which branch to run test on.')
        string(name: 'AUTHOR', defaultValue: 'jenkins', description: 'The one who starts the build.')
        string(name: 'HOST', defaultValue: 'jenkins', description: 'Define the host for the test run.')
        string(name: 'PERFORMANCE_RUN_TIME', defaultValue: '2m', description: 'Define the runtime for the performance test.')
        string(name: 'PERFORMANCE_USERS', defaultValue: '20', description: 'Define the number of users for the performance test.')
        string(name: 'PERFORMANCE_SPAWN_RATE', defaultValue: '5', description: 'Define the spawn rate for the performance test.')
        string(name: 'FRAMEWORK_LOG_DIR', defaultValue: 'reports/logs', description: 'Define the directory for log files.')
        string(name: 'FRAMEWORK_SCREENSHOT_DIR', defaultValue: 'reports/screenshots', description: 'Define the directory for screenshot files.')
        choice(name: 'BROWSER', 
            choices: ['chrome-headless','chrome'], description: 'Define the browser for the test run.')
        choice(name: 'RUN_TYPE',
            choices: ['nightly', 'push', 'manual'], description: 'Define the type of run.')
    }

    environment {
        DOCKER_NETWORK = 'akiro-persist-containers'

        // Config container name based on BUILD_NUMBER
        BE_PORT = '3000'
        FE_PORT  = '4173'

        BE_IMAGE = 'node:22.16.0'
        FE_IMAGE  = "${BE_IMAGE}"

        BE_CONTAINER_NAME = "be-${BUILD_NUMBER}"
        FE_CONTAINER_NAME  = "fe-${BUILD_NUMBER}"
        TEST_CONTAINER = "devtest-${BUILD_NUMBER}"
        
        COMPOSE_PROJECT_NAME="pipeline_${BUILD_NUMBER}"
        DOCKER_VOLUME="src-code-${BUILD_NUMBER}"

        // Backend environment variables
        NODE_ENV='development'
        CLIENT_ORIGIN="http://${FE_CONTAINER_NAME}:${FE_PORT}"

        DB_HOST="${COMPOSE_PROJECT_NAME}-postgres-1"
        DB_PORT=5432
        DB_NAME='demosutdb'
        DB_USER='postgres'
        DB_PASSWORD='postgres'

        JWT_ACCESS_SECRET='devtest_access_secret'
        JWT_ACCESS_EXPIRES_IN='15m'
        JWT_REFRESH_EXPIRES_IN='7d'

        // Frontend environment variables
        VITE_API_URL="http://${BE_CONTAINER_NAME}:${BE_PORT}"

        // Test instance environment variables
        BE_URL="http://${BE_CONTAINER_NAME}:${BE_PORT}"
        FE_URL="http://${FE_CONTAINER_NAME}:${FE_PORT}"

        UNIT_JUNIT_PATH="reports/jest-junit.xml"
        API_JUNIT_PATH="reports/api-junit.xml"
        E2E_JUNIT_PATH="reports/e2e-junit.xml"

        PERF_SEEDED_USER_EMAIL='bxh@gmail.com'
        PERF_SEEDED_USER_PASSWORD='Huy123456'

        TIMESTAMP = "${System.currentTimeMillis()}"

        INFLUXDB_HOST = 'http://influxdb3:8181'
        INFLUXDB_DATABASE = 'devtest_metrics'
        INFLUXDB_TOKEN = credentials('influxdb-token')
    }

    stages {
        stage('Prepare Environment') {
            parallel {
                stage('Prepare Dev Instance') {
                    steps {
                        // Create a named volume
                        echo 'Creating named volume...'
                        echo '-----------------------------------'
                        bat "docker volume create ${DOCKER_VOLUME}"
                        echo 'Named volume created successfully.'
                        echo '-----------------------------------'

                        // Use a temporary container to clone the repository into the named volume
                        echo 'Cloning repository into named volume...'
                        echo '-----------------------------------'
                        echo "BRANCH:         ${BRANCH}"
                        echo '-----------------------------------'
                        bat "docker run --rm -v ${DOCKER_VOLUME}:/app ${BE_IMAGE} sh -c \"git clone -b ${BRANCH} https://github.com/bienxhuy/quotie.git /app/dev-instance\""
                        echo 'Repository cloned into named volume successfully.'
                        echo '-----------------------------------'
                    }
                }

                stage('Prepare Test Instance') {
                    steps {
                        echo 'Running Python-Chrome Docker image...'
                        echo '-----------------------------------'
                        echo "TEST_CONTAINER:               ${TEST_CONTAINER}"
                        echo "DOCKER_NETWORK:               ${DOCKER_NETWORK}"
                        echo "BE_URL:                       ${BE_URL}"
                        echo "FE_URL:                       ${FE_URL}"
                        echo "BROWSER:                      ${BROWSER}"
                        echo "FRAMEWORK_LOG_DIR:            ${FRAMEWORK_LOG_DIR}"
                        echo "FRAMEWORK_SCREENSHOT_DIR:     ${FRAMEWORK_SCREENSHOT_DIR}"
                        echo "UNIT_JUNIT_PATH:              ${UNIT_JUNIT_PATH}"
                        echo "API_JUNIT_PATH:               ${API_JUNIT_PATH}"
                        echo "E2E_JUNIT_PATH:               ${E2E_JUNIT_PATH}"
                        echo "PERF_SEEDED_USER_EMAIL:       ${PERF_SEEDED_USER_EMAIL}"
                        echo "PERF_SEEDED_USER_PASSWORD:    ${PERF_SEEDED_USER_PASSWORD}"
                        echo "PERFORMANCE_RUN_TIME:         ${PERFORMANCE_RUN_TIME}"
                        echo "PERFORMANCE_USERS:            ${PERFORMANCE_USERS}"
                        echo "PERFORMANCE_SPAWN_RATE:       ${PERFORMANCE_SPAWN_RATE}"
                        echo "TIMESTAMP:                    ${TIMESTAMP}"
                        echo "BUILD_NUMBER:                 ${env.BUILD_NUMBER}"
                        echo "BUILD_URL:                    ${env.BUILD_URL}"
                        echo "BRANCH:                       ${env.BRANCH}"
                        echo "AUTHOR:                       ${env.AUTHOR}"
                        echo "HOST:                         ${env.HOST}"
                        echo "INFLUXDB_HOST:                ${INFLUXDB_HOST}"
                        echo "INFLUXDB_DATABASE:            ${INFLUXDB_DATABASE}"
                        echo '-----------------------------------'
                        bat "docker run -d --name ${TEST_CONTAINER} --network ${DOCKER_NETWORK} -e BE_URL=${BE_URL} -e FE_URL=${FE_URL} -e BROWSER=${BROWSER} -e LOG_DIR=${FRAMEWORK_LOG_DIR} -e SCREENSHOT_DIR=${FRAMEWORK_SCREENSHOT_DIR} -e UNIT_JUNIT_PATH=${UNIT_JUNIT_PATH} -e API_JUNIT_PATH=${API_JUNIT_PATH} -e E2E_JUNIT_PATH=${E2E_JUNIT_PATH} -e PERF_SEEDED_USER_EMAIL=${PERF_SEEDED_USER_EMAIL} -e PERF_SEEDED_USER_PASSWORD=${PERF_SEEDED_USER_PASSWORD} -e PERFORMANCE_RUN_TIME=${PERFORMANCE_RUN_TIME} -e PERFORMANCE_USERS=${PERFORMANCE_USERS} -e PERFORMANCE_SPAWN_RATE=${PERFORMANCE_SPAWN_RATE} -e TIMESTAMP=${TIMESTAMP} -e INFLUX_HOST=${INFLUXDB_HOST} -e INFLUX_TOKEN=%INFLUXDB_TOKEN% -e INFLUX_DATABASE=${INFLUXDB_DATABASE} -e BUILD_NUMBER=${env.BUILD_NUMBER} -e BUILD_URL=${env.BUILD_URL} -e BRANCH=${env.BRANCH} -e AUTHOR=${env.AUTHOR} -e HOST=${env.HOST} -w /app pythonwdriver:chrome tail -f /dev/null"
                        echo 'Test instance container started successfully.'
                        echo '-----------------------------------'

                        echo 'Cloning test framework repository...'
                        echo '-----------------------------------'
                        echo "TEST_CONTAINER:    ${TEST_CONTAINER}"
                        echo '-----------------------------------'
                        bat "docker exec ${TEST_CONTAINER} git clone -b main https://github.com/bienxhuy/devtest.git /app/devtest"
                        echo 'Test framework repository cloned successfully.'
                        echo '-----------------------------------'
                    }
                }
            }
        }

        stage('Prepare Containers') {
            parallel {
                stage('Prepare Backend Container') {
                    steps {
                        echo 'Pulling Node.js Docker image...'
                        echo '-----------------------------------'
                        bat "docker pull ${BE_IMAGE}"
                        echo 'Node.js image pulled successfully.'
                        echo '-----------------------------------'


                        // Run Node.js container in the existing network
                        echo 'Starting backend instance container...'
                        echo '-----------------------------------'
                        echo "DOCKER_NETWORK:         ${DOCKER_NETWORK}"
                        echo "DOCKER_VOLUME:          ${DOCKER_VOLUME}"
                        echo "BE_CONTAINER_NAME:      ${BE_CONTAINER_NAME}"
                        echo "NODE_ENV:               ${NODE_ENV}"
                        echo "CLIENT_ORIGIN:          ${CLIENT_ORIGIN}"
                        echo "DB_HOST:                ${DB_HOST}"
                        echo "DB_PORT:                ${DB_PORT}"
                        echo "DB_NAME:                ${DB_NAME}"
                        echo "DB_USER:                ${DB_USER}"
                        echo "DB_PASSWORD:            ${DB_PASSWORD}"
                        echo "JWT_ACCESS_SECRET:      ${JWT_ACCESS_SECRET}"
                        echo "JWT_ACCESS_EXPIRES_IN:  ${JWT_ACCESS_EXPIRES_IN}"
                        echo "JWT_REFRESH_EXPIRES_IN: ${JWT_REFRESH_EXPIRES_IN}"
                        echo '-----------------------------------'
                        bat "docker run -d --name ${BE_CONTAINER_NAME} --network ${DOCKER_NETWORK} -v ${DOCKER_VOLUME}:/app -v /app/dev-instance/backend/node_modules -e NODE_ENV=${NODE_ENV} -e CLIENT_ORIGIN=${CLIENT_ORIGIN} -e DB_HOST=${DB_HOST} -e DB_PORT=${DB_PORT} -e DB_NAME=${DB_NAME} -e DB_USER=${DB_USER} -e DB_PASSWORD=${DB_PASSWORD} -e JWT_ACCESS_SECRET=${JWT_ACCESS_SECRET} -e JWT_ACCESS_EXPIRES_IN=${JWT_ACCESS_EXPIRES_IN} -e JWT_REFRESH_EXPIRES_IN=${JWT_REFRESH_EXPIRES_IN} -w /app ${BE_IMAGE} tail -f /dev/null"
                        echo 'Backend instance container started successfully.'
                        echo '-----------------------------------'

                        // Install backend dependencies, run migrations and start the server
                        echo 'Setting up local CDN for backend...'
                        echo '-----------------------------------'
                        bat "docker exec ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm set registry http://verdaccio:4873/\""

                        echo 'Installing backend dependencies...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm install\""
                        echo 'Backend dependencies installed successfully.'
                        echo '-----------------------------------'


                        echo 'Extracting DB configuration to Host...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo "WORKSPACE:          ${WORKSPACE}"
                        echo '-----------------------------------'
                        bat "mkdir %WORKSPACE%\\db-config"
                        bat "docker cp ${BE_CONTAINER_NAME}:/app/dev-instance/backend/db %WORKSPACE%\\db-config"

                        echo 'Starting DB from Host...'
                        echo '-----------------------------------'
                        echo "COMPOSE_PROJECT_NAME: ${COMPOSE_PROJECT_NAME}"
                        echo '-----------------------------------'
                        bat "cd /d %WORKSPACE%\\db-config\\db && set \"COMPOSE_PROJECT_NAME=${COMPOSE_PROJECT_NAME}\" && docker compose up -d"
                        
                        // Sleep for 20 seconds to ensure the database is up before running migrations
                        sleep(time: 20, unit: 'SECONDS')

                        echo 'Running database migrations...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm run migration:run\""
                        echo 'Database migrations completed successfully.'
                        echo '-----------------------------------'

                        echo 'Extracting seed.sql from Volume to Host...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo "WORKSPACE:          ${WORKSPACE}"
                        echo '-----------------------------------'
                        bat "docker cp ${BE_CONTAINER_NAME}:/app/dev-instance/backend/db/seed.sql %WORKSPACE%\\seed.sql"

                        
                        echo 'Injecting seed data into Postgres Container...'
                        echo '-----------------------------------'
                        echo "DB_HOST:           ${DB_HOST}"
                        echo '-----------------------------------'
                        bat "docker exec -i ${DB_HOST} psql -U postgres -d demosutdb < %WORKSPACE%\\seed.sql"

                        echo 'Building application...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm run build\""
                        echo 'Application built successfully.'
                        echo '-----------------------------------'

                        echo 'Starting backend server...'
                        echo '-----------------------------------'
                        echo "BE_CONTAINER_NAME:  ${BE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec -d ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm run start\""
                        // Sleep for 10 seconds to ensure the server is up before proceeding
                        sleep(time: 10, unit: 'SECONDS')
                        echo 'Backend server started successfully.'
                        echo '-----------------------------------'
                    }
                }

                stage('Prepare Frontend Container') {
                    steps {
                        echo 'Pulling Node.js Docker image...'
                        echo '-----------------------------------'
                        bat "docker pull ${FE_IMAGE}"
                        echo 'Node.js image pulled successfully.'
                        echo '-----------------------------------'

                        echo 'Starting frontend instance container...'
                        echo '-----------------------------------'
                        echo "DOCKER_NETWORK:         ${DOCKER_NETWORK}"
                        echo "FE_CONTAINER_NAME:      ${FE_CONTAINER_NAME}"
                        echo "VITE_API_URL:           ${VITE_API_URL}"
                        echo '-----------------------------------'
                        bat "docker run -d --name ${FE_CONTAINER_NAME} --network ${DOCKER_NETWORK} -v ${DOCKER_VOLUME}:/app -v /app/dev-instance/frontend/node_modules -e VITE_API_URL=${VITE_API_URL} -w /app ${FE_IMAGE} tail -f /dev/null"
                        echo 'Frontend instance container started successfully.'
                        echo '-----------------------------------'

                        echo 'Setting up local CDN for frontend...'
                        echo '-----------------------------------'
                        bat "docker exec ${FE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/frontend && npm set registry http://verdaccio:4873/\""

                        echo 'Installing frontend dependencies...'
                        echo '-----------------------------------'
                        echo "FE_CONTAINER_NAME:  ${FE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec ${FE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/frontend && npm install\""
                        echo 'Frontend dependencies installed successfully.'
                        echo '-----------------------------------'

                        echo 'Building frontend application...'
                        echo '-----------------------------------'
                        echo "FE_CONTAINER_NAME:  ${FE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec ${FE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/frontend && npm run build\""
                        echo 'Frontend application built successfully.'
                        echo '-----------------------------------'

                        echo 'Starting frontend server...'
                        echo '-----------------------------------'
                        echo "FE_CONTAINER_NAME:  ${FE_CONTAINER_NAME}"
                        echo '-----------------------------------'
                        bat "docker exec -d ${FE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/frontend && npm run preview -- --host ${FE_CONTAINER_NAME}\""
                        // Sleep for 10 seconds to ensure the server is up before proceeding
                        sleep(time: 10, unit: 'SECONDS')
                        echo 'Frontend server started successfully.'
                        echo '-----------------------------------'
                    }
                }

                stage('Prepare Test Container') {
                    steps {
                        echo 'Installing Python dependencies...'
                        echo '-----------------------------------'
                        echo "TEST_CONTAINER:    ${TEST_CONTAINER}"
                        echo '-----------------------------------'
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pip install --index-url http://devpi:3141/root/pypi/+simple/ --trusted-host devpi --timeout 120 --retries 10 -r requirements.txt\""
                        echo 'Python dependencies installed successfully.'
                        echo '-----------------------------------'
                    }
                }
            }
        }

        
        stage('Run Unit Tests') {
            steps {
                echo 'Running backend unit tests...'
                echo '-----------------------------------'
                bat "docker exec ${BE_CONTAINER_NAME} bash -c \"cd /app/dev-instance/backend && npm run test || true\""
                echo 'Unit tests completed.'
                echo '-----------------------------------'

                // Copy unit test result to test framework container
                echo 'Copying unit test result to test framework container...'
                echo '-----------------------------------'
                bat "docker cp ${BE_CONTAINER_NAME}:/app/dev-instance/backend/reports/jest-junit.xml %WORKSPACE%\\unit-junit.xml"
                // Create reports directory in the test container if it doesn't exist
                bat "docker exec ${TEST_CONTAINER} bash -c \"mkdir -p /app/devtest/reports\""
                bat "docker cp %WORKSPACE%\\unit-junit.xml ${TEST_CONTAINER}:/app/devtest/${UNIT_JUNIT_PATH}"
                echo 'Unit test result copied successfully.'
                echo '-----------------------------------'
            }
        }

        stage('Execute Test') {
            steps {
                echo 'Starting test execution...'
                echo '-----------------------------------'
                echo "TEST_CONTAINER:    ${TEST_CONTAINER}"
                echo "RUN_TYPE:          ${RUN_TYPE}"
                echo "API_JUNIT_PATH:    ${API_JUNIT_PATH}"
                echo "E2E_JUNIT_PATH:    ${E2E_JUNIT_PATH}"
                echo '-----------------------------------'
                // Execute tests based on run type
                script {
                    if (env.RUN_TYPE == 'push') {
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest tests/api --junitxml=${API_JUNIT_PATH} || true\""
                    } else {
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest tests/api --junitxml=${API_JUNIT_PATH} || true\""
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && pytest tests/e2e --junitxml=${E2E_JUNIT_PATH} || true\""
                        bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && locust -f tests/performance/locustfile.py --headless -u ${PERFORMANCE_USERS} -r ${PERFORMANCE_SPAWN_RATE} -t ${PERFORMANCE_RUN_TIME} || true\""
                    }
                }
                echo 'Test execution completed.'
                echo '-----------------------------------'
            }
        }

        stage('Post Execution') {
            steps {
                echo 'Starting post execution script...'
                echo '-----------------------------------'
                echo "TEST_CONTAINER:    ${TEST_CONTAINER}"
                echo "RUN_TYPE:          ${RUN_TYPE}"
                echo '-----------------------------------'
                // Run post execution script
                bat "docker exec ${TEST_CONTAINER} bash -c \"cd /app/devtest && RUN_TYPE=${RUN_TYPE} python -m core.postexec_main\""
                echo 'Post execution script completed.'
                echo '-----------------------------------'
            }
        }
    }

    post {
        // Always perform cleanup actions after all stages
        cleanup {
            echo 'Pipeline finished.'
            echo '-----------------------------------'
            echo 'Stopping services, archiving and cleaning up...'
            echo '-----------------------------------'
            echo "BE_CONTAINER_NAME:    ${BE_CONTAINER_NAME}"
            echo "FE_CONTAINER_NAME:    ${FE_CONTAINER_NAME}"
            echo "TEST_CONTAINER:       ${TEST_CONTAINER}"
            echo '-----------------------------------'

            // Copy artifacts from test container to Jenkins workspace before stopping the container
            echo 'Copying logs and screenshots from test container to Jenkins workspace...'
            echo '-----------------------------------'
            catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                bat "mkdir %WORKSPACE%\\devtest"
                bat "docker cp ${TEST_CONTAINER}:/app/devtest/reports %WORKSPACE%\\devtest\\reports"
            }

            // Stop and remove the database compose stack
            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "cd /d %WORKSPACE%\\db-config\\db && set \"COMPOSE_PROJECT_NAME=${COMPOSE_PROJECT_NAME}\" && docker compose down -v"
            }

            // Stop and remove the dev and test containers
            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker stop ${BE_CONTAINER_NAME} || echo 'Error stopping ${BE_CONTAINER_NAME}'"
                bat "docker rm -v ${BE_CONTAINER_NAME} || echo 'Error removing ${BE_CONTAINER_NAME}'"
            }
            echo 'Backend instance container stopped and removed.'
            echo '-----------------------------------'

            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker stop ${FE_CONTAINER_NAME} || echo 'Error stopping ${FE_CONTAINER_NAME}'"
                bat "docker rm -v ${FE_CONTAINER_NAME} || echo 'Error removing ${FE_CONTAINER_NAME}'"
            }
            echo 'Frontend instance container stopped and removed.'
            echo '-----------------------------------'

            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker stop ${TEST_CONTAINER} || echo 'Error stopping ${TEST_CONTAINER}'"
                bat "docker rm -v ${TEST_CONTAINER} || echo 'Error removing ${TEST_CONTAINER}'"
            }
            echo 'Test instance container stopped and removed.'
            echo '-----------------------------------'

            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat "docker volume rm ${DOCKER_VOLUME} || echo 'Volume already removed'"
            }

            // Archive logs, screenshots, parse test result temporary
            echo 'Archiving logs, screenshots and parsing test result temporary.'
            echo '-----------------------------------'
            catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                archiveArtifacts artifacts: "devtest/${FRAMEWORK_LOG_DIR}/**", allowEmptyArchive: true
            }
            catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                archiveArtifacts artifacts: "devtest/${FRAMEWORK_SCREENSHOT_DIR}/**", allowEmptyArchive: true
            }
            echo 'Done archieving/parsing.'
            echo '-----------------------------------'

            // Clean Jenkins workspace
            cleanWs()
            echo 'Workspace cleaned.'
            echo '-----------------------------------'
            echo 'Pipeline done.'
        }
    }
}
