# **Configuração do ambiente Kubernetes Cluster com Ansible no CentOS 7**

### **Descrição**

Instalação e configuração dos requisitos para configurar um cluster kubernetes
com ansible.

#### **Requisitos:**
 
 - OpenSSH
 - Ansible 2.6.+
 - Docker 18.+
 - Desabilitar a SWAP de todos os servidores ```swapoff -a``` e comente a entrada da swap no ```/etc/fstab```
 - Conexão livre entre os servidores.

### **./inventories/staging/hosts**

Configure o arquivo ```./inventories/staging/hosts```

```bash
vim ./inventories/staging/hosts

[cluster-k8s]
<server-master>     # Hostname ou IP do servidor master.
<server-workder-1>  # Hostname ou IP do servidor worker-1.
<server-workder-2>  # Hostname ou IP do servidor workder-2.
```

### **/etc/hosts**

Configure o arquivo ```/etc/hosts``` com IP e hostname dos servidores que compoem
o cluster. Essa configuração deve ser feita em todos os servidores.

```bash
echo "<IP>  <MASTER>" >> "/etc/hosts"
echo "<IP>  <WORKER-1>" >> "/etc/hosts"
echo "<IP>  <WORKER-2>" >> "/etc/hosts" 
```

Dentro da pasta do projeto  ```deploy-cluster-k8s``` execute ```anisble-playbook```.

**OBS:** Com o parametro ```-C``` o processo é executado em modo teste.

**Teste:**
```bash
ansible-playbook -i inventories/stagins/hosts -l cluster-k8s -k site.yml -C
```

**Execução:**
```bash
ansible-playbook -i inventories/stagins/hosts -l cluster-k8s -k site.yml
```

**Saída:**
![](/docs/images/img1.jpg)


## **Inicializando o Cluster**

No servidor master inicie o cluster kubernetes.

 ```bash
 kubeadm init --apiserver-advertise-address <IP DO MASTER> --ignore-preflight-errors=all
 ```
 Finalizado a instalação do cluster o kubeadm irá gerar um comando que você precisa executar nos servidores workers., copie a linha e cole em todos os servidores workers do seu cluster.
 
 **Exemplo**
 ```bash
 kubeadm join --token <TOKEN GERADO PELO KUBEADM> <IP DO MASTER>:6443 --discovery-token-ca-cert-hash sha256:<HASH GERADO PELO KUBEADM> --ignore-preflight-errors=all
 ```

Se todos os servidores do cluster estiverem configurados corretamente, no servidor master
verifique se todos os servidores workers são listados.

```bash
kubectl get nodes -o wide
```

![](/docs/images/img2.jpg)

O status dos servidores estão como **```NotReady```**, isso porque é preciso instalar um add-on/plugin de rede para que a comunicação entre os servidores fique disponível.

Existe uma lista de [add-on/plugins](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy) disponíveis.

 ## **Plugin de Rede**

Para esse exemplo vamos usar o [weave-net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/).

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Aguarde até que todos os pods de rede estejam rodando.

![](/docs/images/img3.jpg)

Verifique se todos os servidores do cluster estão disponíveis.

![](/docs/images/img4.jpg)


# **Conclusão**

Esse projeto tem como objetivo fazer a instalação e configuração do cluster kubernetes com 3 servidores,
1 master e 2 workers.

![](/docs/images/img5.jpg)