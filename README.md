# 7.2.3 Infrastructure as code
## Installation du labo
![labo1](labo-ansible-1.png "Labo 1")
![labo1](labo-ansible-2.png "Labo 1")
# Atelier 01
## Challenge 1
```bash
vagrant up ubuntu
vagrant ssh ubuntu
sudo apt update
apt-cache search --names-only ansible
sudo apt install -y ansible
ansible --version
```
Version Ansible :
```bash
ansible 2.10.8
  config file = None
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Aug 15 2025, 14:32:43) [GCC 11.4.0]
```

```bash
exit
vagrant destroy -f ubuntu
```
## Challenge 2
```
sudo apt update
sudo apt install -y ansible
ansible --version
```

Version Ansible : 
```bash
ansible [core 2.17.14]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Aug 15 2025, 14:32:43) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
```

## Challenge 3
```bash
vagrant up rocky
vagrant ssh rocky
sudo dnf update -y
sudo dnf search python
sudo dnf install -y python3.12
sudo dnf install -y python3.12-pip
python3 -m venv ~/.venv/ansible
source ~/.venv/ansible/bin/activate
pip install --upgrade pip
pip install ansible
ansible --version
```

Version Ansible: 
```bash
ansible [core 2.15.13]
  config file = None
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/vagrant/.venv/ansible/lib64/python3.9/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/vagrant/.venv/ansible/bin/ansible
  python version = 3.9.21 (main, Aug 19 2025, 00:00:00) [GCC 11.5.0 20240719 (Red Hat 11.5.0-5)] (/home/vagrant/.venv/ansible/bin/python3)
  jinja version = 3.1.6
  libyaml = True
```

# Atelier 03
```bash
sudo tee /etc/hosts > /dev/null <<'EOF'
# /etc/hosts
127.0.0.1      localhost.localdomain  localhost
192.168.56.10  control.sandbox.lan    control
192.168.56.20  target01.sandbox.lan   target01
192.168.56.30  target02.sandbox.lan   target02
192.168.56.40  target03.sandbox.lan   target03
EOF
```

```bash
ssh-keyscan -t rsa target01 target02 target03 >> .ssh/known_hosts
ssh-keygen
ssh-copy-id vagrant@target01
ssh-copy-id vagrant@target02
ssh-copy-id vagrant@target03
ansible all -i target01,target02,target03 -u vagrant -m ping
```

# Atelier 06
Rendre les *Target Hosts* joignables par leur nom d'hôte simple :
```bash
sudo tee /etc/hosts > /dev/null <<'EOF'
# /etc/hosts
127.0.0.1      localhost.localdomain  localhost
192.168.56.10  control.sandbox.lan    control
192.168.56.20  target01.sandbox.lan   target01
192.168.56.30  target02.sandbox.lan   target02
192.168.56.40  target03.sandbox.lan   target03
EOF
```
Configuration de l'authentification par clé SSH avec les trois *Target Hosts* :
```bash
ssh-keyscan -t rsa target01 target02 target03 >> .ssh/known_hosts
ssh-keygen
ssh-copy-id vagrant@target01
ssh-copy-id vagrant@target02
ssh-copy-id vagrant@target03
```
Installation de Ansible :
```bash
sudo apt update
sudo apt install -y ansible
```
Envoie d'un premier `ping` Ansible sans configuration :
```bash
ansible all -i target01,target02,target03 -u vagrant -m ping
```
Création du fichier `ansible.cfg` et vérification de sa prise en compte par Ansible : 
```bash
mkdir monprojet
cd monprojet/
touch ansible.cfg
ansible --version | head -n 2
```
Mise en place de l'environnement :
```bash
sudo apt install -y direnv
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
source ~/.bashrc
echo 'export ANSIBLE_CONFIG=$(expand_path ansible.cfg)' > .envrc
direnv allow
```
Création de l'inventaire :
```bash
cat << 'EOF' >> ansible.cfg
[defaults]
inventory = ./inventory
EOF
```
```bash
cat << 'EOF' >> inventory 
[testlab]
target01
target02
target03
EOF
```
Activation de la journalisation :
```bash
mkdir ~/journal
echo 'log_path = ~/journal/ansible.log' >> ansible.cfg

# Test de la journalisation
ansible all -i target01,target02,target03 -m ping
cat ~/journal/ansible.log
```
Définition de l'utilisateur `vagrant` pour la connexion aux cibles de notre groupe `[testlab]` :
```bash
cat << 'EOF' >> inventory


[testlab:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=vagrant
EOF
```
Envoie d'un `ping` vers le groupe de machines `[all]` : 
```bash
ansible all -m ping
```
Définition de l'élèvation des droits pour l'utilisateur `vagrant` sur les *Target Hosts* : 
```bash
echo 'ansible_become=yes' >> inventory
```
Affichage de la première ligne du fichier `/etc/shadow` sur tous les *Target Hosts* : 
```bash
ansible all -a "head -n 1 /etc/shadow"
```
# Atelier 07
Installation des paquets `tree`, `git`, et `nmap` : 
```bash
cd ansible/projets/ema/
ansible all -m package -a "name=tree"
ansible all -m package -a "name=git"
ansible all -m package -a "name=nmap"
```
Désinstallation des paquets `tree`, `git`, et `nmap` : 
```bash
ansible all -m package -a "name=tree state=absent"
ansible all -m package -a "name=git state=absent"
ansible all -m package -a "name=nmap state=absent"
```
Copie du fichier `/etc/fstab` du *Control Host* vers les *Target Hosts* sous forme d'un fichier `/tmp/test3.txt` : 
```bash
ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
```
Suppression du fichier `/tmp/test3.txt` sur les *Target Hosts* en utilisant le module `file` : 
```bash
ansible all -m file -a "path=/tmp/test3.txt state=absent"
```
Affichage de l'espace utilisé par la partition principale sur tous les *Target Hosts* : 
```bash
ansible all -m command -a "df -h /"
```
Nous pouvons remarquer que les differents systemes n'utilisent pas le meme espace systeme : 
```bash
debian | CHANGED | rc=0 >>
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/debian--12--vg-root   62G  1.7G   57G   3% /
rocky | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        61G  2.0G   59G   4% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        64G  2.4G   58G   5% /
```
