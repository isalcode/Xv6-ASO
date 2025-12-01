


# Memoria proyecto xv6
### *Ampliación de Sistemas Operativos*

> Isabel María Salcedo y Álvaro Miñarro | Grupo 3.4

## Ejercicio 1: getDate

(Ejercicio guiado en clase)



## Ejercicio 2: dup2

## Ejercicio 3: exit y wait

Pasos seguidos:

1. Sustitución los exit() por exit(0) en los programas de usuario con el comando proporcionado por el enunciado.

2. Sustitución de wait() por wait(NULL) también con el comando proporcionado por el enunciado.

3. Modificación de las funciones sys_exit y sys_wait para que con argint (exit) y argptr (wait) accedan al argumento que se le pasa a exit y wait, en el caso de exit un entero y en el caso de wait un puntero a un entero. **(sysproc.c)**

4. Edición de las funciones del kernel exit y wait del fichero proc.c, añadiendo a la estructura del proceso (struct proc) un int exit_status para que recoja el estado de salida. En exit le hemos añadido el parámetro int que se le pasa y luego curproc->exit_status = status antes de que el proceso pase a estado ZOMBIE. En wait hemos usado copyout(curproc->pgdir, (uint)status, (char*)&p->exit_status, sizeof(int)); para copiar exit_status al puntero, usar copyout es necesario para escribir en memoria de usuario desde el kernel. Status != 0 para comprobar que el puntero es válido. **(proc.c)**

5. Modificación de las declaraciones de exit y wait para que tengan los parámetros adecuados. *(defs.h)*

6. En trap.c, exit(tf->trapno+1) para que el codigo que se le pase sea el número de interrupción si ha sido esta la que ha provocado la salida del proceso. **(trap.c)**

7. Se recoge el estado de salida para imprimirlo por pantalla al acabar un proceso. **(sh.c)**

8. Se añaden las macros proporcionadas por el enunciado para darle el funcionamiento de posix.

## Ejercicio 4: Reserva de páginas bajo demanda

EJERCICIO 1: Implementacion basica (sysproc.c y trap.c)

1. En **sysproc.c**, modificamos sys_sbrk() para que simplemente aumente el tamano del proceso (p->sz) sin reservar memoria fisica real cuando el incremento es positivo.

2. En **trap.c**, capturamos la excepcion T_PGFLT (Page Fault) para reservar (kalloc), limpiar y mapear (mappages) una pagina fisica bajo demanda en la direccion que causo el fallo.

EJERCICIO 2: Avisar de que estamos mintiendo sobra aumentar el tamaño y que no pete (vm.c)

1. Modificamos copyuvm() (usada por fork) para que ignore silenciosamente las paginas no presentes en lugar de lanzar un panic(usar continue), permitiendo que el hijo herede la condicion "lazy" del padre.

## Ejercicio 5: scheduler, getPrio y setPrio

Pasos para modificar el scheduler y crear la estructura de colas:

1. Modificación del struct proc añadiendo los atributos prio (prioridad) y un puntero al siguiente proceso de la cola (sig). **(proc.h)**

2. En la ptable, hemos añadido dos arrays de punteros a struct proc con el tamaño de la cola que apuntan al principio y al final de cada cola de prioridad (para insertar y para sacar). **(proc.c)**

3. En alloc proc inicializamos a null el puntero al siguiente y la prioridad a 5 cuando encontramos un proceso que está UNUSED. **(proc.c)**

4. Creamos una función para insertar en cola a los procesos que inserta a los procesos por el final de la cola usando el puntero de final de cola y si la cola está vacía hace que apunten a este proceso tanto el final como el inicio de la cola. **(addToQueue -> proc.c)**

5. Ahora añadimos esta función en userinit y en fork cada vez que se cambia el estado de un proceso a RUNNABLE que es cuando debe de estar en las colas. También en yield(), en kill() y en wakeup1(). **(proc.c)**

6. Hemos creado la funcion popFromQueue para sacar de la cola el primer proceso y actualizar los punteros de las colas correctamente, contemplando el caso de que la cola quede vacía. **(proc.c)**

7. Para cambiar el scheduler, ahora lo que tiene que recorrer son las colas, no la ptable, y cuando encuentre una (de mayor prioridad a menor) con un proceso esperando lo saca (popFromQueue) y le da la cpu. Luego, cuando este proceso deja de tener la cpu se empieza a iterar desde la cola 0, la de máxima prioridad. **(proc.c)**

Pasos para setPrio:

Pasos para getPrio:


