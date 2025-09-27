pipeline {
	agent any

	stages {
		stage('測試') {
			steps {
				script {
					currentBuild.displayName = "#${env.BUILD_NUMBER} - 測試job"
					currentBuild.description = "multiFilepath=${params.multiFilepath}"
					build(
						job: 'newfile',
						parameters: [
							string(name: 'WS_NAME', value: "test"),
							string(name: 'multiFilepath', value: params.multiFilepath),
						],
						propagate: true,      // 下游失敗時是否讓本工作也失敗
						quietPeriod: 0        // 立即觸發，不延遲
					)
				}
			}
		}
		stage('Cat txt') {
			steps {
				script {
					def p = "/var/jenkins_home/workspace/newfile/test/${params.multiFilepath}"
					echo "Target: ${p}"
					if (!fileExists(p)) {
						error "找不到檔案：${p}"
					}
					def content = readFile(p).trim()
					echo "txt 內容：\n${content}"
				}
			}
		}

		stage('Fetch & Place') {
			steps {
				// 把 newfile 的最後一次成功建置的 ${env}/aaa.txt 拉到目前工作區
				copyArtifacts projectName: 'newfile',
					selector: lastSuccessful(),
					filter: "${params.env}/aaa.txt",
					target: 'tmp',
					flatten: true

				// 放到你想要的位置（通常建議放在「本 job 工作區」，避免寫入別的 job 的 workspace）
				sh """
          set -euo pipefail
          mkdir -p newfile
          mv -f tmp/aaa.txt newfile/aaa.txt
        """
			}
		}
	}
}
