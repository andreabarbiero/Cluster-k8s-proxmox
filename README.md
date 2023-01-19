###### Kiratech Challenge - Kubernetes ######

Obbiettivo:
Eseguire il provisioning di un cluster Kubernetes composto da un manager e due workers e deployare una applicazione composta da almeno tre servizi e che presenti una interfaccia grafica accessibile via browser.

Hypervisor:
La scelta dell'Hypervisor si divideva in 3 software che utilizzo day by day.
-	VMware Workstation
-	VirtualBox
-	Proxmox

Ho scelto Proxmox come hypervisor semplicemente perchè mi sembrava una simulazione più simile agli ambienti enterprice.


Attività:
Per una miglore organizzazione del lavoro, ho diviso la challenge in 6 step,
che andrò ad elencare di seguito:

STEP(1)
Creazione su Proxmox di una VM basata su Debian11 per creare un template da utilizzare con Ansible.

STEP(2)
Provisioning delle VM tramite Ansible.

STEP(3)
Configurazione delle VM tramite Ansible.

STEP(4)
Predisporre un provider Terraform per inizializzare i ruoli di Master e di Worker
Creare un namespace tramite Terraform
Avviare un test per la security 

STEP(5)
Configurare un'app tramite HELM, composta da almeno 3 servizi e che sia aggiornabile con meno disservizio possibile.
portare il codice su Github per avere una history

STEP(6)
Configurazione di una pipeline con un software a scelta per la CI.





