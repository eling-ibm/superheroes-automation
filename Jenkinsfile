pipeline {
    agent any

    parameters {
        string(
            name: 'SUPERHEROES_REPO',
            defaultValue: 'https://github.com/quarkusio/quarkus-super-heroes.git',
            description: 'Git repository URL of the Superheroes application.'
        )
        string(
            name: 'SUPERHEROES_BRANCH',
            defaultValue: 'main',
            description: 'Git branch, tag, or commit SHA to check out from the Superheroes repository.'
        )
        choice(
            name: 'MODE',
            choices: ['native', 'jvm'],
            description: 'Superheroes container image mode to use.'
        )
        choice(
            name: 'BENCHMARK',
            choices: [
                'get-all-heroes',
                'get-all-villains',
                'get-random-hero',
                'get-random-villain',
                'perform-fights',
                'first-fight',
                'first-random-hero',
                'first-random-villain'
            ],
            description: 'Benchmark scenario to run.'
        )
        choice(
            name: 'DRIVER',
            choices: ['hyperfoil', 'loop'],
            description: 'Load driver to use.'
        )
        choice(
            name: 'LOCATION',
            choices: ['local', 'remote'],
            description: 'Where to run the services (local = localhost, remote = servers defined in envs/remote.env.yaml).'
        )
        string(
            name: 'BENCHMARK_PARAMS',
            defaultValue: '',
            description: 'Additional qDup -S overrides, e.g. -S HF_BENCHMARK_PARAMS="-PDURATION=20s"'
        )
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Validate') {
            steps {
                sh '''
                    echo "=== Benchmark configuration ==="
                    echo "  Superheroes repo: ${SUPERHEROES_REPO}"
                    echo "  Superheroes branch: ${SUPERHEROES_BRANCH}"
                    echo "  Mode:             ${MODE}"
                    echo "  Benchmark:        ${BENCHMARK}"
                    echo "  Driver:           ${DRIVER}"
                    echo "  Location:         ${LOCATION}"
                    echo "  Benchmark params: ${BENCHMARK_PARAMS}"

                    [ -f "modes/${MODE}.script.yaml" ] || \
                        { echo "ERROR: modes/${MODE}.script.yaml not found"; exit 1; }

                    [ -d "benchmarks/${BENCHMARK}" ] || \
                        { echo "ERROR: benchmarks/${BENCHMARK} not found"; exit 1; }

                    [ -f "drivers/${DRIVER}.yaml" ] || \
                        { echo "ERROR: drivers/${DRIVER}.yaml not found"; exit 1; }
                '''
            }
        }

        stage('Run Load Test') {
            steps {
                sh '''
                    REPO_PARAMS="-S SUPERHEROES_REPO=${SUPERHEROES_REPO} -S SUPERHEROES_COMMIT=${SUPERHEROES_BRANCH}"
                    ./run.sh "${MODE}" "${BENCHMARK}" "${DRIVER}" "${LOCATION}" "${REPO_PARAMS} ${BENCHMARK_PARAMS}"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'report-output/**', allowEmptyArchive: true
        }
        success {
            echo 'Load test completed successfully.'
        }
        failure {
            echo 'Load test failed. Check the archived report-output for details.'
        }
    }
}
