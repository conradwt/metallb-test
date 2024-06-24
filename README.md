# MetalLB Test

## Software Requirements

- Minikube v1.33.1 or newer

- OrbStack v1.6.2 or newer

Note: This tutorial was updated on macOS 14.5.  The below steps doesn't work with Docker Desktop v4.31.1 because it doesn't expose Linux VM IP addresses to the host OS (i.e. macOS).

## Steps To Reproduce

1.  create Minikube cluster

    ```zsh
    minikube start -p metallb-test
    ```

2.  install MetalLB

    ```zsh
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
    ```

3.  clone github repository

    ```zsh
    git clone https://github.com/conradwt/metallb-test.git
    ```

4.  change directory

    ```zsh
    cd metallb-test
    ```

5.  locate the subnet

    ```zsh
    docker network inspect metallb-test | jq '.[0].IPAM.Config[0]["Subnet"]'
    ```

    Note: This grabs the Subnet value from the first config block and it should
    look something like the following:

    ```json
    {
      ...

      "Config": [
        {
          "Subnet": "194.1.2.0/24",
          "Gateway": "194.1.2.1"
        }
      ]
      ...
    }
    ```

    Then one can use an IP address range like the following:

    ```
    194.1.2.100-194.1.2.110
    ```

6.  update the `metallb-address-pool.yaml`

    ```yaml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: demo-pool
      namespace: metallb-system
    spec:
      addresses:
        - 194.1.2.100-194.1.2.110
    ```

7.  apply the address pool manifest

    ```zsh
    kubectl apply -f metallb-address-pool.yaml
    ```

8.  apply Layer 2 advertisement manifest

    ```zsh
    kubectl apply -f metallb-advertise.yaml
    ```

9.  apply deployment manifest

    ```zsh
    kubectl apply -f nginx-deployment.yaml
    ```

10. apply service manifest

    ```zsh
    kubectl apply -f nginx-service-loadbalancer.yaml
    ```

11. check that your service has an IP address

    ```zsh
    kubectl get svc nginx-service
    ```

    example output

    ```text
    NAME            TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
    nginx-service   LoadBalancer   10.106.207.172   194.1.2.100   80:32000/TCP   17h
    ```

12. expected output

    input:

    ```zsh
    curl 194.1.2.100
    ```

    output:

    ```text
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

13. actual output

    input:

    ```zsh
    curl 194.1.2.100
    ```

    output:

    ```text
    curl: (28) Failed to connect to 194.1.2.100 port 80 after 75001 ms: Couldn't connect to server
    ```
