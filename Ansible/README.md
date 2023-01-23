Creazione di un Cluster Kubernetes su Proxmox VE 7 tramite playbook Ansible.

Il processo è composto da otto playbook, divisi in 3 macro steps:

STEP 1:	Provisioning delle 3 VM 
STEP 2: Configurazione delle 3 VM (la differenza dei tra master e worker si differenzia per le porte aperte sul FW)
STEP 3:	Inizializzazione del Cluster k8s sul nodo Master tramite kubeadm e join dei nodi Worker (questo STEP  avrei dovuto effettuarlo con Terraform ma non sono riuscito a trovare il modo. Argomento da studiare :) )

Di seguito l'elenco dei playbook:

Playbook per il provisioning delle Virtual Machine da un template Debian 11
	-	provisioning-vm.yml

Playbook per le configurazioni e setup dei software necessari.
	-	setup-m1.yml
	-	setup-w1-w2.yml

Playbook per inizializzare il Cluster e per la join dei nodi worker
	-	kubeadm-init-join.yml (installa anche calico)

Descrizione della parte di provisioning:
Il playbook provisioning-vm.yml si connette all'hypervisor, da un template Debian 11 crea una ad una le 3 VM Debian.

Descrizione della parte della configurazione:
I 2 playbook eseguono sostanzialmente lo stesso setup, ovvero attivano i moduli del kernel linux "overlay" e "br_netfilter", eseguono il setup dei software necessari a kubernetes, ma il playbook setup-m1 effettua l'apertura delle porte del firewall che sono diverse rispetto ai worker. 

Descrizione della parte di init e join del cluster:
il playbook eseguirà prima un kubeadm reset, poi se il reset avrà esito success proseguirà con l'inizializzazione del nodo master.
Crea la cartella nascosta ".kube/", copia al suo interno in un file denominato config, il contenuto del file "admin.conf", viene poi creata una cartella che conterrà i file "calico.yaml" e "components.yaml" per la configurazione dei pod, per la network e per il pod che restituisce le metriche del cluster e dei vari pod.
Subito dopo verrà eseguita la join dei nodi worker.

NOTA: prima dell'installazione del file components.yaml ho eseguito una sed che mi inseriesce nel file yaml con la giusta tabulazione l'opzione --kubelet-insecure-tls, per fare in modo che funzioni tutto senza l'opzione --kubelet-insecure-tls bisogna:

editare il configMap di Kubelet "kubectl -n kube-system edit cm kubelet-config" e aggiungere questa opzione "serverTLSBootstrap: true" dopo la parte di logging (di seguito un estratto del CM):

memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s

serverTLSBootstrap: true # opzione aggiunta

shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

La stessa opzione andrà aggiunta nel file config.yaml che si trova nel path "/var/lib/kubelet/config.yaml" su tutti i nodi del cluster e partendo dal nondo master riavviare il servizio kubelet "systemctl restart kubelet.service".

A seguioto del riavvio del servizio kubelet lanciando il comando "kubectl get csr" ci verrà mostrata una tabella dei file csr appena creati da kubelet, 

root@K8S-M1:~# kubectl get csr
NAME        AGE   SIGNERNAME                      REQUESTOR            REQUESTEDDURATION   CONDITION
csr-5mndl   71m   kubernetes.io/kubelet-serving   system:node:k8s-w1   <none>              Pending
csr-snwkf   67m   kubernetes.io/kubelet-serving   system:node:k8s-m1   <none>              Pending
csr-tfc9j   70m   kubernetes.io/kubelet-serving   system:node:k8s-w2   <none>              Pending

a questo punto bisogna approvarli con il comando "kubectl certificate approve <name-csr>" dopo averli approvati tutti lo stato finale sarà il come quello mostrato nella tabella che segue:

root@K8S-M1:~# kubectl get csr
NAME        AGE   SIGNERNAME                      REQUESTOR            REQUESTEDDURATION   CONDITION
csr-5mndl   71m   kubernetes.io/kubelet-serving   system:node:k8s-w1   <none>              Approved,Issued
csr-snwkf   67m   kubernetes.io/kubelet-serving   system:node:k8s-m1   <none>              Approved,Issued
csr-tfc9j   70m   kubernetes.io/kubelet-serving   system:node:k8s-w2   <none>              Approved,Issued

A questo punto potete installare il pod per le metriche del cluster, tramite Helm o direttamente applucando il file yaml senza dover aggiungere l'opzione --kubelet-insecure-tls.







