# Référence rapide — Leçon 3 : Machines virtuelles (EC2)

> Bloc 6 · Leçon 3 — Aide-mémoire.

## Concepts
- **EC2** = le service de VM d'AWS (« Elastic Compute Cloud », la VM louée de la Leçon 1).
- **Instance** = une VM en cours d'exécution.
- **Type d'instance** = gabarit CPU/RAM (ex. `t3.micro` = 1 CPU / 1 Go ; `t3.large` = 2 CPU / 8 Go).
- **AMI** = modèle de système préinstallé (ex. Ubuntu).
- **Key pair** = clé privée (toi) + clé publique (serveur) pour le SSH (Bloc 2).
- **Security group** = pare-feu virtuel de la machine (Leçon 2).
- **running / stopped / terminated** = coûte / coûte peu (disque) / détruite (rien, mais perte).

## Étapes de vie d'une instance
```
AMI + type + clé SSH + security group + subnet
   ↓
run-instances (création)
   ↓
ssh -i ma-cle.pem ubuntu@<IP>   (admin)
   ↓
snapshot (backup)
   ↓
stop / start / terminate (selon besoin)
```

## Commandes AWS (après clés — Leçon 6)
```bash
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*" --query "sort_by(Images, &CreationDate)[-1].{ID:ImageId,Nom:Name}" --output table
aws ec2 run-instances --image-id ami-xxx --instance-type t3.micro --key-name ma-cle-ssh --subnet-id subnet-xxx --associate-public-ip-address
aws ec2 describe-instances --query "Reservations[].Instances[].{ID:InstanceId,Etat:State.Name,IP:PublicIpAddress}"
chmod 600 ma-cle-ssh.pem
ssh -i ma-cle-ssh.pem ubuntu@<IP>
```

## Bonnes pratiques
- Commencer petit (`t3.micro`), dimensionner après observation (sizing).
- Autoscaling pour les pics ; snapshots réguliers.
- Clé privée `chmod 600` et **jamais** dans Git.
- Ne pas exposer SSH à tout Internet (restreindre aux IP de confiance / bastion).
- Arrêter/supprimer ce qui ne sert plus (première économie — FinOps, Leçon 8).