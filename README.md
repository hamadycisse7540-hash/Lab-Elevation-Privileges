# Lab — Post-exploitation et élévation de privilèges sous Windows 7

Rapport de laboratoire réalisé dans le cadre du **Mentorat Cybersécurité CYB6 (FORCE N)**.
Scénario complet de **post-exploitation** sur une cible Windows 7 : création d'un fichier
malveillant avec **msfvenom**, obtention d'un accès distant *reverse TCP*, dissimulation de la
porte dérobée dans un logiciel légitime (**PuTTY**) avec persistance, puis **élévation de
privilèges** par contournement de l'UAC jusqu'à `NT AUTHORITY\SYSTEM`. Le tout mené sur un
laboratoire **isolé** (réseau VirtualBox interne `10.6.6.0/24`), suivi de l'analyse des
contre-mesures défensives.

> ⚠️ Toutes les manipulations ont été effectuées sur des machines virtuelles volontairement
> vulnérables, dans un environnement fermé et sans accès Internet, à des fins strictement
> pédagogiques. N'effectuez jamais ces techniques sur un système sans autorisation écrite.

## Environnement

| Machine | Rôle | IP | Système |
|---|---|---|---|
| Kali Linux | Attaquant (Metasploit, msfvenom) | `10.6.6.1` | Kali |
| Windows 7 | Cible / victime | `10.6.6.2` | Windows 7 SP1 **x86** (CISSE-VM) |

## Contenu du lab

1. **Tâche 1 — Fichier malveillant autonome (Reverse TCP)** : génération d'un `file.exe`
   (`windows/shell/reverse_tcp`, port 9999) avec msfvenom, diffusion via `http.server`,
   écoute avec `multi/handler`, exécution sur la cible et obtention d'un **shell distant** ;
   constat des privilèges limités de l'utilisateur standard (`whoami /priv`).
2. **Tâche 2 — Cheval de Troie associé à PuTTY** : injection d'une charge **Meterpreter**
   dans le binaire légitime PuTTY (`msfvenom -x`, encodage `shikata_ga_nai`), prise de
   contrôle interactive, puis mise en place de la **persistance**
   (`exploit/windows/persistence/registry_userinit`).
3. **Tâche 3 — Élévation de privilèges (UAC Bypass)** : contournement de l'UAC via
   `exploit/windows/local/bypassuac_eventvwr`, passage à `NT AUTHORITY\SYSTEM`
   (`getsystem`), migration vers un processus SYSTEM (`lsass.exe`) et extraction des
   empreintes NTLM (`hashdump`).
4. **Réflexion et contre-mesures** : difficultés rencontrées, application en milieu
   professionnel et hiérarchie des mesures de défense (moindre privilège, mise à jour,
   contrôle applicatif, EDR, protection des identifiants).
5. **Annexe** : récapitulatif des commandes utilisées, de leurs rôles et de leurs options.

## Structure du dépôt

```
.
├── Rapport_Lab_Elevation_Privileges.tex   # Source LaTeX du rapport
├── Rapport_Lab_Elevation_Privileges.pdf   # Rapport compilé (18 pages, 23 captures)
├── Rapport_Lab_Elevation_Privileges.pdf.asc   # Signature GPG détachée du PDF
├── HAMADY_CISSE_public_key.asc            # Clé publique GPG de l'auteur
├── signature.tex                          # Bloc signature / authenticité (LaTeX)
├── images/                                # Captures d'écran légendées du lab
│   └── logo_forcen.png                    # Logo FORCE N (en-tête + page de titre)
├── LICENSE
└── README.md
```

## Compilation

```bash
pdflatex Rapport_Lab_Elevation_Privileges.tex
pdflatex Rapport_Lab_Elevation_Privileges.tex   # 2e passe pour le sommaire et l'annexe
```

## Vérifier l'authenticité

Le PDF du rapport est signé numériquement (GPG). Pour confirmer qu'il vient bien de l'auteur
et qu'il n'a pas été modifié :

```bash
# 1. Importer la clé publique de l'auteur
gpg --import HAMADY_CISSE_public_key.asc

# 2. Vérifier la signature détachée du PDF
gpg --verify Rapport_Lab_Elevation_Privileges.pdf.asc Rapport_Lab_Elevation_Privileges.pdf
```

La vérification doit afficher `Good signature from "HAMADY CISSE ..."`. L'empreinte de la clé
est :

```
A6F4 0DFD EE3E B7AE 80B3  6E77 361C 9E02 B790 303E
```

Les commits de ce dépôt sont eux aussi signés avec cette clé (badge « Verified » sur GitHub).

## Auteur

**HAMADY CISSE** — Mentorat Cybersécurité CYB6, Groupe 3.
Encadrant : M. Ibrahima Samba DIOP.

## Licence

© 2026 HAMADY CISSE. Ce travail est publié sous licence
**[Creative Commons Attribution - Pas d'Utilisation Commerciale 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/)**.

Vous pouvez partager et adapter ce travail **à des fins non commerciales**, à condition
de **créditer l'auteur**. Voir le fichier [`LICENSE`](LICENSE) pour les termes complets.
