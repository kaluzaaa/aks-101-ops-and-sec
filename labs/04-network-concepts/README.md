# 4. Explain network concepts for applications in AKS

ToDo
https://docs.microsoft.com/en-us/azure/virtual-network/container-networking-overview
https://docs.microsoft.com/en-us/azure/aks/configure-advanced-networking
https://docs.microsoft.com/en-us/azure/aks/internal-lb
https://docs.microsoft.com/en-us/azure/aks/static-ip


Łukasz, w nastepnym Labie zakładam, że tutaj postawią gotowy klaster z Azure CNI (bez --network-policy). Ja w nastepnym kroku doinstaluję na nim DaemonSet NPM v1.18 i wdrożę prosty nginx ingress + kilka usług by pokazać działanie polityk.
Potem ty w #6 podrasujesz ingress - SSL czy też ILB.
