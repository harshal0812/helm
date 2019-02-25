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
             
    7.  What is .Release.Name ---
    
            Helm tracks all releases. Release is an built in object (like a class). It renders several parameters like 
            
            Release.Name
            
            Release.Namespace
            
            Release.Revision .. and so on. 
            
            name is the unique Random name that helm assigns to each release of the package
    
    8.  Run helm in dry run mode - 
        
        helm install --debug --dry-run ./nginxdemo2/
        
        Observations -
        
              name: washed-wolverine-configmap
        
        Change the files inside template directory and replace {{ .Release.Name }}-configmap with {{ .Release.Namespace }}-configmap
        
        Execute - helm install --debug --dry-run ./nginxdemo2/
        
        Observations - 
        
              name: default-configmap
    
    9.  helm install ./nginxdemo2
    
    10. Verify Installation
    
    11. helm list -- to get list of current releases
    
    12. helm delete RELEASE_NAME
    

##  Working with values.yaml

    1.  helm create nginxdemo3
    
    2.  rm -fr nginxdemo3/templates/*
    
    3.  cp nginxdemo2/templates/* nginxdemo3/templates/

    4.  cd nginxdemo3
    
    5.  vi values.yaml and remove all data from  the file. 
    
    6.  Add the below values to value.yaml
    
        nginximage: nginx:latest
        replicas: 3
        ports:
          containerport: 80
    
    7.  Make the below changes inside templates/nginx-deployment.yaml
    
            replicas: 1 to replicas: {{ .Values.replicas }}
            
            image: nginx to {{ .Values.nginximage }}
            
            - containerPort: 80 to - containerPort: {{ .Values.ports.containerport }}
            
    8.  helm install --debug --dry-run ./nginxdemo3
    
    9.  Verify the values of the manifest 

    10. helm install nginxdemo3

    11. helm list 
    
    12. helm delete RELEASE_NAME



##  Using helm Template Functions and Pipe operator 

    1.  Template functions are used to transform the supplied data from Values.yaml and make it more usable 
    
    2.  cp -R nginxdemo3  nginxdemo4
    
    3.  Add the below entry inside nginxdemo4/values.yaml
        
        nginxname: nginx

    3.  cd nginxdemo4/templates
    
    4.  Modify the below in nginx-deployment.yaml - 
    
            - name: nginx   to   - name: {{ quote .Values.nginxname }}
            
    5.  helm install --debug --dry-run nginxdemo4
        
        Observations: 
        
            - name: "nginx"   ----  quotes added 
    
    6.  Modify the below in nginx-deployment.yaml -
    
            - name: {{ quote .Values.nginxname }}    to   - name: {{  .Values.nginxname | upper | quote }}
            
    7.  helm install --debug --dry-run nginxdemo4
    
        Observations:
            
            - name: "NGINX"
            
    8.  Modify the below in nginx-deployment.yaml -
    
            - name: {{  .Values.nginxname | upper | quote }}  to   - name: {{  .Values.nginxname | upper | repeat 5 | quote }}
            
    9.  helm install --debug --dry-run nginxdemo4
    
            Observations:
            
            - name: "NGINXNGINXNGINXNGINXNGINX"
            
    9.  Modify the below in nginx-deployment.yaml -
    
            image: {{ .Values.nginximage }}    to    image: {{ .Values.nginximageNOTEXISTING | default "nginx:1.15.8" | quote }}
            
    10. helm install --debug --dry-run nginxdemo4
    
            Observations:
            
            image: "nginx:1.15.8"
            

##  Control Structures and Loops 

    1.  cp -R nginxdemo4 nginxdemo5
    
    2.  Edit values.yaml file as below - 
    
        nginximage: nginx:latest 
        
                    TO
        
        nginximagename: nginx
        nginximagetag: latest
        
    3.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below 
    
            image: {{ .Values.nginximageNOTEXISTING | default "nginx:1.15.8" | quote }}
            
                                    TO
                                    
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }} 
            
    4.  helm install --debug --dry-run nginxdemo5
    
        Observations:
            
                    image: nginx:latest

    
    5.  Modify values.yaml as below 
    
            nginximagetag: latest  to  nginximagetag: 1.15.4
    
    5.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below 
    
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }} 
            
                                    TO
                                    
            {{ if eq .Values.nginximagename "1.15.8" }}
            image: {{ .Values.nginximagename }}:{{ .Values.nginximagetag }}
            {{ else }}
            image: nginx:latest
            {{ end }}
    
    6.  helm install --debug --dry-run nginxdemo5
    
        Observations: 
            
            image: nginx:latest
            
    7.  Add below list to values.yaml
    
            deploymentlabels:
              - app: frontend
              - tier: webtier
              - application: nginx
              
    8.  Modify - nginxdemo5/templates/nginx-deployment.yaml as below
            
            metadata:
              name: nginx
              
              
                                 TO
                                 
            metadata:
              name: nginx
              labels:
                {{- range $key, $val := .Values.deploymentlabels }}
                {{ $key }}: {{ $val | quote }}
                {{- end }}

    9.  helm install --debug --dry-run nginxdemo5
    
        Observations:
        
            metadata:
              name: nginx
              labels:
                app: "frontend"
                application: "nginx"
                tier: "webtier"

              



         
    
        
    
    

    
        
    

   
     




