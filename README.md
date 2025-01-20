# K8s-Prometheus-Grafana
Monitoring K8s with the Kubernetes Dashboard, Prometheus, and Grafana

hands-on lab
Monitoring K8s with the Kubernetes Dashboard, Prometheus, and Grafana
 
1 Logging In to the Amazon Web Services Console
Introduction
This lab experience involves Amazon Web Services (AWS), and you will use the AWS Management Console to complete the instructions in the following lab steps.
The AWS Management Console is a web control panel for managing all your AWS resources, from EC2 instances to SNS topics. The console enables cloud management for all aspects of the AWS account, including managing security credentials and even setting up new IAM Users.
 
 
 
Enter the following credentials created just for your lab session, and click Sign in:
Account ID or alias: 021018722059
IAM user name: student
Password: DUhjO_jC1k15wE
https://021018722059.signin.aws.amazon.com/console?region=us-west-2

3. Select the US West (Oregon) us-west-2 region using the upper-right drop-down menu on the AWS Management Console:
 
 
Amazon Web Services is available in different regions all over the world, and the console lets you provision resources across multiple regions. You usually choose a region that best suits your business needs to optimize your customer’s experience, but you must use the US West 2for this lab.
 
Recommendation: For an optimal lab experience, ensure that you are using a supported browser, opens in a new tab when accessing the AWS Management Console. For example, an up-to-date version of Google Chrome, opens in a new tab or Mozilla Firefox, opens in a new tab.
 
 
 
 
 
2 Connecting to the CloudAcademy Web based IDE
Introduction
In this Lab Step, you will use your local browser to navigate to the CloudAcademy web-based general-purpose IDE.
 
Instructions
1. In the AWS Management Console search bar, enter EC2, and click the EC2 result under Services:
 
2. Click on Instances. Wait for the ide.cloudacademy.platform.instance EC2 instance to be launched, and then select it, and locate and copy the assigned IPv4 Public IP address. An example IPv4 address number is 54.69.21.244
Notes: 
The following EC2 instances are launched in the given order:
k8s.cloudacademy.platform.instance
ide.cloudacademy.platform.instance
It takes approx. 2-15 minutes for both instances to become available - please be patient.
Click the refresh icon periodically to check that both instances have been created and are in a running state.
 
3. The web-based CloudAcademy IDE has been configured on port 80. Using your browser, navigate to the IDE hosted on the ide.cloudacademy.platform.instance EC2 instance using the public IP address you just copied: 
Note:
Remember to use the public IP address assigned to youride.cloudacademy.platform.instance. 
It takes approximately 1-2 minutes for the web-based CloudAcademy IDE service to startup on the ide.cloudacademy.platform.instance EC2 instance - try refreshing the browser page request until access is successful - please be patient.
It has been configured to listen on port 80
An example URL would be http://35.91.56.109, opens in a new tab where "35.91.56.109, opens in a new tab" is the public IP from the last instruction
 
 
Summary
In this Lab Step, you used your local browser to navigate to the CloudAcademy web-based general-purpose IDE.
 


 
 
 
3 Deploy a Simple API Application
0/2
Introduction
In this Lab Step, you'll deploy a simple Python Flask web based API into the lab provided Kubernetes cluster. The API has been instrumented to provide various metrics which will be collected by Prometheus for observability purposes. 
 
Instructions  
1. Expand the Files tree view by clicking on the Files tab on the left handside menu, and then open the project/code/api directory:
2. The api directory contains the following 3 files which have then been used to build the cloudacademydevops/api-metrics container image. Open each of the following files within the editor view and review their contents.
api.py
Dockerfile
requirements.txt
The api.py file contains the Python source code which implements the example API. In particular take note of the following:
Line 5 - imports a PromethusMetrics module to automatically generate Flask based metrics and provide them for collection at the default endpoint /metrics
Line 10-32 - implements 5 x API endpoints:
/one
/two
/three
/four
/error
All example endpoints, except for the error endpoint, introduce a small amount of latency which will be measured and observed within both Prometheus and Grafana.
The error endpoint returns an HTTP 500 server error response code, which again will be measured and observed within both Prometheus and Grafana.
The Docker container image containing this source code has already been built using the tag cloudacademydevops/api-metrics
3. Within the Files tab on the left handside menu, open the project/code/k8s directory and click on the api.yml file:
4. The api.yml file contains the Kubernetes resources that will be created for the API when deployed into the cluster. In particular note the following:
Lines 1-25: API Deployment containing 2 pods
Line 22: Pods are based off the container image cloudacademydevops/api-metrics
Lines 27-46: API Service - loadbalances traffic across the 2 API Deployment pods
Lines 34-37: API Service is annotated to ensure that the Prometheus scraper will automatically discover the API pods behind it. Prometheus will then collect their metrics from the discovered targets
5. Perform the API deployment. Start a new terminal session by selecting the Terminal top menu item and then click on the New Terminal option:
 
 
6. A new terminal session is presented in the bottom pane:
7. Deploy the API application. In the terminal execute the following command:
Copy code
 
1kubectl apply -f ./code/k8s2
 
8. Confirm that the API pods are in a running status. In the terminal execute the following command:
Copy code
 
1kubectl get pods
 
9. Confirm that the service has been created. In the terminal execute the following command:
Copy code
 
1kubectl get svc2
 
10. In order to generate traffic against the deployed API - spin up a single generator pod. In the terminal execute the following command:
Copy code
 
1kubectl run generator --env="API_URL=http://api-service:5000" --image=cloudacademydevops/api-generator --image-pull-policy IfNotPresent
 
11. Confirm that the generator pod is in a running status. In the terminal execute the following command:
Copy code
 
1kubectl get pods
 
 
Summary
In this Lab Step, you deployed a simple Python Flask web based API, which has been instrumented to automatically collect and provide metrics that will be collected by Prometheus. Additionally, you also deployed a single generator pod which will continually make HTTP requests against the API. In the next Lab Step you will install and configure Prometheus.
 
 
 
 
 
QA Platform
Accelerate progress up the cloud curve with Cloud Academy's digital training solutions. Build a culture of cloud with technology and guided learning experiences.
 
4 Install and Configure K8s Dashboard
0/1
Introduction
In this Lab Step, you'll install and configure the Kubernetes Dashboard, opens in a new tab and expose it over the Internet on port 30990, allowing you to then access it from your own workstation. The Kubernetes Dashboard is a web-based Kubernetes user interface. You can use Kubernetes Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and/or manage other cluster resources.
Instructions  
1. Create a new monitoring namespace within the cluster. In the terminal execute the following command:
Copy code
1kubectl create ns monitoring2
2. Using Helm, install the Kubernetes Dashboard using the publicly available Kubernetes Dashboard Helm Chart. Deploy the dashboard into the monitoring namespace within the lab provided cluster. In the terminal execute the following commands:
Copy code
1{2helm repo add k8s-dashboard https://kubernetes.github.io/dashboard3helm repo update4helm install k8s-dashboard --namespace monitoring k8s-dashboard/kubernetes-dashboard --set=protocolHttp=true --set=serviceAccount.create=true --set=serviceAccount.name=k8sdash-serviceaccount --version 3.0.25}
3. Establish permissions within the cluster to allow the Kubernetes Dashboard to read and write all cluster resources. In the terminal execute the following command:
Copy code
1kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=monitoring:k8sdash-serviceaccount
 
4. The Kubernetes Dashboard web interface now needs to be exposed to the Internet so that you can browse to it. To do so, create a new NodePort based Service, and expose the web admin interface on port 30990. In the terminal execute the following command:
Copy code
1{2kubectl expose deployment k8s-dashboard-kubernetes-dashboard --type=NodePort --name=k8s-dashboard --port=30990 --target-port=9090 -n monitoring3kubectl patch service k8s-dashboard -n monitoring -p '{"spec":{"ports":[{"nodePort": 30990, "port": 30990, "protocol": "TCP", "targetPort": 9090}]}}'4}
 
5. Get the public IP address of the Kubernetes cluster that Prometheus has been deployed into . In the terminal execute the following command:
Copy code
1export | grep K8S_CLUSTER_PUBLICIP
 
6. Copy the Public IP address from the previous command and then using your local browser, browse to the URL: http://PUBLIC_IP:30990.
 
 
Summary
In this Lab Step, you installed the Kubernetes Dashboard into the monitoring namespace within the Kubernetes cluster. You then setup and exposed the dashboard using a NodePort based Service. You then logged into the dashboard and confirmed that it was functional. In the next Lab Step you will install and configure Prometheus to start collecting metrics.
QA Platform
Accelerate progress up the cloud curve with Cloud Academy's digital training solutions. Build a culture of cloud with technology and guided learning experiences.
 
{

helm repo add k8s-dashboard https://kubernetes.github.io/dashboard

helm repo update

helm install k8s-dashboard --namespace monitoring k8s-dashboard/kubernetes-dashboard --set=protocolHttp=true --set=serviceAccount.create=true --set=serviceAccount.name=k8sdash-serviceaccount --version 3.0.2

}
Installation
General-purpose web UI for Kubernetes clusters
 
5 Install and Configure Prometheus
0/1
Introduction
In this Lab Step, you'll install and configure Prometheus, opens in a new tab into the lab provided Kubernetes cluster. Prometheus is an open-source systems monitoring and alerting service. You'll configure Prometheus to perform automatic service discovery of both the API pods launched in the previous Lab Step, and the cluster's nodes. Prometheus will then automatically begin to collect metrics for both the API pods and the cluster's nodes. HTTP request based metrics will be collected from the API pods, and Memory and CPU utilization metrics will be collected from the cluster's nodes. 
You will configure the Prometheus web admin interface to be exposed over the Internet on port 30900, allowing you to then access it from your own workstation.
 
Instructions  
1. Using Helm, install Prometheus using the publicly available Prometheus Helm Chart. Deploy Prometheus into the monitoring namespace within the lab provided cluster. In the terminal execute the following commands:
Copy code
1{2helm repo add prometheus-community https://prometheus-community.github.io/helm-charts3helm repo add stable https://charts.helm.sh/stable4helm repo update5helm install prometheus --namespace monitoring --values ./code/prometheus/values.yml prometheus-community/prometheus --version 13.0.06}


2. Confirm that Prometheus has been successfully rolled out within the cluster. In the terminal execute the following command:
Copy code
1kubectl get deployment -n monitoring -w2
 
Note: The previous command puts a watch on all deployments happening in the monitoring namespace. Exit the watch when all deployments have a READY status of 1/1 (CTRL+C to exit)
3. Confirm that the Prometheus Node Exporter DaemonSet resource has been created successfully. The Prometheus Node Exporter is used to collect Memory and CPU metrics off each node within the Kubernetes cluster. In the terminal execute the following command:
Copy code
1kubectl get daemonset -n monitoring
 
4. Patch the Prometheus Node Exporter DaemonSet to ensure that Prometheus can collect Memory and CPU node metrics. In the terminal execute the following command:
Copy code
1kubectl patch daemonset prometheus-node-exporter -n monitoring -p '{"spec":{"template":{"metadata":{"annotations":{"prometheus.io/scrape": "true"}}}}}'
 
5. The Prometheus web admin interface now needs to be exposed to the Internet so that you can browse to it. To do so, create a new NodePort based Service, and expose the web admin interface on port 30900. In the terminal execute the following command:
Copy code
1{2kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-main --port=30900 --target-port=9090 -n monitoring3kubectl patch service prometheus-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 30900, "port": 30900, "protocol": "TCP", "targetPort": 9090}]}}'4}5
 
6. Get the public IP address of the Kubernetes cluster that Prometheus has been deployed into. In the terminal execute the following command:
Copy code
1export | grep K8S_CLUSTER_PUBLICIP
 
7. Copy the Public IP address from the previous command and then using your local browser, browse to the URL: http://PUBLIC_IP:30900.
 
Note: The public IP address you use will be specific to your lab session and will therefore be different from the one shown above.
8. Within Prometheus, click the Status top menu item and then select Service Discovery:
 
9. Confirm that the following service discovery targets have been configured:
 
10. Within Prometheus, click the Status top menu item and then select Targets:
 
11. Confirm that the following Targets are available:
 
Summary
In this lab step, you installed Prometheus into the monitoring namespace within the Kubernetes cluster. You then set up and exposed the Prometheus web admin interface using a NodePort based Service. You then logged into the Prometheus web admin interface and confirmed the service discovery was working correctly. In the next lab step you will install and configure Grafana and import a prebuilt dashboard that pulls real-time data from Prometheus.
QA Platform
Accelerate progress up the cloud curve with Cloud Academy's digital training solutions. Build a culture of cloud with technology and guided learning experiences.
 
6 Install and Configure Grafana
0/1
Introduction
In this Lab Step, you'll install and configure Grafana, opens in a new tab into the monitoring namespace within the lab provided Kubernetes cluster. Grafana is an open source analytics and interactive visualization web application, providing charts, graphs, and alerts for monitoring and observability requirements. You'll configure Grafana to connect to Prometheus which you setup in the previous lab step as a data source. Once connected, you'll import and deploy a prebuilt Grafana dashboard.
You will configure the Grafana web admin interface to be exposed over the Internet on port 30300, allowing you to then access it from your own workstation.
 
Instructions  
1. Using Helm, install Grafana using the publicly available Grafana Helm Chart. You will deploy Grafana into the monitoring namespace within the lab provided cluster. In the terminal execute the following commands:
Copy code
1{2helm repo add grafana https://grafana.github.io/helm-charts3helm repo update4helm install grafana --namespace monitoring grafana/grafana --version 6.1.145}6
2. Confirm that the Grafana deployment has been rolled out successfully. In the terminal execute the following command:
Copy code
1kubectl get deployment grafana -n monitoring -w2
 
Note: The previous command puts a watch on the grafana deployment taking place in the monitoring namepace. Exit the watch when the deployment has a READY status of 1/1 (CTRL+C to exit)
3. The Grafana web admin interface now needs to be exposed to the Internet. To do so, create a new NodePort based Service, exposing the web admin interface on port 30300. In the terminal execute the following command:
Copy code
1{2kubectl expose deployment grafana --type=NodePort --name=grafana-main --port=30300 --target-port=3000 -n monitoring3kubectl patch service grafana-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 30300, "port": 30300, "protocol": "TCP", "targetPort": 3000}]}}'4}5
 
4. Extract the default admin password which will be required to login. In the terminal execute the following command:
Copy code
1kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
 
5. Get the public IP address of the Kubernetes cluster that Grafana has been deployed into. In the terminal execute the following command:
Copy code
1export | grep K8S_CLUSTER_PUBLICIP
 
6. Copy the Public IP address from the previous command and then using your local browser, browse to port http://PUBLIC_IP:30300.
 
7. To login into Grafana, use the following credentials:
Email or username: admin
Password: <default admin password extracted in step 4 above>
 
8. Having successfully authenticated, the Welcome to Grafana home page is displayed:
 
9. Within the Data Sources section, click on the Add your first data source option:
 
10. In the Add data source view, select the Prometheus option by clicking on it's Select button:
 
11. In the Data Sources / Prometheus configuration view update the HTTP URL to be the same URL that you previously used to browse to the Prometheus web admin interface. Leave all other default settings as is. In particular its important to leave the Name field set to Prometheus. Complete the Prometheus data source setup by clicking  on the Save & Test button at the bottom. 
 
12. Confirm that the Prometheus connectivity is valid and working - indicated by a green highlighted Data source is working message:
 
13. Return to the IDE. Within the Files tab on the left handside menu, open the project/code/grafana directory and click on the dashboard.json file to open it within the editor pane. In the editor pane, select all of the configuration and copy it to the clipboard.
 
14. Return to Grafana and this time select the Create icon (+) on the main left hand side menu, and then select the dashboard Import option like so:
 
15. In the Import view, paste in the copied Grafana dashboard json into the Import via panel json area and then click the Load button:
 
16. Under Import Options, accept all defaults without changing anything, and then click the Import button:
 
17. Grafana will now load the prebuilt dashboard and start rendering visualisations using live monitoring data streams taken from Prometheus. The dashboard view automatically refreshes every 5 seconds.
 
Summary
In this lab step, you installed Grafana into the monitoring namespace within the Kubernetes cluster. You then set up and exposed the Grafana web admin interface using a NodePort based Service. You then logged into the Grafana web admin interface and setup Prometheus as a data source. You then imported a prebuilt dashboard. Grafana then loaded the dashboard and starting pulling live monitoring data from the Prometheus data source. 
QA Platform
Accelerate progress up the cloud curve with Cloud Academy's digital training solutions. Build a culture of cloud with technology and guided learning experiences.
 