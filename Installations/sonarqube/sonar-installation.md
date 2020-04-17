Installation & setup sonarqube;
# REQUIREMENTS:
1. EC2 instance (t2.medium or t2.large)--3GB RAM  required for sonarqube
	<with required java version installed>
	<with required version of mysql-server installed> 
2. mysql database (for store the sonar scan details) in reality we used these one so we need it

# INSTALLATION:
1. login into the instance & excute
    $ sudo -i
    $ sudo apt-get update
2. Now goto sonarqube downloads page on google, then choose the version of sonarqube(7.7 or 6.7.7 or 6.6..etc);
then see the requirements for that particular version of sonarqube(java version & mysql srver versions), excute
    <in these we are installing sonarqube-6.7.7>
    $ sudo apt-get install openjdk-8-jdk -y
    $ sudo apt-get install mysql-server -y
    $ sudo apt-get install unzip -y
    $ cd /opt/
    $ wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-6.7.7.zip
    $ unzip sonarqube-6.7.7.zip
    $ mv sonarqube-6.7.7 sonarqube
3. sonarqube was not run with root so we need a user to run it
    $ useradd <username>(sonaradmin)
    $ passwd <username>(sonaradmin)
    Now give user to owner nd other permissions;
    $ chown -R <ownername>:<groupname> <directory>
    $ chown -R sonaradmin:sonaradmin /opt/sonarqube/
    $ ls -l
    $ chmod -R 775 /opt/sonarqube/
# MYSQL SETUP
1. goto aws copy sonarqubeserver private ip addres
2. goto RDS> click on database> goto security> in inbound rules paste the private ip address here
        <172.33.33.45/32>
3. now click on database copy the end point
        <.........amazon.com>
4. goto git>
    $ mysql -h <paste endpoint> -u <username> -p (give username while creating the database)
      <enter database password>
5. here we create a database and two users;
    $ CREATE DATABASE sonarqube;  (sonarqube is database name)
    $ CREATE USER sonarqube@'localhost' IDENTIFIED BY 'some_secure_password';
    $ CREATE USER sonarqube@'%' IDENTIFIED BY 'some_secure_password';
    $ GRANT ALL ON sonarqube.* to sonarqube@'localhost';
    $ GRANT ALL ON sonarqube.* to sonarqube@'%';
    $ show databases; (database name shown)
    $ SHOW Users FROM mysql-server;
    $ FLUSH PRIVILEGES;
    $ EXIT  (exit from the mysql console)
# sonarqube server setup
1. edit the /opt/sonarqube/conf/sonar.properties
    $ cd /opt/sonarqube/
    $ cd conf/
    $ ls (sonar.properties file shown)
    $ vi sonar.properties
    ''''''
            sonar.jdbc.username=sonarqube<username>
            sonar.jdbc.password=sonar123<database password>
            <remove '#'>
    ''''''
    ''''''
            <goto aws copy database endpoint>
            sonar.jdbc.url=jdbc:mysql://<database endpoint>:3306/sonarqube?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
            <remove '#' here>
    ''''''
    ''''''
        in webserver section;
        <remove '#'>
        web.server.host=0.0.0.0
        web.server.context=/sonar (these will connect to our sonarqube server by passing 'sonar)
        sonar port=9000
    ''''''
    exit vi
# To RUN sonarqube service
    $ su sonaradmin
    $ cd /opt/sonarqube/
    $ ls
    $ cd bin/
    $ cd linux-x86-64/
    $ ls (.sonar.sh shown)
    $ ./sonar.sh start
    $ ./sonar.sh status
goto aws take instance public ip nd paste it on google
    <ip:9000/sonar>
login>by deafault username is admin nd password also admin

# SONARQUBE SETUP & INTEGRATION WITH JENKINS
1. login into sonarqube server <ip:9000/sonar>   (for generate token)
    click on Administator>click on my account>goto security>give some name(test)>click on generate token>copy the token number
2. login into jenkins
    goto credentials>click on add credentials>click on domain section select 'secret text'>paste the token number at id>save
3. install sonarqube scanner pluggin
    goto manage jenkins>manage plugins>click on available>search sonar>click on sonaqube scanner>install without restart
SETUP;
    goto manage jenkins>configure jenkins>click on add sonarqube server>
                                                        NAME: SONAR-6.7.7
                                                        URL:  http://ip add:9000/sonar
                                                        TOKEN: Choose secret text
    goto manage jenkins>global tool configuration> click on add sonarqube scanner
                                                        NAME: sonar_scannner
                                                        RUN_HOME: /opt/sonarqube/
INTEGRETION;
Create a job
pipeline script
''''''''
        node {

   stage('SCM') {
	  git 'https://github.com/spring-projects/spring-petclinic.git'
   }
   
   stage ('build the packages') {
	  sh 'mvn package'
   }
   
   stage('SonarQube analysis') {
    // performing sonarqube analysis with "withSonarQubeENV(<Name of Server configured in Jenkins>)"
    withSonarQubeEnv('SONAR-6.7.4') {
      // requires SonarQube Scanner for Maven 3.2+
      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
    }
  }

}
''''''''
click on build
click on sonarscan symbol then we show the scan details