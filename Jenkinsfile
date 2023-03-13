podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    try
	{
     stage('Build a gradle project') {
	  git branch: 'main', url: 'https://github.com/vishalshende83/week6.git'
      container('gradle') {
        stage('Build a gradle project') {
          sh '''
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
        }

        stage('Unit Test') {
		if (env.BRANCH_NAME == 'feature' || env.BRANCH_NAME == 'master')
		 {
          echo "Unit Test for branch : ${env.BRANCH_NAME}"
		  sh '''
          ./gradlew test
          '''
		 }
		else
		 {
		  echo "Unit Test Skipped for branch : ${env.BRANCH_NAME}"
		  }
        }
		stage('Code Coverage') {
		if (env.BRANCH_NAME == 'master')
		 {
          echo 'Code Coverage for branch : ${env.BRANCH_NAME}'
		  sh '''
          ./gradlew jacocoTestCoverageVerification
          ./gradlew jacocoTestReport
          '''
		 }
		 else
		 {
		  echo "Code Coverage Skipped for branch : ${env.BRANCH_NAME}"
		 }
		
        }
		stage('Static code analysis') {
		if (env.BRANCH_NAME == 'feature' || env.BRANCH_NAME == 'master')
		 {
          echo "Static code analysis for branch : ${env.BRANCH_NAME}"
		  sh '''
          ./gradlew checkstyleMain
          ./gradlew jacocoTestReport
          '''
		 }
		 else
		 {
		  echo "Static code analysis Skipped for branch : ${env.BRANCH_NAME}"
		 }
		
       }
      }
	 }
	  try
	  {
	   stage('Build Java Image') {
       container('kaniko') {
        stage('Build Image and Push to Docker Repository') {
         if (env.BRANCH_NAME == 'feature')
         { 
		     sh '''
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                /kaniko/executor --context `pwd` --destination vishalshende83/calculator-feature:0.1
                '''
         }
         else
         {
          sh '''
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                /kaniko/executor --context `pwd` --destination vishalshende83/calculator-feature:0.1
                '''
         }
		  }
     }
    }
	 }
	 catch (Exception E) 
	 {
		echo 'Failure detected while Building Java Image'
	 }
	 
	}
	catch (Exception E) 
	{
		echo 'Failure detected while building a gradle project'
	}

    
  }
}