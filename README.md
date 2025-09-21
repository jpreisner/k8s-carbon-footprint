# k8s-carbon-footprint

Le fichier `k8s-carbon-footprint.xlsx` est un fichier Excel qui permet de calculer une estimation de l'empreinte carbone de pods Kubernetes.

Disclaimer : cette calculatrice ne permet que de faire une estimation en se basant sur des hypothèses et en simplifiant grandement le calcul. Si vous le pouvez, privilégiez des outils d'évaluation comme [Kepler](https://github.com/sustainable-computing-io/kepler) ou [Scaphandre](https://github.com/hubblo-org/scaphandre) pour avoir des résultats plus précis.

Voici une explication du calcul réalisé par `k8s-carbon-footprint.xlsx`.

- [1. Hypothèses](#1-hypothèses)
  - [1.1. Energie consommée par 1 CPU](#11-energie-consommée-par-1-cpu)
  - [1.2. Energie consommée par 1 Go de RAM](#12-energie-consommée-par-1-go-de-ram)
  - [1.3. Conversion kWh en kgCO2eq](#13-conversion-kwh-en-kgco2eq)
  - [1.4. Power Usage Effectivenes (PUE)](#14-power-usage-effectivenes-pue)
  - [1.5. Heures de disponibilités et d'arrêts des pods](#15-heures-de-disponibilités-et-darrêts-des-pods)
- [2. Formule](#2-formule)
  - [2.1. Calcul de l'énergie de la consommation CPU d'un pod par an](#21-calcul-de-lénergie-de-la-consommation-cpu-dun-pod-par-an)
  - [2.2. Calcul de l'énergie de la consommation RAM d'un pod par an](#22-calcul-de-lénergie-de-la-consommation-ram-dun-pod-par-an)
  - [2.3. Calcul de l'empreinte carbone d'un pod par an](#23-calcul-de-lempreinte-carbone-dun-pod-par-an)

## 1. Hypothèses

Les hypothèses détaillées ci-dessous sont les valeurs par défaut présente dans la feuille de calcul.

Elles sont toutes modifiables depuis l'onglet `Paramètres`.

### 1.1. Energie consommée par 1 CPU

D'après [cloud-carbon-footprint](https://github.com/cloud-carbon-footprint/cloud-carbon-footprint/blob/trunk/microsite/docs/HowItWorks/Methodology.md#compute), l'énergie consommée par 1 CPU peut s'estimer via la formule suivante :

`Average Watts = Min Watts + Avg vCPU Utilization * (Max Watts - Min Watts)`

En prenant les hypothèses de AWS :
- `Min Watts = 0,74`
- `Max Watts = 3,5`

Et en prenant l'hypothèse que : `Avg vCPU Utilization = 50%`,

Alors l'énergie consommée par 1 CPU est estimée à :

`E(cpu) = 0,00212 kWh`

### 1.2. Energie consommée par 1 Go de RAM
D'après [cloud-carbon-footprint](https://github.com/cloud-carbon-footprint/cloud-carbon-footprint/blob/trunk/microsite/docs/HowItWorks/Methodology.md#memory), l'énergie consommée par 1 Go de RAM peut être estimée de la manière suivante :

`E(ram) = 0,000392 kWh`

### 1.3. Conversion kWh en kgCO2eq

D'après [statista](https://fr.statista.com/infographie/33063/intensite-carbone-production-electricite-par-pays-en-europe/), en tenant compte du mix énergétique français, alors :

`G = 0,056 kgCO2eq./kWh`

### 1.4. Power Usage Effectivenes (PUE)

Afin de prendre en compte l'efficacité énergétique des serveurs, voici l'hypothèse prise pour représenter le [PUE](https://fr.wikipedia.org/wiki/Indicateur_d%27efficacit%C3%A9_%C3%A9nerg%C3%A9tique) : 

`PUE = 1,5`

### 1.5. Heures de disponibilités et d'arrêts des pods

L'hypothèse prise sur les heures de disponibilités : 08h00 - 20h00 en semaine. Cela signifie que les pods sont arrêtés de 20h à 8h en semaine ainsi que durant les weekends.

Cela se traduit par le calcul des heures sur 1 an : 
- `Nombre d'heures en période active = 12 heures * 5 jours * 52 semaines = 3120 heures`
- `Nombre d'heures en période inactive = Nobmre d'heures dans une année - Nombre d'heures en période active = 8760 - 3120 = 5640 heures`

## 2. Formule

### 2.1. Calcul de l'énergie de la consommation CPU d'un pod par an

Prérequis : 

- Mesurer `CPUmoy (en période active)` : la consommation moyenne en CPU d'un pod pendant la période où le pod sera UP
- Mesurer `CPUmoy (en période inactive)` : la consommation moyenne en CPU d'un pod pendant la période où le pod sera UP

Calcul (en kWh) : 

`E(cpu d'un pod par an) = [CPUmoy (en période active) * Nombre d'heures (en période active) + CPUmoy (en période inactive) * Nombre d'heures (en période inactive)] * E(cpu) * PUE`

### 2.2. Calcul de l'énergie de la consommation RAM d'un pod par an

Prérequis : 

- Mesurer `RAMmoy (en période active)` : la consommation moyenne en RAM (Go) d'un pod pendant la période où le pod sera UP
- Mesurer `RAMmoy (en période inactive)` : la consommation moyenne en RAM (Go) d'un pod pendant la période où le pod sera UP

Calcul (en kWh) : 

`E(ram d'un pod par an) = [RAMmoy (en période active) * Nombre d'heures (en période active) + RAMmoy (en période inactive) * Nombre d'heures (en période inactive)] * E(ram) * PUE`

### 2.3. Calcul de l'empreinte carbone d'un pod par an

Calcul (en kgCO2eq) :

`GES (d'un pod par an) = [E(cpu d'un pod par an) + E(ram d'un pod par an)] * G`

## Contributeurs
- Richard PLANCHON
- Julien PREISNER (@jpreisner)