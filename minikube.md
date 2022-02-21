
# Install Minikube + DNS 

## Install Hyper-V
Open powershell as administrator (run as) \
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
==Reboot your system==

## Install Minikube
Open powershell again as administrator
```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

## Add minikube to your path 
#### After adding Minikube to your path ensure you restart powershell!
```powershell 
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}
```


## Install kubectl 
```powershell
Invoke-WebRequest -OutFile 'c:\minikube\kubectl' -Uri 'https://dl.k8s.io/release/v1.23.0/bin/windows/amd64/kubectl.exe' -UseBasicParsing
```
If the download process is to slow then copy the weblink and download through the browser and save kubectl.exe in c:\minikube

## Start Minikube 
```powershell 
minikube start
```
## Enable Minikube Ingress/DNS addons
```powershell
minikube addons enable ingress
minikube addons enable ingress-dns
```

## Add Minikube DNS to your system 
#### It is important to know that you will need the DNS entry again each time you reboot your system
```powershell
Add-DnsClientNrptRule -Namespace ".test" -NameServers "$(minikube ip)"
```

## Test ingress: 
```powershell 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/minikube/master/deploy/addons/ingress-dns/example/example.yaml
```

## Check if ingress is created
```powershell
kubectl get ingress
```

## Test connection
```powershell 
ping hello-john.test
```

## Stop minikube enviromnent
```powershell
minikube stop
``` 


