**[English version](README.md)**

# Ce que ce rôle fait et ne fait pas

Ce rôle est conçu pour automatiser le déploiement d'instances [PeerTube](https://joinpeertube.org) avec [ansible](https://www.ansible.com).

Cependant, **il n'installe pas tous les services nécessaires au fonctionnement de PeerTube**. En effet, il existe d'excellents rôles ansible pour déployer ces services, et la plupart des utilisateurs d'ansible en utilisent déjà certainement.

Voici ce qu'il faut installer par ailleurs :

- NodeJS
- Yarn
- Nginx
- PostgreSQL
- Redis
- Un certificat TLS valide

Ce rôle se contente d'installer PeerTube et de configurer les services nécessaires (y-compris Nginx et PostgreSQL), en considérant que toutes les dépendances sont déjà correctement installées.

Pour l'instant, il fonctionne uniquement avec **Debian 11 (bullseye)**. Une version compatible avec Debian 12 (bookworm) est en préparation.

À noter également que ce rôle est conçu pour **déployer** PeerTube et non pour le **mettre à jour**. Si vous avez déjà installé PeerTube et voulez le mettre à jour, il est conseillé de suivre [les instructions officielles de mise à jour](https://docs.joinpeertube.org/install/any-os#upgrade) avant de relancer une nouvelle version de ce rôle.


# Variables

## Version de PeerTube

La version de PeerTube installée par ce rôle est définie dans `vars/main.yml`. C'est actuellement la version **6.0.2**. Cette variable n'est pas censée être modifiée, car il s'agit de la seule version supportée.


## Variables à définir obligatoirement

Ce rôle nécessite qu'un certain nombre de variables propres à l'hôte (à l'instance de PeerTube) soient définies, sachant que pour elles il n'y a pas de valeur par défaut :

- `peertube_user_password` : hash du mot de passe de l'utilisateur système peertube (cf. https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module)
- `peertube_postgresql_user_passwd` : mot de passe de l'utilisateur peertube dans postgresql (haché ou non, cf. https://docs.ansible.com/ansible/latest/collections/community/postgresql/postgresql_user_module.html#parameter-password)
- `peertube_webserver_hostname` : hostname du serveur (utilisé aussi dans la configuration de nginx et le certificat TLS)
- `peertube_secrets` : un "secret" propre à l'instance (généré avec `openssl rand -hex 32`)

Pour des raisons de sécurité, il peut être utile de chiffrer certaines de ces variables avec `ansible-vault`.


## Variables optionnelles

Les autres variables sont optionnelles, dans la mesure où elles ont des valeurs par défaut.

Une seule variable n'est pas liée directement à la configuration de PeerTube :

- `peertube_import_key` : si ne veut pas réimporter à chaque fois la clé OpenPGP nécessaire pour vérifier la signature du paquet zip de PeerTube, on peut mettre cette variable à `false`. Il faudra la remettre à `true` si cette clé est modifiée (en cas d'expiration, de révocation, de changement de signataire, etc.).

Toutes les autres variables déterminent les paramètres du fichier `production.yaml`. Leur nom reflète la structure YAML de ce fichier, préfixée par `peertube_`. Les valeurs possibles sont (pour la plupart) commentées dans [templates/production.yaml.j2](templates/production.yaml.j2), et les valeurs par défaut se trouvent dans [defaults/main.yml](defaults/main.yml).

**Ce rôle ne prend pas en charge tous les paramètres de PeerTube.** Cependant, les variables disponibles devraient déjà couvrir la plupart des besoins de déploiement. Si vous avez besoin d'exposer des variables supplémentaires, n'hésitez pas à constribuer sous forme de "merge request" ;-)

Attention, deux ensembles de variables doivent être définies en concordance avec la façon dont PostgreSQL et Redis sont installés :

### Paramètres de PostgreSQL

- `peertube_postgresql_hostname`
- `peertube_postgresql_port`
- `peertube_postgresql_ssl`
- `peertube_postgresql_suffix`
- `peertube_postgresql_username`
- `peertube_postgresql_pool_max`

### Paramètres de Redis

- `peertube_redis_hostname`
- `peertube_redis_port`


# Exemple de playbook

```
- name: My PeerTube instance
  hosts:
    mynode
  become: true

  vars:
    peertube_user_password: MY_LONG_PASSWORD_HASH
    peertube_postgresql_user_passwd: MY_POSTGRESQL_PASSWORD
    peertube_webserver_hostname: MY_DOMAIN.TLD
    peertube_secrets: MY_LONG_SECRET_LINE

  roles:
    # - nodejs, yarn, nginx, etc.
    - peertube-core
```


# Licence

Ce rôle est distribué selon les termes de la [licence BSD-3-Clause](LICENSE).
