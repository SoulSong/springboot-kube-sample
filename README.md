# Background
This is a simple example of a blue/green deployment. It depends on springboot2, builds by maven, deploys by kubernetes.

# How To Use

## Change Version
Change the project version in `pom.xml` as follow:
```pom
    <groupId>com.shf.example</groupId>
    <artifactId>springboot-kube-sample</artifactId>
    <version>Blue</version>
```    
It will be read in `VersionController`:
```java
@Slf4j
@RestController
public class VersionController {

    @Autowired(required = false)
    private BuildProperties buildProperties;

    @GetMapping("/version")
    public String version() {
        if (null == buildProperties) {
            return "UnKnow";
        }
        return buildProperties.getVersion();
    }

}
```

## Build image 
Dockerfile is in ./docker folder, just input `mvn clean install` command. Then we could get two images:
* local-dtr.com/springboot-kube-sample:Blue                   
* local-dtr.com/springboot-kube-sample:Green

## Deploy into kubernetes cluster
In ./kubernetes folder, you will find all yaml-files for kubernetes.

### Create namespace
* kubectl create namespace blue-green

### Create deployment
* kubectl apply -f deployment-blue.yaml
* kubectl apply -f deployment-green.yaml

### Create service(`blue`)
* kubectl apply -f service-blue.yaml

### Create ingress
* kubectl apply -f ingress.yaml -n blue-green

### Modify hosts file
Add hosts configuration into your **/etc/hosts file, the domains which defined in ingress.yaml
```text
127.0.0.1	shf.sample.com
```

### Check Version
> INPUT
``` text
curl -i http://shf.sample.com/sample-service/version
```
> OUTPUT
```  text
Blue
```

### Change service version(`green`)
* kubectl apply -f service-green.yaml

### ReCheck Version
> INPUT
``` text
curl -i http://shf.sample.com/sample-service/version
```
> OUTPUT
``` text
Green
```
