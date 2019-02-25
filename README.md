# helm

# HELM - The Package Manager for Kubernetes 

##  Helm installation 
    
    a.  Install Helm package 
    
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
    
    root@ip-172-31-33-248:~# which helm
    /usr/local/bin/helm
    
    root@ip-172-31-33-248:~# which tiller
    /usr/local/bin/tiller
    
    b.  Configure Tiller
    
    Helm installs the tiller service on your cluster to manage charts.
    
    Since RBAC is our official mode of securing access, we will create RBAC policies for tiller to manage deployments
    
    kubectl -n kube-system create serviceaccount tiller

    kubectl create clusterrolebinding tiller \
     --clusterrole=cluster-admin \
     --serviceaccount=kube-system:tiller
     
    helm init --service-account tiller
    
    root@ip-172-31-33-248:~# kubectl get pods -n kube-system | grep -i tiller
    tiller-deploy-689d79895f-75swz                         1/1     Running   0          15s
    
##  Helm Charts 

    Structure of Helm Chart -
     
    mychart/
       Chart.yaml
       values.yaml
       charts/
       templates/
    
    1.  The templates/ directory is for template files. 
    
        When Tiller evaluates a chart, it will send all of the files in the templates/ directory through the template rendering engine. 
    
        Tiller then collects the results of those templates and sends them on to Kubernetes. 
       
    2.  The values.yaml file is also important to templates. 
    
        This file contains the default values for a chart.
        
        These values may be overridden by users during helm install or helm upgrade
        
    3.  The Chart.yaml file contains a description of the chart.
    
    4.  The charts/ directory may contain other charts or subcharts. 
    

##  Creating our First Chart 

    We will deploy an nginx application - 
    
      1.  nginx - deployed as a deployment 
      
      2.  nginx.conf and virtualhost.conf deployed as configmap 
      
      3.  Add configmap as volume to nginx.conf 
    
    
    a.  helm create nginxdemo

    b.  Files inside nginxdemo/templates
        
        1.  NOTES.txt - The “help text” for your chart. This will be displayed to your users when they run helm install.
        
        2.  _helpers.tpl: A place to put template helpers that you can re-use throughout the chart
    
        3.  rm -fr *
        
        4.  cp nginx* nginxdemo/templates/
        
        5.  echo "You are installing NGINX with CONFIGMAP as a volume" >  nginxdemo/templates/NOTES.txt
        
        6.  Note the random name - NAME:   killer-liger
        
        7.  helm get manifest killer-liger
        
        8.  helm delete killer-liger


##  Adding variables to HELM 

    We will recreate our nginx application to add some new variables 
    
    1.  helm create nginxdemo2
    
    2.  cd nginxdemo2/templates
    
    3.  rm -fr * 
    
    4.  cp nginx* nginxdemo2/templates/
    
    5.  cd nginxdemo2/templates/
    
    6.  vi nginx-configmap.yaml nginx-deployment.yaml
    
        a. nginx-configmap.yaml
        
            Modify name: nginx-conf to name: {{ .Release.Name }}-configmap
            
        b.  nginx-deployment.yaml
        
             Modify name: nginx-conf to {{ .Release.Name }}-configmap
             
    7. 
    

   
     




