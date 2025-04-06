# Helm-Minecraft-mods

# Minecraft Helm Chart

## Description

Ce projet Helm permet le déploiement automatisé d'un serveur Minecraft modifié dans un cluster Kubernetes. Le chart prend également en charge la configuration de mods et de modpacks spécifiques, ainsi que la persistance des données du serveur.

---

## Installation

### Prérequis

- Cluster Kubernetes opérationnel.
- Outil Helm installé (minimum v3).
- Stockage persistant configuré (par exemple, `openebs-hostpath`).

### Commandes d'installation

1. Clonez ce dépôt :

   ```bash
   git clone https://github.com/votre-utilisateur/Helm-Minecraft-mods.git
   cd Helm-Minecraft-mods
   ```

2. Installez le chart Helm avec les valeurs par défault:

   ```bash
    helm upgrade --install server-minecraft . -n minecraft
   ```

3. (Optionnel) Personnalisez le fichier `values.yaml` avant de lancer la commande pour adapter les paramètres à vos besoins.

4. Mettez à jour une installation existante :

   ```bash
    helm upgrade --install server-minecraft . -f values.yaml -n minecraft
   ```

---

## Configuration de `values.yaml`

Le fichier `values.yaml` gère la configuration de base pour le serveur Minecraft et les mods associés. Voici un aperçu des paramètres principaux :

### 1. **Image de base**

L'image Docker utilisée pour exécuter le serveur Minecraft.

```yaml
image:
  repository: itzg/minecraft-server
  tag: latest
```

- **`repository`** : Repository Docker de l'image Minecraft.
- **`tag`** : Tag de l'image (peut être remplacé par une version spécifique).

---

### 2. **Configuration du serveur Minecraft**

Ajoutez les paramètres Minecraft tels que le mode de jeu, les ressources mémoire et le type de serveur utilisé (par exemple, CurseForge pour les modpacks).

```yaml
minecraft:
  mode: survival                   # Mode de jeu (ex : survival, creative)
  type: CURSEFORGE                 # Type de serveur Minecraft
  initMemory: 8G                   # Mémoire initiale (RAM) allouée
  maxMemory: 12G                   # Mémoire maximale (RAM) autorisée
  modpackPath: /modpacks/Craftoria-Server-1.17.3.zip  # Chemin vers le modpack
```

- **`modpackPath`** : Charge un fichier modpack spécifique pour déployer automatiquement les mods associés.

---

### 3. **Configuration des mods/modpacks**

La section `modsconfig` permet de gérer un modpack à l'aide d'une URL directe.

```yaml
modsconfig:
  url: "https://edge.forgecdn.net/files/6307/377/Craftoria-Server-1.17.3.zip"
  name: Craftoria-Server-1.17.3.zip
```

- **`url`** : Lien direct pour télécharger le modpack demandé (fichier `.zip` ou `.jar`).
- **`name`** : Nom du fichier modpack qui sera téléchargé.

---

### 4. **Configuration des volumes persistants**

Les données du serveur (monde, configurations, mods, etc.) sont sauvegardées à l'aide des volumes Kubernetes configurés ici :

```yaml
persistence:
  data:
    claimName: minecraft-data          # Nom du PVC pour les données Minecraft
    storageClass: openebs-hostpath     # Classe de stockage
    accessMode: ReadWriteOnce          # Mode d'accès au volume
    size: 250Gi                        # Taille du volume

  modpacks:
    claimName: modpacks                # Nom du PVC pour les fichiers de modpacks
    storageClass: openebs-hostpath
    accessMode: ReadWriteOnce
    size: 25Gi                         # Taille du volume pour les modpacks
```

---

### 5. **Configuration des services**

Exposez le serveur Minecraft via un service Kubernetes, en définissant le port et le type de service.

```yaml
service:
  type: LoadBalancer            # Type de service (NodePort ou LoadBalancer)
  port: 25565                   # Port principal pour Minecraft
```

---

### Exemple complet de `values.yaml`

```yaml
image:
  repository: itzg/minecraft-server
  tag: latest

minecraft:
  mode: survival
  type: CURSEFORGE
  initMemory: 8G
  maxMemory: 12G
  modpackPath: /modpacks/Craftoria-Server-1.17.3.zip

modsconfig:
  url: "https://edge.forgecdn.net/files/6307/377/Craftoria-Server-1.17.3.zip"
  name: Craftoria-Server-1.17.3.zip

resources:
  requests:
    cpu: "4"
    memory: "12Gi"
  limits:
    cpu: "8"
    memory: "16Gi"

persistence:
  data:
    claimName: minecraft-data
    storageClass: openebs-hostpath
    accessMode: ReadWriteOnce
    size: 250Gi

  modpacks:
    claimName: modpacks
    storageClass: openebs-hostpath
    accessMode: ReadWriteOnce
    size: 25Gi

service:
  type: LoadBalancer
  port: 25565
```

---

## Remarque sur le déploiement des mods

Lorsque vous déployez un modpack, Helm téléchargera automatiquement le fichier depuis l’URL spécifiée dans `modsconfig.url` et l'extraira dans le répertoire défini (par exemple, `/modpacks`).

### Exemple : Ajouter un autre modpack

Pour ajouter un nouveau modpack, mettez à jour la section `modsconfig` dans `values.yaml` avec l'URL et le nom du fichier:

```yaml
modsconfig:
  url: "https://example.com/path/to/another-modpack.zip"
  name: "another-modpack.zip"
```

Puis appliquez les modifications :

```bash
 helm upgrade --install server-minecraft . -f values.yaml -n minecraft
```

---

## Notes

Lors du déploiement:
- Assurez-vous que les URLs des modpacks ou des mods sont valides.
- Vérifiez les capacités disponibles pour votre stockage persistant (Taille minimale recommandée : `250Gi` pour les données + `25Gi` pour les modpacks).

---

## License

Ce projet est sous licence **MIT**. Veuillez consulter [LICENSE](./LICENSE) pour plus de détails.

---
