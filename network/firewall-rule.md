gcloud compute firewall-rules create \
mynetwork-ingress-allow-ssh-from-cs \
--network mynetwork --action ALLOW --direction INGRESS \
--rules tcp:22 --source-ranges $ip --target-tags=lab-ssh


gcloud compute instances add-tags mynet-vm-2 \
    --zone europe-west1-d \
    --tags lab-ssh
gcloud compute instances add-tags mynet-vm-1 \
    --zone us-east1-c \
    --tags lab-ssh


Stateful firewalls
In VPC networks, firewall rules are stateful. So for each initiated connection tracked by allow rules in one direction, the return traffic is automatically allowed, regardless of any rules.


gcloud compute firewall-rules create \
mynetwork-ingress-allow-icmp-internal --network \
mynetwork --action ALLOW --direction INGRESS --rules icmp \
--source-ranges 10.128.0.0/9


gcloud compute firewall-rules create \
mynetwork-ingress-deny-icmp-all --network \
mynetwork --action DENY --direction INGRESS --rules icmp \
--priority 500

gcloud compute firewall-rules update \
mynetwork-ingress-deny-icmp-all \
--priority 2000

gcloud compute firewall-rules list \
--filter="network:mynetwork"

gcloud compute firewall-rules create \
mynetwork-egress-deny-icmp-all --network \
mynetwork --action DENY --direction EGRESS --rules icmp \
--priority 10000