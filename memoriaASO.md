


# Memoria proyecto xv6
### *Ampliación de Sistemas Operativos*

> Isabel María Salcedo y Álvaro Miñarro | Grupo 3.4

## Ejercicio 1: getDate

(Ejercicio guiado en clase)
1. Se añade *date* a la variable UPROGS del Makefile dentro de la carpeta user.

2. Se añade el fichero date.c, que será el programa de usuario que llamará a la nueva llamada al sistema. Se proporciona el código en la presentación.

3. En **syscall.h** se le da un número a la llamada.

4. En **user/usys.S** se añade la llamada *date*.

5. En **syscall.c** añadimos la definición de la función *sys_date()*.

6. Se implementa la llamada *sys_date()* en **sysproc.c**.



## Ejercicio 2: dup2

Para la creación de esta llamada al sistema hemos seguido los pasos del ejercicio 1 pero aplicados a esta función. En este caso la implementación de la función se hace en **sysfile.c**, justo debajo de *sys_dup()*. La implementación de *sys_dup2()* consiste en recuperar el descriptor antiguo de fichero con argfd y el nuevo con argint, ya que no sabemos si este descriptor está libre o apunta ya a algo. Se hacen las respectivas comprobaciones y, en el caso de que apuntara a algo, este descriptor se cerraría y haríamos que apuntase a lo mismo que f_old.

Otra modificación que pedía el ejercicio era la del shell, en la función *runcmd* en el fichero **sh.c**. Concretamente, en el caso de PIPE, donde antes se usaba dup y ahora teníamos que pasar a usar dup2, por lo que ya no hace falta cerrar los descriptores de fichero antes de hacer dup2, ya que la propia función lo hace si es necesario.

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

Pasos para getPrio y setPrio:

1. Añadimos los números de syscall SYS_getprio (24) y SYS_setprio (25). **(syscall.h)**

2. Implementamos sys_getprio y sys_setprio que recogen los argumentos con argint y llaman a las funciones del kernel. **(sysproc.c)**

3. Añadimos las declaraciones extern y las entradas en la tabla de syscalls. **(syscall.c)**

4. Declaramos getprio y setprio en las definiciones del kernel. **(defs.h)**

5. Implementamos getprio(pid) que recorre la ptable buscando el proceso por PID y devuelve su prioridad, o -1 si no existe. **(proc.c)**

6. Implementamos setprio(pid, prio) que valida el rango de prioridad (0-9), busca el proceso por PID, actualiza su prioridad y si está RUNNABLE lo mueve de su cola antigua a la nueva. **(proc.c)**

7. Añadimos los stubs SYSCALL(getprio) y SYSCALL(setprio) para el espacio de usuario. **(user/usys.S)**

8. Declaramos las funciones getprio y setprio para programas de usuario. **(user/user.h)**


