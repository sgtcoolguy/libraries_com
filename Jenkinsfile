#! groovy
timestamps {
	// Need maven 3.3+
	node('edtftpjpro && linux && eclipse && jdk') {
		try {
			stage('Checkout') {
				checkout scm
			}

			stage('Dependencies') {
				sh 'cp -f $HOME/edtftpjpro/* $WORKSPACE/bundles/com.aptana.ide.libraries.subscription/'
			}

			stage('Build') {
				withEnv(["PATH+MAVEN=${tool name: 'Maven 3.5.0', type: 'maven'}/bin"]) {
					sh 'mvn clean verify'
				}
				dir('releng/com.aptana.ide.libraries.subscription.update/target') {
					archiveArtifacts artifacts: "repository/**/*"
					def jarName = sh(returnStdout: true, script: 'ls repository/features/com.aptana.ide.feature.libraries.subscription_*.jar').trim()
					def version = (jarName =~ /.*?_(.+)\.jar/)[0][1]
					currentBuild.displayName = "#${version}-${currentBuild.number}"
				}
			}

			// If not a PR, trigger studio3-core build for same branch
			if (!env.BRANCH_NAME.startsWith('PR-')) {
				build job: "Studio/studio3/${env.BRANCH_NAME}", wait: false
			}
		} catch (e) {
			// if any exception occurs, mark the build as failed
			currentBuild.result = 'FAILURE'
			throw e
		} finally {
			step([$class: 'WsCleanup', notFailBuild: true])
		}
	} // node
} // timestamps
