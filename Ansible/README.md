Creazione di un Cluster Kubernetes su Proxmox VE 7 tramite playbook Ansible.

Il processo è composto da otto playbook, divisi in:

Playbook per il provisioning delle Virtual Machine da un template Debian 11
	-	provisioning-m1.yml
	-	provisioning-w1.yml
	-	provisioning-w2.yml

Descrizione:
Il playbook provisioning-m1.yml si connette all'hypervisor e da un template Debian 11, crea una VM (destinata al nodo Master) chiamata Debian.
Gli altri due playbook (w1-w2) creeranno altre due vm generiche ma destinate ai nodi worker.

NOTA: Potevo fare un playbook unico, ma ho preferito dividere.

Playbook per le configurazioni e setup dei software necessari.
	-	setup-m1.yml
	-	setup-w1.yml
	-	setup-w2.yml

Descrizione:
I 3 playbook eseguono sostanzialmente lo stesso setup, ovvero attivano i moduli del kernel linux "overlay" e "br_netfilter", eseguono il setup dei software necessari a kubernetes, ma il playbook setup-m1 effettua l'apertura delle porte del firewall che sono diverse rispetto ai worker. 

Playbook per inizializzare il Cluster e per la join dei nodi worker
	-	kubeadm-init.yml (installa anche calico)
	-	kubeadm-join.yml



