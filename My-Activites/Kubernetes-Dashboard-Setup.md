- Install headlamp using helm by following [doc](https://headlamp.dev/docs/latest/installation/in-cluster/#using-helm)
- Expose
```bash
kubectl port-forward -n kube-system svc/my-headlamp 8081:80

localhost:8081
```
- Create user and copy the token to access by following [doc](https://headlamp.dev/docs/latest/installation/#create-a-service-account-token)
