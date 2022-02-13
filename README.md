# ingress-nginx-auth-upstream-keepalive-test
Verification sample for testing if ingress-nginx's external auth's upstream keepalive block works or not

This verification suite has a:
- dummy auth app
- dummy app
- ansible-playbook

The `dummy auth` app gives back HTTP 200 only if called with the appropriate api key and api secret key.
The `dummy app` dummy verifies it: the authorization header must be able to be decriptable with the provided symmetric key `SECRET_KEY` residing as an environment variable.
The `ansible-playbook` makes it easy to get a testing environment up and running, it:
- clones and builds the PR for the [auth-keepalive-upstream PR](https://github.com/kubernetes/ingress-nginx/pull/8219) by [Leki75](http://github.com/leki75/)
- builds the two dummy images
- creates a kind cluster
- deploys the ingress-nginx helmchart with the custom controller image


# Run
You need to have `ansible` and `helm` installed.
To give it a run, 

```
ansible-playbook install.yaml
```

To start up a client to test keepalive under load:

```
docker run --rm -it --network host  client bash
```
A load-tester util [hey](https://github.com/rakyll/hey) is installed in the client image. 
The shell history contains a single item which will give the system a nice load.