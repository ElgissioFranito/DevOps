# Aide-mémoire — Structures de contrôle et fonctions Bash

> **Bloc 3 · Leçon 2** — Fiche de référence pour écrire scripts conditionnels et boucles.

## 📌 Conditions

| Expression | Effet |
|-----------|------|
| `if [ "$x" = "a" ]; then ...; fi` | Compare des **chaînes** (`=` egal, `!=` diff.)
| `if [ "$n" -eq 5 ]; then ...; fi` | Compare des **entiers** (`-eq`, `-ne`, `-gt`, `-lt`, `-ge`, `-le`)
| `if [[ "$s" =~ ^web ]]; then ...; fi` | Test **regex** (moderne, `[[ ]]` uniquement)
| `if [ -f "$f" ]` / `-d` / `-z` / `-n` | Existe(en tant que fichier/dossier), chaîne vide / non vide
| `if cmd; then ...; fi` | Test sur le **code de sortie** d'une commande
| `elif` / `else` | Chaîne de conditions

## 📌 Boucles

```bash
# for sur une liste explicite
for env in production staging dev; do
    echo "$env"
done

# for sur des fichiers
for conf in /etc/app/*.conf; do
    echo "$conf"
done

# while : lire un fichier
while IFS= read -r ligne; do
    echo "$ligne"
done < /etc/app/hosts.txt

# while : polling
try=0
while [[ "$try" -lt 10 ]] && ! curl -sf http://localhost:8080/health; do
    ((try++)); sleep 2
done
```

## 📌 `case`

```bash
case "$var" in
    production)  echo "prod" ;;
    staging)     echo "staging" ;;
    *)           echo "inconnu"; exit 1 ;;
esac
```

## 📌 Fonctions

```bash
nom() {
    local p="$1"          # nomme + local
    # corps...
    return 0             # toujours explicite
}

# Appel
nom "valeur"
# Resultat de l'appel : $?
```

## 📌 Opérateurs de test fichiers,valeurs

| Test | Vrai si |
|------|---------|
| `-f` | fichier régulier existe
| `-d` | dossier existe
| `-e` | le chemin existe (fichier ou dossier)
| `-z` | chaîne vide
| `-n` | chaîne non vide
| `-r` | fichier lisible
| `-x` | fichier exécutable

## 📌 Effets de bord à connaître

- `[ ... ]` est une **commande externe** : exige des espaces de chaque côté des crochets.
- Dans `[ ]`, `<` / `>` sont des **redirections**, pas des comparaisons → utiliser `-lt` / `-gt`.
- `$(cmd)` exécute une commande et insère sa sortie (substitution de commande).
- `$((expr))` évalue une **arithmétique** (ex: `$((try+1))`).
