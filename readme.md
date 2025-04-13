
# 📦 Stockage distant NFS - Configuration complète

## I) Stockage distant NFS

---

### 1) 🖥️ Serveur NFS

#### 🛠️ Installation
```bash
dnf install nfs-utils
```

#### 🚀 Démarrage et activation du service
```bash
systemctl start nfs-server
systemctl enable nfs-server
```

#### 🔥 Configuration du firewall
```bash
firewall-cmd --add-service={rpc-bind,nfs,mountd} --permanent
firewall-cmd --reload
```

#### 📂 Création de partage NFS
```bash
mkdir /partage
vim /etc/exports
# Ajouter :
# /partage *(rw,no_root_squash)
systemctl restart nfs-server
```

---

### 2) 💻 Côté Client

#### 🛠️ Installation
```bash
dnf install nfs-utils
```

#### 🔧 Montage

- **Temporaire**
```bash
mkdir /rep1
mount @IPserveur:/partage /rep1
```

- **Permanent**
```bash
vim /etc/fstab
# Ajouter :
# @IPserveur:/partage /rep1 nfs _netdev 0 0

mount -a
```

---

### 3) 🔁 Client - Autofs

#### 🛠️ Installation
```bash
dnf install autofs
```

#### 🚀 Démarrage et activation
```bash
systemctl start autofs
systemctl enable autofs
```

#### 📌 Mappage Direct
```bash
vim /etc/auto.ticR
# /rep1 @IPserveur:/partage

vim /etc/auto.master
# /- /etc/auto.ticR

systemctl restart autofs
```

#### 📌 Mappage Indirect
```bash
vim /etc/auto.master
# /rep1 /etc/auto.ticR

vim /etc/auto.ticR
# rep2 -rw 192.168.232.129:/partage
```

---

### 4) 👤 Montage des répertoires personnels via Autofs

#### 🖥️ Côté Serveur

**Utilisateur :** `rayen`, **UID :** `3564`, **Home :** `/server/rayen`
```bash
useradd -u 3564 -d /server/rayen rayen -b /rayen

vim /etc/exports
# /server/rayen *(rw,no_root_squash)

systemctl restart nfs-server
```

#### 💻 Côté Client
```bash
useradd -u 3564 -d /client/rayen -M rayen

vim /etc/auto.master
# /client /etc/auto.rayen

vim /etc/auto.rayen
# rayen -rw @ip_reseau:/server/rayen

systemctl restart autofs
```

---

📝 **Remarque :** Remplacez `@IPserveur` ou `@ip_reseau` par l'adresse IP réelle du serveur NFS.
