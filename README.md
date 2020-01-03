# airflow-on-kubernetes
running Airflow on kubernetes cluster with Celery executor

Clone the https://github.com/puckel/docker-airflow.git

cd docker-airflow
edit the Docker file

add kubernetes in the following line like this

&& pip install apache-airflow[crypto,celery,postgres,hive,jdbc,mysql,kubernetes,ssh${AIRFLOW_DEPS:+,}${AIRFLOW_DEPS}]==${AIRFLOW_VERSION}

docker build -t <imageName> .

to decrease all this , can use directly this docker image
docker pull varaginikarthik/airflow_aws_image:latest

Install Helm
	-> 	curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
    
	->	chmod 700 get_helm.sh
		
	->	./get_helm.sh
		
	->  	helm init

kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 

helm init --service-account tiller --upgrade


After the HELM instalation

Clone this helm repo https://github.com/helm/charts.git

cd charts/stable/airflow

edit the values.yaml file

change the image name to the new one that we created

   image:
    repository: varaginikarthik/airflow_aws_image
    tag: latest
    
enable persistance storage to true, because Airflow uses postgres db .

  persistence:
    -  enabled: true
save the changes.

now install the aiflow with heml command

helm install --namespace "airflow" --name "airflow" stable/airflow -f values.yaml

it will take some to for web pod to come up.

create ingress for web pod to access Airflow UI outside.

Atlast we can see the AirflowUI in web with ingress IP.


for GIT SYNC in Airflow 

Git Sync gets the dags from our git repo, this is popularly used 

open the values.yaml file, do the following changes for respective fields

git:

     ##
     
     ## url to clone the git repository
     
    url: https://github.com/karthikVaragini/ganiDAGs.git ------ Repo where our Dags reside
    
     ##
     
     ## branch name, tag or sha1 to reset to
     
     ref: master
     
     privateKeyName: ""
     
     gitSync:
     
       ## Turns on the side car container
       
      enabled: true
      
       ## The amount of time in seconds to git pull dags
       
      refreshTime: 2
      
   initContainer:
   
     ## Fetch the source code when the pods starts
     
     enabled: true
     

again install the Airflow with the following command 

helm install --namespace "airflow" --name "airflow" stable/airflow -f values.yaml

to delete the Airflow use the following helm command

helm delete --purge airflow



