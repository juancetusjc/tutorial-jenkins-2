#!groovy

node {
   // ------------------------------------
   // -- ETAPA: Compilar
   // ------------------------------------
   stage 'Compilar'
   
   // -- Configura variables
   echo 'Configurando variables'
   def mvnHome = tool 'M3'
   env.PATH = "${mvnHome}/bin:${env.PATH}"
   echo "var mvnHome='${mvnHome}'"
   echo "var env.PATH='${env.PATH}'"
   
   // -- Descarga código desde SCM
   echo 'Descargando código de SCM'

   echo 'Ejecutando rm -rf *'
   sh 'rm -rf *'

   echo 'Paso Checkout'   

   checkout scm
   
   // -- Compilando
   echo 'Compilando aplicación'
   sh 'mvn clean compile'
   
   // ------------------------------------
   // -- ETAPA: Test
   // ------------------------------------
   stage 'Test'
   echo 'Ejecutando tests'
   try{
      sh 'mvn verify'
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   }catch(err) {
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      if (currentBuild.result == 'UNSTABLE')
         currentBuild.result = 'FAILURE'
      throw err
   }
   
   // ------------------------------------
   // -- ETAPA: Instalar
   // ------------------------------------
   stage 'Instalar'
   echo 'Instala el paquete generado en el repositorio maven'
   sh 'mvn install -Dmaven.test.skip=true'

   stage 'Deploy Nexus'
   echo 'Despliega el paquete generado en el repositorio Nexus'
   
   sshagent(credentials: ['oracle-pre-priv']) {
    //sh 'ssh -o StrictHostKeyChecking=no -l oracle 192.168.4.129 uname -a'    
    sh 'uname -a'    
  }

   //sh 'mvn clean deploy'

   // ------------------------------------
   // -- ETAPA: Archivar
   // ------------------------------------
   stage 'Archivar'
   echo 'Archiva el paquete el paquete generado en Jenkins'
   step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar, **/target/*.war', fingerprint: true])
}
