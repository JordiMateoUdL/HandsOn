# Exercicis de les diapositives Shell Scripting

Escriu una funció que calculi el màxim d'una llista de nombres enters.

```bash
#!/bin/bash
function maximum {
  # TODO
}

maximum 4 8 15 16 23 42
echo "Maximum [4 8 15 16 23 42] is: $?"
exit 0
# Sortida: Maximum [4 8 15 16 23 42] is: 42
```

<details>
<summary>Solució (versió 0)</summary>

En aquesta versió s'usa l'operador `[ ]` per a comparar els nombres. Això requereix inicialitzar la variable `max` a 0. També cal `$` davant de les variables `i` i `max`. Això és perquè l'operador `[ ]` és un operador de substitució de variables i necessita expandir les variables abans de comparar-les. Expandir una variable significa substituir-la pel seu valor.

```bash
#!/bin/bash
function maximum {
  max=0
  for i in $@; do
    if [ "$i" -gt "$max" ]; then
      max=$i
    fi
  done
  return $max
}
```

</details>

<details>
<summary>Solució (versió 1)</summary>

En aquesta versió s'usa l'operador aritmètic `(( ))` per a comparar els nombres. Això permet evitar la inicialització de la variable `max` a 0. Tampoc cal `$` davant de les variables `i` i `max`. Això és perquè l'operador `(( ))` no és un operador de substitució de variables, sinó un operador aritmètic.

```bash
#!/bin/bash
function maximum {
  for i in $@; do
    if (( i > max )); then
      max=$i
    fi
  done
  return $max
}
```

Un altra versió més compacta és la següent:

```bash
#!/bin/bash
function maximum {
  for i in $@; do
    (( i > max )) && max=$i
  done
  return $max
}
```
</details>

<details>
<summary>Solució (versió 2)</summary>

Un altra possible solució és utilitzant el bucle `while` i l'operador `expr`. També utilitza l'operador `shift` per a processar els paràmetres.

```bash
#!/bin/bash

function maximum {
  max=0
  while [ $# -gt 0 ]; do
    if [ `expr $1 \> $max` -eq 1 ]; then
      max=$1
    fi
    shift
  done
  return $max
}
```

</details>

<details>
<summary>Solució (versió 3)</summary>

Un altra possible solució és utilitzant el bucle `for` i  recorrent utilitzant l'índex `i`. En aquest cas, utilitzem la sintaxi `${!i}` per a accedir al valor de la variable `i`. L'operador '!` és un operador de substitució de variables i necessita expandir la variable `i` abans de substituir-la pel seu valor.

```bash
#!/bin/bash
function maximum {
  max=0
  for (( i=1; i<=$#; i++ )); do
    if (( ${!i} > max )); then
      max=${!i}
    fi
  done
  return $max
}
```

o bé utilitzant el shift:

```bash
#!/bin/bash
function maximum {
  mida=$#
  max=0
  for (( i=1; i<=$mida; i++ )); do
    if (( $1 > max )); then
      max=$1
    fi
    shift
  done
  return $max
}
```

o bé accedint directament als vectors de paràmetres $@ indexat per $i:

```bash
#!/bin/bash
function maximum {
  declare -a array=("$@")
  max=${array[0]}  

  for ((i=1; i<${#array[@]}; i++)); do
    if (( ${array[i]} > $max )); then
      max=${array[i]}
    fi
  done

  echo $max 
}
```

</details>

## Exercici 2

```bash
#!/bin/bash
mkdir -p test
touch test/fitxer1.txt
touch test/fitxer2.txt
touch test/fitxer3.c
touch test/fitxer4.txt

# TODO

echo "Fitxers amb extensió .txt modificats a .old:"
ls test/*.old
# un altra forma per expandir el contingut de ls
ls test/*.{old}
rm -rf test
exit 0
# Sortida:
# Fitxers amb extensió .txt modificats a .old:
# fitxer1.old fitxer2.old fitxer4.old
```

<details>
<summary>Solució</summary>

```bash
#!/bin/bash
mkdir -p test
touch test/fitxer1.txt
touch test/fitxer2.txt
touch test/fitxer3.c
touch test/fitxer4.txt

for i in test/*.txt; do
  mv $i ${i%.txt}.old
done

echo "Fitxers amb extensió .txt modificats a .old:"
ls test/*.old

rm -rf test
```

La comanda `ls test/*.old` mostra els fitxers amb extensió `.old` que hi ha al directori `test`. 

Per exemple:

```bash
mkdir -p test
touch test/fitxer1.txt
touch test/fitxer2.txt
touch test/fitxer3.c
touch test/fitxer4.txt
bash -c 'ls test/*.c'
sh -c 'ls test/*.c'
rm -rf test
```

Un altra forma de mostrar els fitxers amb extensió .old és utilitzant la comanda `grep`:

```bash
mkdir -p test
touch test/fitxer1.txt
touch test/fitxer2.txt
touch test/fitxer3.c
touch test/fitxer4.txt

bash -c 'ls test | grep '\.c$''
sh -c 'ls test | grep '\.c$''

# A classe us he posat *.c però per funcionar amb * necessiteu posar -e *.c

bash -c 'ls test | grep -E '*.c$''
# grep: warning: * at start of expression

Per tant, es millor utilitzar el grep inicial.

# A classe heu comentat un altra solució
bash -c 'ls test/${1:-*}.c'
# En aquest cas ${1:-*} és una substitució de paràmetres. Si no hi ha paràmetres, el valor per defecte és *.

# Es pot simplificar així:
bash -c 'ls test/${1-*}.c'

# Finalment, es pot utilitzar la comanda find:
bash -c 'find test -name "*.c"'
sh -c 'find test -name "*.c"'

rm -rf test
```

</details>

## Exercici 3

Crea un script que llegeixi un fitxer de text amb el nom users.txt i que contingui una llista de noms d'usuari. El format del fitxer és el següent:
user;password;status. Aquest script ha de mostrar per pantalla tots els atributs dels usuaris que estan actius (status=1).

<details>
<summary>Solució</summary>

```bash
#!/bin/bash

cat << EOF > users.txt
user1;password1;1
user2;password2;0
user3;password3;1
user4;password4;0
EOF

IFS=';'
while read user password status; do
  if [ "$status" -eq 1 ]; then
    echo "User: $user"
    echo "Password: $password"
    echo "Status: $status"
    echo
  fi
done < users.txt

rm users.txt
```

</details>