pipeline {
    agent any
    stages {
        stage ('dumpDB') {
            steps {
                sh '''
                    ssh dnt3@10.0.2.15 <<EOF
                     mysqldump --column-statistics=0 --user=epsi --password="epsimysql" --socket='/opt/lampp/var/mysql/mysql.sock' --databases epsi > /var/www/dnt3/`date +%Y-%m-%d-%H`-dump.sql
                     exit
                    EOF
                '''
            }
        }
        stage ('clone + install') {
            steps {
                sh '''
                    ssh dnt3@10.0.2.15 <<EOF
                     cd /var/www/dnt3
                     rm -rf DNT3-test
                     git clone https://github.com/epsi-mat/DNT3.git DNT3-test
                     cd DNT3-test
                     echo -e "NODE_ENV=production\n\nMYSQL_DATABASE=epsitest\nMYSQL_USER=epsi\nMYSQL_PASSWORD=epsimysql\nMYSQL_ROOT_PASSWORD=rootepsimysql123\nMYSQL_HOST=localhost\nMYSQL_HOSTNAME=db\nMYSQL_PORT=3306\nMYSQL_SOCKET=/var/run/mysqld/mysqld.sock\nPORT=3001" > .env
                     npm install
                     exit
                    EOF
                '''
            }
        }
        stage ('Unit test + Integration test') {
            steps {
                sh '''
                    ssh dnt3@10.0.2.15 <<EOF
                     cd /var/www/dnt3/DNT3-test
                     pm2 start bin/www
                     jest test/unit.test.js --ci
                     jest test/commandes.test.js --ci
                     pm2 stop bin/www
                     exit
                    EOF
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    ssh dnt3@10.0.2.15 <<EOF
                     cd /var/www/dnt3/DNT3
                     git pull
                     npm install --production
                     exit
                    EOF
                '''
            }
        }
        stage('Smoke test production') {
            steps {
                sh '''
                    ssh dnt3@10.0.2.15 <<EOF
                     cd /var/www/dnt3/DNT3-test
                     jest test/smoke.test.js --ci
                     exit
                    EOF
                '''
            }
        }
    }  
}
