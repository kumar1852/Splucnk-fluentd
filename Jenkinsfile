node("master")
{

   properties([
         parameters([
           string(name: 'CONFIG', description:'Absolute path to the kubeconfig'),
           string(name: 'DOCKER_IMAGE', description:'Docker Image for fluentd'),
           string(name: 'FLUENT_SPLUNK_HOST', description:'Splunk Host'),
           string(name: 'FLUENT_SPLUNK_TOKEN', description:'Splunk Token'),
           string(name: 'SPLUNK_INDEX', description:'Splunk Index'),
           string(name: 'CLUSTER_NAME', description:'A unique identifier for the cluster'),
           string(name: 'k8sversion', defaultValue: '1.8', description:'Either 1.7 or 1.8'),
           string(name: 'environment', defaultValue: 'engprod', description:'environment engprod, prod,stage, dev'),
         ])
       ])
    def dstemplate

    stage "checkout scm"
         checkout scm


    stage "Update config files"
        if(k8sversion=="1.7"){
            dstemplate="fluentd-ds-nonrbac-template.yaml"
        }else{
            dstemplate="fluentd-ds-rbac-template.yaml"
        }
        def dt = sh(script: "date +%s", returnStdout: true).toString().trim()
        sh("sed -i.bak -e 's#__DOCKER_IMAGE__#${DOCKER_IMAGE}#' configs/$dstemplate")
        sh("sed -i.bak -e 's#__FLUENT_SPLUNK_HOST__#${FLUENT_SPLUNK_HOST}#' configs/$dstemplate")
        sh("sed -i.bak -e 's#__FLUENT_SPLUNK_TOKEN__#${FLUENT_SPLUNK_TOKEN}#' configs/$dstemplate")
        sh("sed -i.bak -e 's#__SPLUNK_INDEX__#${SPLUNK_INDEX}#' configs/$dstemplate")
        sh("sed -i.bak -e 's#__CLUSTER_NAME__#${CLUSTER_NAME}#' configs/$dstemplate")
        sh("sed -i.bak -e 's#__ROLL__#${dt}#' configs/$dstemplate")

    stage "Deploy Fluentd"
        sh("sleep 5")
        sh("echo Deploying fluentd Configmap")
        //sh("kubectl --kubeconfig='${CONFIG}' version")

        sh("kubectl --kubeconfig='${CONFIG}' apply -f configs/fluentd-configmap-${environment}.yaml")
        sh("sleep 5")
        script
        {
            while (sh(script: "kubectl --kubeconfig='${CONFIG}' get configmaps fluentd-ds-config -n kube-system", returnStatus: true) != 0)
            {
                sh ("echo configmap fluentd-ds-config still not created...")
                sleep 5
            }
        }
        sh("echo Deploying fluentd Daemonset")
        sh("kubectl --kubeconfig='${CONFIG}' apply -f configs/$dstemplate")

        sh("echo fluentd daemonset deployed -- please use kubectl get pods command to validate if all the fluentd pods are up")

}
