## Containerize the Application

There are multiple options for containerizing a Spring Boot application.
As long as you are already building a Spring Boot jar file, you only need to call the plugin directly.
```shell
./gradlew bootBuildImage
```
You can run the container locally:
```shell
docker run -p 8080:8080 spring-kafka-k8s:0.0.1-SNAPSHOT
```
Then you can check that it works in another terminal:
```shell
curl localhost:8080/actuator/health
```

## Deploy the Application to Kubernetes

Load the image into Minikube: Load the Docker image into Minikube by running the following command:
```shell
minikube image load spring-kafka-k8s:0.0.1-SNAPSHOT
```
Services of type LoadBalancer can be exposed via the minikube tunnel command. It must be run in a separate terminal window to keep the LoadBalancer running.
Ctrl-C in the terminal can be used to terminate the process at which time the network routes will be cleaned up.
```shell
minikube tunnel
```
To avoid having to look at or edit YAML, for now, you can ask kubectl to generate it for you. The only thing that might vary here is the --image name.
If you deployed your container to your own repository, use its tag instead of this one:
```shell
kubectl create deployment spring-kafka-k8s --image=spring-kafka-k8s:0.0.1-SNAPSHOT --replicas=2 --dry-run=client -o=yaml > deployment.yml
echo --- >> deployment.yml
kubectl create service loadbalancer spring-kafka-k8s --tcp=8080:8080 --dry-run=client -o=yaml >> deployment.yml
```
The imagePullPolicy for a container and the tag of the image affect when the kubelet attempts to pull (download) the specified image.
You can take the YAML generated above and edit it if you like, or you can apply it as is:
```shell
kubectl apply -f deployment.yml
```