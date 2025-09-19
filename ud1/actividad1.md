# Ejemplo: LanzaProceso.java

Este programa en Java lanza un proceso externo y muestra su código de salida.

```java
import java.io.IOException;
import java.util.Arrays;

public class LanzaProceso {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.err.println("Debe indicarse comando a ejecutar.");
            System.exit(1);
        }

        ProcessBuilder pb = new ProcessBuilder(args);

        try {
            Process p = pb.start();
            int codRet = p.waitFor();
            System.out.println("La ejecución de " + Arrays.toString(args)
                    + " devuelve " + codRet
                    + " -> " + (codRet == 0 ? "Ejecución correcta" : "ERROR"));
        } catch (IOException e) {
            System.err.println("Error durante ejecución del proceso");
            e.printStackTrace();
            System.exit(2);
        } catch (InterruptedException e) {
            System.err.println("Proceso interrumpido");
            System.exit(3);
        }
    }
}
```

---

## Actividades propuestas

### 1.2 Prueba el programa con los siguientes casos y comenta su funcionamiento:

a) Ejecución de un proceso que se ejecuta sin errores.  
   Ejemplo en Linux:  
   ```bash
   java LanzaProceso ls /etc
   ```

b) Ejecución de un programa que no existe.  
   Ejemplo en Linux:  
   ```bash
   java LanzaProceso programa_que_no_existe
   ```

c) Ejecución de un proceso que termina con un código de error.  
   Ejemplo en Linux:  
   ```bash
   java LanzaProceso ls /et
   ```

---

### 1.3 Crear un programa que use `isAlive()`

Escribe un programa que lance un proceso y utilice el método `isAlive()` para comprobar si sigue en ejecución.  
Debe comprobar cada 3 segundos si el proceso sigue vivo, mostrando un mensaje. Cuando ya no lo esté, el programa debe terminar.

Puedes utilizar `Thread.sleep(int tiempo_ms)` para pausar la ejecución.

Ejemplo de **script de prueba** (bucle.sh), que dura unos 10 segundos:

```bash
#!/bin/bash
i=0
while [ $i -lt 10 ]
do
  echo $i
  i=$((i + 1))
  sleep 1
done
```

Ejecución:
```bash
chmod +x bucle.sh
java ComprobarProceso ./bucle.sh
```
