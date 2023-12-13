# Mapa de memòria d'un procés

El mapa de memòria d'un procés esta format per regions de memòria. Cada regió és una zona contigua de memòria amb unes característiques comunes. Aquestes característiques són:

- **Direcció inicial**: Direcció de memòria on comença la regió.
- **Direcció final**: Direcció de memòria on acaba la regió.
- **Permisos**: Permisos d'accés a la regió.
- **Tipus**: Tipus de regió.
- **Nom**: Nom de la regió.
- **Mida**: Mida de la regió.
- **Fitxer**: Fitxer associat a la regió.
- **Biblioteca**: Biblioteca associada a la regió.

Per obtenir el mapa de memòria d'un procés s'ha d'executar la comanda `pmap` amb el PID del procés. Per exemple, per obtenir el mapa de memòria del procés amb PID 1:

```bash
pmap 1
1:   /sbin/init splash
00007f5b6c000000    152K r-x--   /sbin/init
00007f5b6c022000   8192K -----   /sbin/init
00007f5b6c822000      4K r----   /sbin/init
00007f5b6c823000      4K rw---   /sbin/init
00007f5b6c824000     12K rw---   [ anon ]
00007f5b6c827000     92K r-x--   /lib/x86_64-linux-gnu/libnss_files-2.31.so
00007f5b6c83e000   2044K -----   /lib/x86_64-linux-gnu/libnss_files-2.31.so
00007f5b6ca3d000      4K r----   /lib/x86_64-linux-gnu/libnss_files-2.31.so
...
```

En aquest exemple, podem veure diferents regions de memòria. Per exemple, la primera regió de memòria comença a l'adreça `0x00007f5b6c000000` i acaba a l'adreça `0x00007f5b6c022000`. Aquesta regió té permisos de lectura i execució (`r-x--`). Aquesta regió està associada al fitxer `/sbin/init` i té una mida de 152K.

Quan s'activa l'execució d'un programa, es creen diferent regions dins del mapa de memòria del procés. Aquestes regions són:

1. **Codi**: Regió compartida de lectura i execució. Mida fixa. Conté el codi del programa.
2. **Dades inicialitzades**: Regió privada on cada procés té una còpia pròpia de les variables. Lectura, escriptura i mida fixa. Conté variables globals, estàtiques i inicialitzades.
3. **Dades no inicialitzades**: Regió privada de lectura i escriptura. Mida fixa. Conté variables globals, estàtiques i no inicialitzades.
4. **Heap**: Regió privada de lectura i escriptura. Mida variable. Aquesta regió creixarà amb l'assignació dinàmica de memòria i decreixerà amb la seva alliberació. Normalment, creix cap a les direccions més altes del mapa.
5. **Stack**: Regió privada de lectura i escriptura. Mida variable. Aquesta regió creixerà amb les crides a funcions, i decreixerà quan aquestes retornin. Normalment, creix cap a les direccions més baixes del mapa.

```bash
Adreça Baixa
    +------------------------+
    | Codi                   |  
    +------------------------+
    | Dades Inicialitzades   |  
    +------------------------+
    | Dades No Inicialitzades|  
    +------------------------+
    | Heap                   |  
    +------------------------+
    | Stack                  |  
    +------------------------+
Adreça Alta
```





Considereu el següent programa:

```c
int a;
char *s;
char buf[1024];

int
main(int argc, char **argv) {
    int b;
    char *string = "abcd";
}
```

Indiqueu el mapa de memòria del procés que s'executa amb aquest programa. Indicant a quin segment (text,dades inicialitzades, dades no inicialitzades, heap i stack) es trobaran les diferents variables.

<details>
<summary>Veure solució</summary>

```bash
+-------------------------+
| Codi                    |  (Codi executable - no proporcionat)
+-------------------------+
| Dades Inicialitzades    |  (a, s)
+-------------------------+
| Dades No Inicialitzades |  (buf)
+-------------------------+
| Heap                    |  (res)
+-------------------------+
| Stack                   |  (b, string, argc, argv)
+-------------------------+
```

Les dades inicialitzades contenen les variables globals, estàtiques i inicialitzades. En el nostre programa, **a i s** són variables globals i estàtiques. La variable a és inicialitzada a 0 i la variable s és inicialitzada a NULL per defecte.

Les dades no inicialitzades contenen les variables globals, estàtiques i no inicialitzades. En el nostre programa, buf és una variable global i estàtica i no està inicialitzada. Únicament es reserva memòria per a la variable **buf**.

La heap conté la memòria dinàmica del programa. En el nostre programa, no es fa cap crida a funcions que reservin memòria dinàmica.

La stack conté les variables locals i els paràmetres de les funcions. En el nostre programa, **b** i **string** són variables locals de la funció **main()**. Finalment, els paràmetres de la funció **main()** són **argc** i **argv**.

</details>

## Stack

La **stack** és un bloc de memòria contigu. Utilitza un registre anomenat **stack pointer** (SP) que apunta a la part superior de la stack. La part inferior de la stack està situada a una adreça fixa. La seva mida s'ajusta dinàmicament pel nucli en temps d'execució. La CPU implementa instruccions per **PUSH** i **POP** a la stack.

El sistemes operatius utilitzen la **stack** s'utilitza per emmagatzemar de forma dinàmica les variables locals, per passar els arguments a les funcions o per retorna els valors de les funcions.

Per exemple:

```c
void f(int a, int b, int c) {
   char b1[2];
   char b2[3];
}

void main() {
  f(1,2,3);
}
```

En aquest exemple, la funció **main()** afegeix 3 arguments a la stack i crida a la funció **f()**:

```assembly
push $3
push $2
push $1
call f
```

L'instrucció **call f** posarà l'adreça de retorn RET (*Return Address*) i el punt de referència SFP (*Saved Frame Pointer - SFP*) a la pila abans de saltar a la funció f().

```assembly
push ebp          ; Guarda el valor del frame pointer actual (EBP) a la pila (SFP)
mov ebp, esp      ; Copia el valor actual de SP (stack pointer) a EBP, establint-lo com el nou frame pointer
sub esp, <size>   ; Allibera espai per a les variables locals, restant-ne la mida de SP
```

A la funció **f()**, es reserven dues variables locals a la pila. Aquestes variables són vectors de caràcters amb 2 i 3 posicions, respectivament. Així, es reserven 2 posicions a la pila per a la variable b1 i 3 posicions per a la variable b2, resultant en la següent disposició de la pila:

```css
<------ Part baixa de la Memòria       Part alta de la Memòria ------>
b2[0] b2[1] b2[2] b1[0] b1[1] sfp ret a b c
<------ Part alta de la Stack           Part baixa de la Stack ------>
```

Quan la funció **f()** finalitza la seva execució, la instrucció ret agafa l'adreça de retorn i el SFP de la pila i els utilitza per restaurar l'Instruction Pointer (IP), permetent a l'execució del programa continuar després de la crida a la funció.

> Noteu: Un Buffer Overflow es donarà quan s'intenta escriure dades a un buffer que està fora dels límits del buffer. Aquests errors es poden aprofitar per executar codi arbitrari.

Per a més informació, podeu consultar el següent manual des d'on s'ha extret aquest exemple: [Smashing the Stack for Fun and Profit](http://insecure.org/stf/smashstack.html).

**Quina diferència hi ha entre StackOverflow i BufferOverflow?** *Poseu un exemple de cada cas.*

<details>
<summary>Veure solució</summary>

Un **StackOverflow** es produeix quan s'intenta escriure dades a la stack que estan fora dels límits de la stack. Per tant, la stack es desborda. Una manera molt simple de desbordar-la és cridant a una funció recursiva que no té condició de sortida. D'aquesta manera la stack va creixent fins que es desborda. Per exemple:

```c
void f() {
  f();
}

void main() {
  f();
}
```

Un **BufferOverflow** es produeix quan s'intenta escriure dades a un buffer que està fora dels límits d'un buffer. Per exemple:

```c
void f() {
  char buffer[10];
  gets(buffer);
}

void main() {
  f();
}
```

En aquest cas, si un usuari introdueix més de 10 caràcters, el buffer es desborda.

</details>

## Heap

La **heap** és una àrea de memòria utilitzada per emmagatzemar dades dinàmiques. A diferència de la stack, la heap permet allotjar dades que no tenen una vida útil vinculada a la crida de funcions i que han de persistir durant tota l'execució del programa. La gestió de la heap és responsabilitat del programador, i sovint es fan servir funcions com ```malloc``` i ```free``` en llenguatges com C per assignar i alliberar memòria dinàmica.

Quan es demana memòria a la **heap**, el sistema operatiu pot assignar blocs contigus de memòria per satisfer la sol·licitud.

```c
int *p = (int*)malloc(sizeof(int));
```

Es podria donar el cas que la **heap** i la **stack** creixin cap a la mateixa direcció i es trobin. En aquest cas, si la heap creix cap a la stack, es podria donar el cas que la heap sobreescrigués la stack. Això es coneix com a **heap overflow**.

Avalua el següent codi i indica quins problemes es produeixen i quina diferencia hi ha entre utilitzar la funció ```malloc``` o ```calloc``` (A,B).

```c
#include <stdlib.h>

int main() {
    int *array = (int *)calloc(5 * sizeof(int)); //A
    int *array = (int *)malloc(5 * sizeof(int)); //B

    for (int i = 0; i < 5; i++) {
        printf("array[%d] = %d\n", i, array[i]);
    }

    array[5] = 42;
    free(array);
    return 0;
}
```

<details>
<summary>Veure solució</summary>

En aquest exemple, es demana memòria per a 5 enters. Si utilitzem la versió (A) amb ```malloc```. Podem observar el primer problema de Segmentation Fault. Això es degut a que ```malloc``` no inicialitza la memòria que reserva. Per tant, no podem imprimir els valors de la memòria reservada. En el cas de la versió (B) amb ```calloc```, la memòria reservada es inicialitzada a 0. Per tant, podem imprimir els valors de la memòria reservada sense problemes.

El segon problema es produeix quan s'intenta accedir a la posició 5 del vector. En aquest cas, el programa intenta accedir a la posició 5 del vector. En aquest cas, les dues versions produeixen un **heap overflow**.

</details>
