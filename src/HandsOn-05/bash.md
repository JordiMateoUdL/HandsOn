# Programació de Scripts (BASH)

## Sintaxi bàsica

Un script **Bash** és un conjunt de comandes emmagatzemat en un fitxer de text amb l'extensió *.sh*. Aquest script es pot executar com un programa per mitjà de la línia de comandes. Els paràmetres es passen a l'script mitjançant les variables especials $1, $2, etc., on $1 és el primer paràmetre, $2 és el segon, i així successivament.

```bash
if [ $# -ne 2 ]; then
    echo "Usage: $0 <dir> <ext>"
    exit 1
fi
```

En aquest fragment, **$#** representa el nombre total de paràmetres passats. L'estructura if verifica si el nombre de paràmetres és diferent de 2. En aquest cas, mostra un missatge d'ús i acaba amb un codi d'error. L'estructura **$0** representa el nom del programa.

## Activitat 01: Calculadora

En aquesta activitat, implementarem una calculadora que ens permeti sumar, restar, multiplicar i dividir dos nombres enters. Per això, crearem un script anomenat **calc.sh**. Aquest script rep dos paràmetres i una opció. Els dos paràmetres són els dos nombres enters i l'opció és l'operació que volem realitzar. Aquest script mostrarà el resultat de l'operació per pantalla.

```bash
#!/bin/bash
# Nom: calc.sh

if [ $# -ne 3 ]; then
    echo "Usage: $0 <num1> <num2> <op>"
    exit 1
fi

num1=$1
num2=$2
op=$3

case $op in
    "sum")
        echo "$num1 + $num2 = $((num1+num2))"
        ;;
    "sub")
        echo "$num1 - $num2 = $((num1-num2))"
        ;;
    "mul")
        echo "$num1 * $num2 = $((num1*num2))"
        ;;
    "div")
        echo "$num1 / $num2 = $((num1/num2))"
        ;;
    *)
        echo "Error: $op is not a valid operation"
        exit 1
        ;;
esac
```

Aquest script utilitza la sentència `case`. Aquesta sentència ens permet avaluar una variable i executar una acció en funció del valor d'aquesta variable. En aquest cas, la variable és **$op** i l'acció és mostrar el resultat de l'operació.

Aquest script es pot executar de la següent manera:

```bash
bash calc.sh 2 3 sum
# Resultat: 2 + 3 = 5
```

Ara millorarem aquest script per assegurar-nos que els paràmetres que rep són enters. Per això, utilitzarem la comanda `expr`. Aquesta comanda ens permet avaluar una expressió. La comanda `expr "$1" + 1` intenta avaluar l'expressió aritmètica **$1 + 1**. Si *$1* no és un enter, això generaria un error, que redirigim a **/dev/null**. Aquest procés es repeteix per al segon paràmetre.

```bash
#!/bin/bash

if ! expr $1 + 1 &> /dev/null || ! expr $2 + 1 &> /dev/null; then
    echo "Error: $1 or $2 is not an integer"
    exit 1
fi
```

Pensa un altra forma de comprovar si els paràmetres són enters.

<details>
<summary>Resposta</summary>

Podem utilitzar una expressió regular per comprovar si els paràmetres són enters. Per exemple, per comprovar si el paràmetre $1 és un enter:

```bash
if [[ ! "$1" =~ ^[0-9]+$ ]] || [[ ! "$2" =~ ^[0-9]+$ ]]; then
    echo "Error: $1 or $2 is not an integer"
    exit 1
fi
```

Per avaluar expressions regular en bash es requereix **[[ var =~regex ]]**. L'expressió regular **^[0-9]+$** indica que la cadena ha de contenir només dígits (0-9) i res més. Si alguna cosa diferent d'un enter es troba, llavors mostrarà un missatge d'error i sortirà amb un codi d'error.

Notes sobre expressions regulars:

- El caràcter *^*, això fixa l'expressió a l'inici d'una línia.
- La expressió *[0-9]+*, això coincideix amb almenys un caràcter del conjunt 0-9 -> 1 o més caràcters del rang 0-9.
- La expressió *.+*, això coincideix com a mínim amb un caràcter de qualsevol tipus.
- La expressió *[0-9]*, coincideix amb un sol dígit.
- El caràcter *?*, assegura 0 o 1 de l'expressió anterior.
- El caràcter *$*, això fixa l'expressió al final de la línia.

</details>

Anem a millorar aquest script perquè ens permeti realitzar operacions amb nombres decimals. Per això, utilitzarem la comanda `bc`. Aquesta comanda ens permet avaluar expressions aritmètiques. Per exemple, per avaluar l'expressió aritmètica **2.5 + 3.5**:

```bash
echo "2.5 + 3.5" | bc
# Resultat: 6.0
```

Modifiquem el script anterior perquè ens permeti realitzar operacions amb nombres decimals. Per això, utilitzarem la comanda `bc`. Aquesta comanda ens permet avaluar expressions aritmètiques. Per exemple, per avaluar l'expressió aritmètica **2.5 + 3.5**:

```bash
#!/bin/bash
# Nom: calc.sh

if [ $# -ne 3 ]; then
    echo "Usage: $0 <num1> <num2> <op>"
    exit 1
fi

num1=$1
num2=$2
op=$3

if [[ ! "$1" =~ ^[0-9]+\.?[0-9]*$ ]] || [[ ! "$2" =~ ^[0-9]+\.?[0-9]*$ ]]; then
    echo "Error: $1 or $2 is not a number"
    exit 1
fi

case $op in
    "sum")
        echo "$num1 + $num2 = $(echo "$num1 + $num2" | bc)"
        ;;
    "sub")
        echo "$num1 - $num2 = $(echo "$num1 - $num2" | bc)"
        ;;
    "mul")
        echo "$num1 * $num2 = $(echo "$num1 * $num2" | bc)"
        ;;
    "div")
        echo "$num1 / $num2 = $(echo "$num1 / $num2" | bc)"
        ;;
    *)
        echo "Error: $op is not a valid operation"
        exit 1
        ;;
esac
```

Analitzeu el codi següent i digueu quin és el resultat de l'execució d'aquest script. Raoneu la resposta. Indicant per que el resultat és diferent del que s'espera.

```bash
#!/bin/bash
# Nom: ch01.sh

cat > ch01.data << 'EOT'
102
071
210
153
EOT

clear
cat ch01.data
declare -i SUM=0

while read X; do
    SUM+=$X
done < ch01.data

echo "Suma: $SUM != 536"
rm -f ch01.data
```

<details>
<summary>Resposta</summary>

El resultat de l'execució d'aquest script és el següent:

```bash
102
071
210
153
Suma: 522 != 536
```

El resultat és diferent del que s'espera perquè el nombre 071 no és un nombre decimal. Aquest nombre és un nombre octal. Per tant, el resultat de la suma és 522 i no 536. En bash, els nombres que comencen per 0 són nombres octals. Per exemple, el nombre 071 en octal és el nombre 57 en decimal. Per tant, el resultat de la suma és 522 i no 536.

Per assegurar-nos que els nombres que llegim són nombres decimals, podem utilitzar la comanda `bc`. Aquesta comanda ens permet avaluar expressions aritmètiques. Per exemple, per avaluar l'expressió aritmètica **071**:

```bash
echo "071" | bc
# Resultat: 71
```

Per tant, podem millorar el script anterior de la següent manera:

```bash
#!/bin/bash
# Nom: ch01.sh

cat > ch01.data << 'EOT'
102
071
210
153
EOT

clear
cat ch01.data
declare -i SUM=0

while read X; do
    SUM+=$(echo "$X" | bc)
done < ch01.data

echo "Suma: $SUM != 536"
rm -f ch01.data
```

</details>

Aquest exemple s'ha adaptat del Llibre **Bash It Out** de Sylvain Leroux. Podeu trobar més informació amb [FOSS Linux Blog](https://itsfoss.com).

## Activitat 02: Comptador de fitxers

En aquesta activitat, volem comptar el nombre de fitxers que hi han en el directori actual. Per això, utilitzarem la comanda `ls`. Aquesta comanda ens permet llistar els fitxers d'un directori.

Per fer-ho crearem un directori (act02) amb 3 fitxers de text:  

```bash
#!/bin/bash
# Nom: act02.sh

mkdir act02
cd act02
touch file1.txt file2.txt $'file3\nok'.txt
ls -l 
ls | wc -l
cd ..
rm -rf act02
```

El resultat de l'execució d'aquestes comandes és el següent:

```bash
total 0
-rw-r--r-- 1 user user 0 mar  4 11:45 file1.txt
-rw-r--r-- 1 user user 0 mar  4 11:45 file2.txt
-rw-r--r-- 1 user user 0 mar  4 11:45 file3\nok.txt

4
```

**NOTA**: La comanda `ls -l` ens mostra el nombre de fitxers que hi ha en el directori actual. En aquest cas, hi ha tres fitxers. La comanda `ls | wc -l` ens mostra el nombre de línies que retorna la comanda `ls`.

Quina és la diferència entre les dues comandes? Per què la comanda `ls -l` retorna tres fitxers i la comanda `ls | wc -l` retorna quatre línies?

<details>
<summary>Resposta</summary>

La comanda `ls -l` ens mostra el nombre de fitxers que hi ha en el directori actual. En aquest cas, hi ha tres fitxers. La comanda `ls | wc -l` ens mostra el nombre de línies que retorna la comanda `ls`. En aquest cas, la comanda `ls` retorna quatre línies. Això és degut a que el fitxer `file3\nok.txt` conté un salt de línia. Per tant, la comanda `ls` retorna quatre línies. 

Tingueu en compte que `\n` és un caracter vàlid per un nom de fitxer. Es cert que no es recomana utilitzar aquest caracter en els noms de fitxers, però és vàlid.

Per obtenir el nombre correcte podem utiltizar `ls -q | wc -l`. Aquesta comanda ens mostra el nombre de fitxers que hi ha en el directori actual. En aquest cas, hi ha tres fitxers.
</details>

Aquest exemple s'ha extret del Llibre **Bash It Out** de Sylvain Leroux. Podeu trobar més informació amb [FOSS Linux Blog](https://itsfoss.com).

En aquest punt ja podem implementar el nostre script. El script **count.sh** ha de comptar el nombre de fitxers total de fitxers acabats en una extensió determinada i en un directori concret. La extensió i el directori són arguments que es passen al programa. El resultat de l'script serà el nombre total de fitxers i que mostri aquest número per pantalla.

a. Implementa utiltizant les comandes *ls* i *wc*.

<details>
<summary>Resposta</summary>

```bash
#!/bin/bash
# Nom: count.sh

if [ $# -ne 2 ]; then
    echo "Usage: $0 <dir> <ext>"
    exit 1
fi

dir=$1
ext=$2

ls $dir/*.$ext | wc -l
```

</details>

b. Implementa utilitzant una estructura iterativa **for in**.

<details>
<summary>Resposta</summary>

```bash
#!/bin/bash
# Nom: count.sh

if [ $# -ne 2 ]; then
    echo "Usage: $0 <dir> <ext>"
    exit 1
fi

dir=$1
ext=$2

count=0
for file in $dir/*.$ext; do
    count=$((count+1))
done

echo $count
```

</details>

c. Compara els dos scripts. Quin és més eficient des del punt de vista del temps d'execució?

Per mesurar l'eficiencia dels dos scripts, utilitzarem la comanda `time`. Aquesta comanda ens permet mesurar el temps d'execució d'un programa. Per això, inicialitzarem un directori amb 1000 fitxers de text i mesurarem el temps d'execució dels dos scripts. Els resultats són els següents:

```bash
#!/bin/bash
# Nom: test_count.sh

# Crearen un directori amb 1000 fitxers de text
mkdir test
for i in {1..1000}; do
    touch test/file$i.txt
done

# Executar els dos scripts
time bash ./count_txt_a.sh test txt
time bash ./count_txt_b.sh test txt

# Eliminar el directori
rm -rf test
```

Els resultats de l'execució dels dos scripts són els següents:

```bash
1000

real	0m0,089s
user	0m0,045s
sys	0m0,041s
1000

real	0m0,064s
user	0m0,056s
sys	0m0,006s
```

Aquests resultats ens mostren que el segon script és més eficient que el primer. El camp `real` ens mostra el temps real d'execució del programa. El camp `user` ens mostra el temps que el processador ha estat executant el programa. El camp `sys` ens mostra el temps que el processador ha estat executant el sistema. Per tant, podem veure que el segon script és més eficient que el primer. El primer script ha estat executat durant 0,089 segons, mentre que el segon script ha estat executat durant 0,064 segons. Això és degut a que el primer script executa dues comandes (`ls` i `wc`) i el segon script només executa una comanda (`for`).

d. Implementa una nova versió del script. Aquesta versió ens ha de permetre comptar el nombre de fitxers en un directori i en els subdirectoris d'aquest directori.

<details>
<summary>Resposta</summary>

```bash
#!/bin/bash
# Nom: count_rec.sh

if [ $# -ne 2 ]; then
    echo "Usage: $0 <dir> <ext>"
    exit 1
fi  

dir=$1
ext=$2

count=0
for file in $dir/*.$ext; do
    count=$((count+1))
done

for subdir in $dir/*; do
    if [ -d $subdir ]; then
        count=$((count+$(bash $0 $subdir $ext)))
    fi
done

echo $count
```

</details>

## Activitat 03: Eliminació segura de fitxers

Imagineu que revisant el vostre servidor debian us trobeu amb aquesta situació:

```bash
$ ls -1
-rf *
```

Com eliminaríeu aquest fitxer? Es perillosa aquesta situació?

<details>
<summary>Resposta</summary>

Molts haureu respost ho eliminaré amb `rm -rf *`. Aquesta resposta és incorrecta. Aquesta comanda eliminarà tots els fitxers del directori actual. Això és degut a que l'asterisc (*) representa tots els fitxers del directori actual. Per tant, la comanda `rm -rf *` eliminarà tots els fitxers del directori actual.

</details>


Com ho faríeu per eliminar aquest fitxer de forma segura?

<details>
<summary>Resposta</summary>

Per poder eliminar aquest fitxer de forma segura, podem utilitzar `rm -- '-rf *'`. Aquesta comanda ens permet eliminar el fitxer `-rf *`. Això és degut a que `--` indica que tots els arguments que hi ha després d'aquesta comanda són arguments de la comanda `rm`. Per tant, la comanda `rm -- '-rf *'` eliminarà el fitxer `-rf *`.

</details>

Aquest exemple s'ha extret del Llibre **Bash It Out** de Sylvain Leroux. Podeu trobar més informació amb [FOSS Linux Blog](https://itsfoss.com).

La comanda `rm` estàndard no demana confirmació abans d’eliminar un arxiu. Escriurem un script anomenat `safe_rm`, tal que faci una còpia abans d’eliminar un únic arxiu (és a dir, únicament permetrà eliminar un sol arxiu a la vegada). Aquesta còpia la guardarà al directori apuntat per la variable d'entorn **TRASH**.

- El script rep un únic argument des de la línia de comandes. Mostra un missatge d’error si no s’ha passat cap argument o si n’hi ha més d’un.
- Crearà un directori anomenat **safe_rm_recycle** en el directori home de l'usuari que executa l'scrpit, si encara no existeix.
- El script copiarà l’arxiu indicat pel primer argument al directori **safe_rm_recycle**.
- Eliminarà l’arxiu del directori actual.

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 <file>"
    exit 1
fi

file=$1
trash=$HOME/.safe_rm_recycle

if [ ! -d $trash ]; then
    mkdir $trash
fi

cp $file $trash
rm $file
```

Podem instal·lar aquest script en el directori **$HOME/bin** i afegir aquest directori al **PATH** del sistema. Per això, executarem les següents comandes:

```bash
mkdir $HOME/bin
cp safe_rm $HOME/bin

export PATH=$PATH:$HOME/bin
```

Si volem que aquestes comandes s'executin cada cop que s'obri una nova terminal, hem d'afegir-les al fitxer **.bashrc**. Per això, executarem la següent comanda:

```bash
echo "export PATH=$PATH:$HOME/bin" >> $HOME/.bashrc
source $HOME/.bashrc
```

Podem millorar aquest script perquè ens demani confirmació abans d'eliminar l'arxiu. Per això, utilitzarem la comanda `read`. Aquesta comanda ens permet llegir una línia de la entrada estàndard i guardar-la en una variable. El següent script és una millora del script anterior:

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 <file>"
    exit 1
fi

file=$1
trash=$HOME/.safe_rm_recycle

if [ ! -d $trash ]; then
    mkdir $trash
fi

echo "Are you sure you want to delete $file? (y/n)"
read answer

if [ $answer == "y" ]; then
    cp $file $trash
    rm $file
fi
```

## Activitat 04: Fàbrica de processos

Ara implementarem un script que ens permeti comprovar si existeix un altre procés en execució amb el mateix nom però PID diferent executant-se en el sistema. En cas afirmatiu, el script ha de mostrar un missatge d'error i acabar amb un codi d'error. 

La comanda `pidof` ens permet obtenir el PID d'un procés a partir del seu nom. Per exemple, per obtenir el PID del procés `bash`:

```bash
pidof bash
```

```bash
sleep 120 &; pidof sleep
# Resultat: 1234
```

La comanda `basename` ens permet obtenir el nom d'un fitxer a partir de la seva ruta. Per exemple, per obtenir el nom del fitxer `/home/user/file.txt`:

```bash
basename /home/user/file.txt
# Resultat: file.txt
```

Per tant, si obtenim un PID a partir del nom del programa (basename $0) i el nombre de PID és major que 1, llavors ja existeix un procés en execució amb el mateix nom. El següent script implementa aquesta funcionalitat:

```bash
#!/bin/bash
# Nom: check_process.sh

if [ $# -ne 1 ]; then
    echo "Usage: $0 <name>"
    exit 1
fi

name=$1

if [ $(pidof $(basename $0) | wc -w) -gt 1 ]; then
    echo "Error: Existeix un procés anomenat $name en execució. El seu PID és $(pidof $(basename $0))."
    exit 1
fi
```

Ara crearem un altre script que farà un bucle infinit. Per exemple:

```bash
#!/bin/bash
# Nom: bucle_infinit.sh

while : ; do
    echo "Press [CTRL+C] to stop.."
    sleep 1
done
```

Aquest script es pot executar en background. Per exemple:

```bash
bash bucle_infinit.sh &
```

Ara podem utiltizar el script `check_process.sh` per comprovar si existeix un procés en execució amb el mateix nom. Per exemple:

```bash
bash check_process.sh bucle_infinit.sh
# Resultat: Error: Existeix un procés anomenat bucle_infinit.sh en execució. El seu PID és 1234.
```

Podem millorar el script `check_process.sh` perquè ens permeti eliminar el procés en execució. Per això, utilitzarem la comanda `kill`.

```bash
#!/bin/bash
# Nom: check_process.sh

if [ $# -ne 1 ]; then
    echo "Usage: $0 <name>"
    exit 1
fi

name=$1
pid=`pidof $(basename $0)`

if [ $(pid | wc -w) -gt 1 ]; then
        echo "[ERROR]: Aquest script ja s’esta executant amb pid $pid"
        kill $pid
        echo "[INFO]: S’ha eliminat el procés $pid"
    fi
    exit 1
fi
```

Ara farem un nou script que ens permeti crear **N** processos en execució al background. Aquest processos faran un sleep d'un nombre aleatori de segons i després acabaran. El procés pare esperarà que els fills acabin i mostrarà l'estat d'acabament de cada procés fill.

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 <n>"
    exit 1
fi

n=$1
declare -i completed=0
declare -i failed=0
declare -a pids

for ((i=0; i<$n; i++)); do
    sec=$((RANDOM % 10))
    sleep $sec &
    pids[$i]=$!
    printf "Process %d with pid %d will sleep for %d seconds\n" $i ${pids[$i]} $sec
done

for pid in ${pids[@]}; do
    wait $pid
    status=$?
    echo "Process $pid finished with status $status"
    
    if [ $status -eq 0 ]; then
        completed=$((completed+1))
    else
        failed=$((failed+1))
    fi
done

echo "Completed: $completed"
echo "Failed: $failed"
```

Per causar un error en un procés, el pare pot matar un procés fill. Per això, utilitzarem la comanda `kill`. Per exemple, per matar el procés fill amb PID 1234:

```bash

#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 <n>"
    exit 1
fi

n=$1
declare -i completed=0
declare -i failed=0
declare -a pids

for ((i=0; i<$n; i++)); do
    sec=$((RANDOM % 10))
    sleep $sec &
    pids[$i]=$!
    printf "Process %d with pid %d will sleep for %d seconds\n" $i ${pids[$i]} $sec
done

# Matarem un procés fill
target=$((RANDOM % 9))
kill ${pids[$target]}

for pid in ${pids[@]}; do
    wait $pid
    status=$?
    echo "Process $pid finished with status $status"
    
    if [ $status -eq 0 ]; then
        completed=$((completed+1))
    else
        failed=$((failed+1))
    fi
done

echo "Completed: $completed"
echo "Failed: $failed"
```

Podem millorar aquest script implementant diferents funcions. Per exemple, una funció per crear els processos fills, una funció per esperar que els processos fills acabin i una funció per mostrar l'estat d'acabament de cada procés fill. El següent script implementa aquestes funcions:

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo "Usage: $0 <n>"
    exit 1
fi

n=$1

declare -i completed=0
declare -i failed=0
declare -a pids

function create_processes() {
    for ((i=0; i<$n; i++)); do
        sec=$((RANDOM % 10))
        sleep $sec &
        pids[$i]=$!
        printf "Process %d with pid %d will sleep for %d seconds\n" $i ${pids[$i]} $sec
    done
}

function wait_processes() {
    for pid in ${pids[@]}; do
        wait $pid
        status=$?
        echo "Process $pid finished with status $status"
        
        if [ $status -eq 0 ]; then
            completed=$((completed+1))
        else
            failed=$((failed+1))
        fi
    done
}

function show_status() {
    echo "Completed: $completed"
    echo "Failed: $failed"
}

create_processes
wait_processes
show_status
```
