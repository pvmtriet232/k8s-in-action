<!-- Chapter 05 -->
<!-- dry run a single pod output to a file  -->
k run kiada --image=pvmtriet233/kiada:0.1 --dry-run=client -o yaml > example.yaml
<!-- apply manifest -->
k apply -f path/to/file.yaml

<!-- Connect to a pod from a one-off client pod -->
k run --image=curlimages/curl -it --restart=Never --rm <pod-name> curl <ip:port>

