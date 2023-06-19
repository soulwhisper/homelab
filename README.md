# homelab

# Architecture
- OPNsense firewall;
- 2 servers, RHEL;
- Cloudstack, small-scale; KVM;
- FreeIPA VM x4, RHEL;
- AdguardHome VM x2, RHEL;
- EKS-A Cluster x2, Ubuntu 20.04 LTS; one for APP, the other for database;
- Step-ca for ACME on db-cluster;
- non-k8s containers on another VM, RHEL;
