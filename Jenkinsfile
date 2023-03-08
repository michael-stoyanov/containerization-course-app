pipeline
{
    agent
    {
        label 'docker-node'
    }
    environment
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages
    {
        stage('Downloading repository')
        {
            steps
            {
                sh '''
                 if [ -d bgapp ]; then 
                      cd bgapp
                      git pull http://192.168.99.102:3000/gitea/bgapp 
                    else
                      git clone http://192.168.99.102:3000/gitea/bgapp 
                    fi
                '''
            }
        }
        stage('Create env file with proper data')
        {
            steps
            {
                sh '''
                    cd bgapp
                    if [[ ! -f .env ]]; then
                        var_path=$PWD
                        echo "PROJECT_ROOT=${var_path}/web" >> .env
                        echo 'DB_ROOT_PASSWORD=12345' >> .env
                    fi
                '''
            }
        }
    	stage('CleanUp existing containers')
        {
            steps
            {
                sh 'docker container rm -f db-bgapp web-bgapp || true'
                sh 'docker image rm -f maio246/bgapp-db-gitea maio246/bgapp-web-gitea || true'

            }
        }
        stage('Execute docker compose for images')
        {
            steps
            {
                sh '''
                cd bgapp
                docker compose up -d
                '''
            }
        }
        stage('Tests')
        {
            steps
            {
                echo 'Test #1 - Reachability'
                sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200'
                
                echo 'Test #2 - Sofia city Appears'
                sh "curl --silent http://localhost:8080 | grep София"
            }
        }
       	stage('CleanUp')
        {
            steps
            {
                sh 'docker container rm -f bgapp-db-1 bgapp-web-1 || true'
            }
			
        }
        stage('Login') 
        {
            steps 
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push the web image') 
        {
            steps 
            {
                sh '''
					docker image tag bgapp-web maio246/bgapp-web-gitea
					docker push maio246/bgapp-web-gitea
				'''
            }
        }
        stage('Push the db image') 
        {
            steps 
            {
                sh '''
					docker image tag bgapp-db maio246/bgapp-db-gitea
					docker push maio246/bgapp-db-gitea
				'''
            }
        }
		
		stage('Cleaning the containers')
		{
			steps
			{
				sh 'docker container rm -f web-bgapp db-bgapp || true'
			}
		}
		stage('Cleaning the images')
		{
			steps
			{
				sh 'docker image rm -f bgapp-web bgapp-db || true'
			}
		}
        stage('Deploying the images')
        {
            steps
            {
				
				sh 'docker container run -d -p 8080:80 --name web-bgapp -v /home/jenkins/workspace/bgapp/web:/var/www/html:ro --net bgapp_app-network maio246/bgapp-web-gitea'
				sh 'docker container run -d --name db-bgapp --net bgapp_app-network -e MYSQL_ROOT_PASSWORD=12345 maio246/bgapp-db-gitea'			
            }
        }
    }
}