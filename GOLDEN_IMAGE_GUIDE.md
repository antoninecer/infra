# Průvodce vytvořením "Golden Image" (šablony) v Proxmoxu

V playbooku `create_vm.yml` odkazujeme na proměnnou `vm_template`, která má výchozí hodnotu `ubuntu-2204-cloud-template`. Toto je váš **"golden image"** – v terminologii Proxmoxu se tomu říká **šablona (template)**.

## Co je to Golden Image / Šablona?

Je to předpřipravený, čistý a standardizovaný obraz virtuálního stroje, který slouží jako základ pro rychlé vytváření nových VM. Místo toho, abyste každý nový server instalovali od nuly z ISO obrazu, jednoduše naklonujete tuto šablonu.

### Klíčové výhody:
1.  **Rychlost:** Vytvoření nového VM z šablony trvá desítky sekund, na rozdíl od minut či desítek minut při instalaci z ISO.
2.  **Konzistence:** Všechny nové VM mají stejný základní stav, stejné balíčky, stejnou konfiguraci a stejné bezpečnostní nastavení.
3.  **Automatizace:** Šablony jsou klíčové pro automatizaci. Náš `create_vm.yml` playbook spoléhá právě na existenci šablony, ze které klonuje nové servery pro K8s cluster.

## Jak vytvořit Golden Image?

Nejlepší způsob, jak vytvořit dobrou šablonu, je použít nástroje jako `cloud-init`, které umožňují automatickou konfiguraci VM při prvním startu (nastavení sítě, uživatelů, SSH klíčů atd.).

### Postup (přehled):

1.  **Stáhnout Cloud Image:** Nejjednodušší je stáhnout si již hotový "cloud-init" obraz operačního systému, například Ubuntu Cloud Image.
    ```bash
    wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
    ```

2.  **Vytvořit novou VM v Proxmoxu:** Vytvořte v Proxmoxu novou VM s minimální konfigurací.

3.  **Importovat stažený obraz:** Importujte stažený `.img` soubor na storage této nové VM.
    ```bash
    qm importdisk <vmid> jammy-server-cloudimg-amd64.img <storage_name>
    ```

4.  **Připojit disk a nastavit Cloud-Init:** V hardwaru VM připojte nově importovaný disk jako hlavní a přidejte "Cloud-Init drive".

5.  **Nainstalovat `qemu-guest-agent`:** Spusťte VM, připojte se do konzole a nainstalujte `qemu-guest-agent` pro lepší komunikaci s Proxmoxem.
    ```bash
    sudo apt-get update
    sudo apt-get install qemu-guest-agent
    ```

6.  **Vyčistit a připravit VM:** Můžete provést další úpravy (instalace základních balíčků, nastavení atd.). Poté je dobré VM "vyčistit".

7.  **Převést VM na šablonu:** Vypněte VM a v Proxmox UI ji převeďte na šablonu (pravým tlačítkem na VM -> "Convert to template").

### Automatizace pomocí Ansible

Celý tento proces můžeme také zautomatizovat pomocí Ansible. Mohli bychom vytvořit nový playbook, například `create_template.yml`, který by tyto kroky provedl za vás.

Pokud budete chtít, mohu vám takový playbook připravit. Prozatím se ale můžete zaměřit na ruční vytvoření první šablony podle kroků výše, abyste se seznámil/a s procesem.
