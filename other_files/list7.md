## Define a Command and Arguments for a Container

```
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```
## Use environment variables to define arguments

```
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    env:
    - name: MESSAGE
      value: "hello world"
    command: ["/bin/echo"]
    args: ["$(MESSAGE)"]
  restartPolicy: OnFailure
```

```
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```