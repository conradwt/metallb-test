# MetalLB Test

## Steps To Reproduce

1.  create Minikube cluster

    ```zsh
    minikube start -p metallb-test
    ```

2.  clone github repository

    ```zsh
    git clone https://github.com/conradwt/metallb-test.git
    ```

3.  change directory

    ```zsh
    cd metallb-test
    ```

4.  locate the address pool

    ```zsh
    docker network inspect minikube
    ```

    Note: grab the Subnet value from the first config block and it should
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

5.  update the `metallb-address-pool.yaml`

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

6.  apply the address pool manifest

    ```zsh
    kubectl apply -f metallb-address-pool.yaml
    ```

7.  apply Layer 2 advertisement manifest

    ```zsh
    kubectl apply -f metallb-advertise.yaml
    ```

8.  apply deployment manifest

    ```zsh
    kubectl apply -f nginx-deployment.yaml
    ```

9.  apply service manifest

    ```zsh
    kubectl apply -f nginx-service-loadbalancer.yaml
    ```

10. check that your service has an IP address

    ```zsh
    kubectl get svc nginx-service
    ```

    example output

    ```text
    NAME            TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
    nginx-service   LoadBalancer   10.106.207.172   194.1.2.100   80:32000/TCP   17h
    ```

11. expected output

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

12. actual output

    input:

    ```zsh
    curl 194.1.2.100
    ```

    output:

    ```text
    curl: (28) Failed to connect to 194.1.2.100 port 80 after 75001 ms: Couldn't connect to server
    ```
