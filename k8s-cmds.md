<!-- Chapter 05 -->
<!-- dry run a single pod output to a file  -->
k run kiada --image=pvmtriet233/kiada:0.1 --dry-run=client -o yaml > example.yaml
<!-- apply manifest -->
k apply -f path/to/file.yaml

<!-- Connect to a pod from a one-off client pod -->
k run --image=curlimages/curl -it --restart=Never --rm <pod-name> curl <ip:port>

<!-- connect to a pod via k port-forward -->
k port-forward <pod-name> port...

<!-- VIEW APPLICATION LOGS -->
<!-- Streaming logs using -f flag -->
k logs <pod-name> -f

<!-- log with parameters -->
k logs <pod-name> --timestamps=true --since=3m --since-time=<time-stamp> --tail=10

<!-- log from previous container -->
k logs --previous (-p) ...
<!-- display logs of pods with multiple containers -->
k logs <pod-name> -c <container-name> 
k logs <pod-name> --all-containers

<!-- COPY FILE TO AND FROM CONTAINERS ( HELPFUL WHEN DEVELOPING) -->
k cp <pod-name>:path/to/file   path/to/destination 
k cp path/to/file  <pod-name>:path/to/destination

<!-- EXECUTING CMD IN RUNNING CONTAINERS -->

k exec <pod-name> -- <cmd...>
k exec -it <pod-name> -- bash
k exec -it <pod-name> -c <container-name> -- bash
<!-- ATTACH TO A RUNNING CONTAINERS -->

k attach <pod-name> (-i) 

<!-- Display a pod phase  -->
k get pod <pod-name> -o yaml | grep phase

<!-- display only some field of an object manifest by: -->
k get po <pod-name> -o json | jq .status.conditions




