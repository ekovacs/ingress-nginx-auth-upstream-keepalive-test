# ingress-nginx-auth-upstream-keepalive-test
Under heavy load, nginx subrequests may create too many lingering client sockets, which can cause the subrequest to fail with 500
```
[crit] 51#51: *2067723 connect() to <auth-service-ip>:<port> failed (99: Address not available) while connecting to upstream, client: <client-ip>, server: <your-server>, request: "GET /path HTTP/2.0", subrequest: "/_external-auth-Lw-Prefix", upstream: "http://<auth-service-ip>:<port>/auth", host: "<host>"
```
To prevent that, auth subrequests should use keepalive connections.
[Leki75](https://github.com/leki75) provided just that in [PR](https://github.com/kubernetes/ingress-nginx/pull/8219)

This verification sample is for testing how ingress-nginx's external auth's upstream keepalive block works, to check how connections behave when this feature is `on` or `off`.



This verification suite has a:
- dummy auth app
- dummy app
- ansible-playbook

The `dummy auth` app gives back HTTP 200 only if called with the appropriate api key and api secret key.
The `dummy app` dummy verifies it: the authorization header must be able to be decryptable with the provided symmetric key `SECRET_KEY` residing as an environment variable.
The `ansible-playbook` makes it easy to get a testing environment up and running, it:
- clones and builds the PR for the [auth-keepalive-upstream PR](https://github.com/kubernetes/ingress-nginx/pull/8219) by [Leki75](http://github.com/leki75/)
- builds the two dummy images
- creates a kind cluster
- deploys the ingress-nginx helmchart with the custom controller image


# Run
You need to have `make`, `ansible` and `helm` installed.
To give it a run, 

```
ansible-playbook install.yaml
```

To start up a client to test keepalive under load:

```
docker run --rm -it --network host client bash
```
A load-tester util [hey](https://github.com/rakyll/hey) is installed in the client image. 
The shell history contains a single item which will give the system a nice load.

While the loadtester runs, one can check the number of `ESTABLISHED` connections on each side of the line:
ingress-controller side:
```
export ingress_namespace=ingress-nginx-keepalive-test; for pod in $(k get pod -n $ingress_namespace -l app.kubernetes.io/component=controller,app.kubernetes.io/instance=$ingress_namespace -o name ); do
      echo "$pod:"; k exec -n $ingress_namespace $pod -- netstat -nat | awk '{tw[$5" "$6]++}END{for(dst in tw){ if(tw[dst]>1) print tw[dst]" "dst}}' | sort -n; echo "\n"
done
```

auth app side:
```
export namespace=default; export authApp=auth; export authPort=8080; for pod in $(k get pod -n $namespace -l app=$authApp -o name ); do
      echo "$pod:"; k exec -n $namespace $pod -- netstat -natl | awk '$4 ~ /[0-9]+:8080/{split($5, foreign, ":");tw[foreign[1]" "$6]++}END{for(dst in tw){ print tw[dst]" "dst}}' | sort -n; echo "\n"
done
```
Note: when the traffic is higher than the number of keepalive connections, then nginx will still build up an ephemeral TCP connection to the destination [as per nginx documentation](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive). You can play around with this by changing the number of concurrency in the `hey` command, eg.: `-c 5` or `-c 100`.