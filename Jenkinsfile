node {
	checkout scm

	def jobs = [:]

	toxenv = sh(script: 'tox --listenvs', returnStdout: true).trim().split('\n')
	toxenv.each {
		item ->
			jobs[item] = {
				stage('Preprare venv') {
					catchError(buildResult: 'NOT_BUILT', stageResult: 'FAILURE') {
						sh(script: 'export TMPDIR=/var/tmp; tox -r -e ' + item)
					}
				}
				try {
					stage('Test') {
						catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
							sh(script: 'tox -e ' + item + ' -- test')
						}
					}
				}
				finally {
						sh(script: 'tox -e ' + item + ' -- destroy')
				}
			}
	}

	parallel(jobs)
}
